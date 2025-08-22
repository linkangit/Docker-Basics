# Docker Basics: A Practical Tutorial to Build, Run, and Ship Containers

This hands-on guide walks you from zero to publishing a production-ready image. We’ll focus on Docker **images** (what you “generate”) and **containers** (what you run).


## What you’ll need

* Docker Desktop (macOS/Windows) or Docker Engine (Linux).
* A terminal.
* A Docker Hub or GitHub Container Registry (GHCR) account (optional, for pushing).

## 1) Core concepts

* **Image**: Read-only template with your app + OS packages.
* **Container**: A running instance of an image (with its own filesystem, process tree, network).
* **Dockerfile**: Recipe for building images.
* **Registry**: Where images live (Docker Hub, GHCR, etc.).
* **Tag**: A label for an image version (e.g., `1.2.0`, `latest`).


## 2) Your first image (Python example)

### Project layout

```
hello-docker/
├─ app.py
├─ requirements.txt
└─ Dockerfile
```

**app.py**

```python
from flask import Flask
app = Flask(__name__)

@app.get("/")
def home():
    return {"msg": "Hello from Docker!"}

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

**requirements.txt**

```
flask==3.0.3
```

**.dockerignore** (create this too—speeds builds, keeps images clean)

```
__pycache__/
*.pyc
*.pyo
*.pyd
*.sqlite3
.env
.git
.gitignore
```

**Dockerfile** (simple + small using slim base)

```dockerfile
# 1) Base layer
FROM python:3.12-slim

# 2) System deps (optional) & no-cache apt pattern
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
 && rm -rf /var/lib/apt/lists/*

# 3) Workdir
WORKDIR /app

# 4) Copy only requirements first (better cache)
COPY requirements.txt .

# 5) Install Python deps
RUN pip install --no-cache-dir -r requirements.txt

# 6) Copy the rest
COPY . .

# 7) Expose app port (documentation only)
EXPOSE 8080

# 8) Default command
CMD ["python", "app.py"]
```

### Build & run

```bash
# build (the trailing dot means “use current directory as build context”)
docker build -t hello-docker:1.0 .

# list images
docker images

# run in foreground, map host port 8080 -> container 8080
docker run --rm -p 8080:8080 hello-docker:1.0

# open http://localhost:8080
```

Stop with `Ctrl+C`. To run detached:

```bash
docker run -d --name hello -p 8080:8080 hello-docker:1.0
docker logs -f hello
docker stop hello
```



## 3) Tagging and pushing to a registry

```bash
# choose your registry namespace (Docker Hub shown)
# log in first:
docker login

# tag
docker tag hello-docker:1.0 yourname/hello-docker:1.0
docker tag hello-docker:1.0 yourname/hello-docker:latest

# push
docker push yourname/hello-docker:1.0
docker push yourname/hello-docker:latest
```

(For GHCR use `ghcr.io/<org-or-user>/<name>:tag` and `docker login ghcr.io`.)



## 4) Multi-stage builds (smaller, faster)

**Go example** (compiles in one stage, runs in tiny runtime):

```dockerfile
# build stage
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN go build -o /out/app ./...

# runtime stage
FROM gcr.io/distroless/base-debian12
COPY --from=build /out/app /app
EXPOSE 8080
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

**Node.js example** (separate deps install + production image):

```dockerfile
# deps stage
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# build (if you transpile) – optional
FROM deps AS build
COPY . .
RUN npm run build

# runtime stage
FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps /app/node_modules /app/node_modules
COPY --from=build /app/dist /app
EXPOSE 3000
CMD ["node", "index.js"]
```



## 5) Development workflow tips

* **Iterate quickly**: copy dependency manifests first (e.g., `requirements.txt`, `package.json`) to leverage Docker layer cache.
* **.dockerignore** aggressively (node\_modules, build artifacts, VCS metadata).
* Bind-mount code for hot reload in dev:

  ```bash
  docker run --rm -it -p 8080:8080 -v "$PWD":/app hello-docker:1.0
  ```
* Use **Compose** to wire multiple services.

**docker-compose.yml** (web + db)

```yaml
services:
  web:
    build: .
    ports: ["8080:8080"]
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/app
    depends_on: [db]
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=postgres
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

Run with:

```bash
docker compose up --build
```



## 6) Debugging containers

* Shell into a running container:

  ```bash
  docker exec -it <container_name_or_id> sh
  # or bash if installed
  ```
* Check logs:

  ```bash
  docker logs -f <container>
  ```
* Inspect networking/ports:

  ```bash
  docker ps
  docker port <container>
  docker inspect <container> --format '{{json .NetworkSettings}}' | jq .
  ```



## 7) Security & production basics

* **Run as non-root** where possible:

  ```dockerfile
  RUN adduser --disabled-password --gecos '' appuser
  USER appuser
  ```
* Pin versions (base images, dependencies).
* Prefer minimal bases (e.g., `-slim`, distroless, alpine\*).

  * *Note*: Alpine + glibc apps can be tricky; test thoroughly.
* Avoid secrets in images. Use env vars, Docker secrets, or your platform’s secret manager.
* Keep images updated; rebuild when base images publish fixes.



## 8) Size optimization checklist

* Multi-stage builds (compile in one stage, copy binaries/assets to slim runtime).
* Clean package caches (apt, pip caches).
* `.dockerignore` to shrink build context.
* For Python: use wheels and `--no-cache-dir`.
* For Node: `npm ci --omit=dev` or separate build/runtime stages.
* For compiled apps: static binaries if appropriate.



## 9) Reproducible tagging strategy

* Semantic: `1.4.2`, `1.4`, `1`, `latest`.
* Git-based: `git describe --tags` or short SHA (`yourapp:sha-abc1234`).
* Automate with CI (GitHub Actions, GitLab CI) to build on push to `main` and tag releases.



## 10) Common pitfalls (and fixes)

* **“It works locally but not in Docker”**
  Expose the right port, bind to `0.0.0.0`, not `127.0.0.1`.
* **Huge images**
  Use multi-stage builds, minimal bases, and prune layers.
* **Cache busting every build**
  Copy dependency manifests before app code.
* **Missing files at runtime**
  Verify `COPY` paths; use `docker run -it --entrypoint sh` to inspect image.
* **Permission errors**
  If running as non-root, ensure file ownership (`chown`) and writable dirs.



## 11) Quick reference (commands you’ll use a lot)

```bash
docker build -t name:tag .
docker run --rm -it -p HOST:CONTAINER name:tag
docker ps -a
docker stop <container>; docker rm <container>
docker images; docker rmi <image>
docker tag src:tag dst:tag
docker login; docker push dst:tag
docker exec -it <container> sh
docker system prune -f
docker compose up --build
```



## 12) Next steps you can try

* Add a healthcheck to your image.
* Add CI to build, scan, and push on every commit.
* Convert a real project you have into a multi-stage build—measure image size before/after.

