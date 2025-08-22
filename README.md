# Docker Tutorial for Complete Beginners

Welcome to your journey into Docker! This tutorial will take you from zero knowledge to running your first containerized application. No prior experience needed.

## What is Docker? (Explained Simply)

Imagine you're moving to a new apartment. You could pack everything loosely in boxes, hoping it all fits and works in your new place. Or you could use shipping containers - standardized boxes that protect your belongings and work anywhere.

Docker does the same thing for software. It packages your application and everything it needs (like a shipping container) so it runs the same way on any computer.

### Key Concepts

- **Container**: A lightweight package containing your application and everything it needs to run
- **Image**: The blueprint or template for creating containers (like a recipe)
- **Docker Engine**: The software that manages containers on your computer

## Docker vs Virtual Machines: What's the Difference?

Think of your computer as an apartment building:

**Virtual Machines** are like having separate apartments, each with their own:
- Kitchen (operating system)
- Utilities (system resources)
- Everything duplicated

**Docker Containers** are like having separate rooms that share:
- The same kitchen (host operating system)
- Utilities efficiently
- Much lighter and faster

```
Virtual Machines          Docker Containers
┌─────────────────┐      ┌─────────────────┐
│   App A         │      │   App A         │
│   OS            │      │   App B         │
│   Hypervisor    │      │   App C         │
├─────────────────┤      │   Docker Engine │
│   App B         │      │   Host OS       │
│   OS            │      └─────────────────┘
│   Hypervisor    │      
├─────────────────┤      
│   Host OS       │      
└─────────────────┘      
```

**Why Docker is Better:**
- Faster startup (seconds vs minutes)
- Uses less memory and storage
- Consistent across different environments
- Easy to share and deploy

## Installing Docker

### Windows (Windows 10/11)

1. **Download Docker Desktop**
   - Visit https://docker.com/products/docker-desktop
   - Click "Download for Windows"

2. **Install Docker Desktop**
   - Run the downloaded installer
   - Follow the setup wizard
   - Restart your computer when prompted

3. **Verify Installation**
   - Open Command Prompt or PowerShell
   - Type: `docker --version`
   - You should see version information

### macOS

1. **Download Docker Desktop**
   - Visit https://docker.com/products/docker-desktop
   - Click "Download for Mac"
   - Choose your chip type (Intel or Apple Silicon)

2. **Install Docker Desktop**
   - Open the downloaded .dmg file
   - Drag Docker to Applications folder
   - Launch Docker from Applications

3. **Verify Installation**
   - Open Terminal
   - Type: `docker --version`

### Linux (Ubuntu/Debian)

1. **Update Package Index**
   ```bash
   sudo apt update
   ```

2. **Install Docker**
   ```bash
   sudo apt install docker.io
   ```

3. **Start Docker Service**
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

4. **Add Your User to Docker Group** (optional, avoids using sudo)
   ```bash
   sudo usermod -aG docker $USER
   ```
   Log out and back in for this to take effect.

5. **Verify Installation**
   ```bash
   docker --version
   ```

## Essential Docker Commands

Let's learn the fundamental commands you'll use every day:

### 1. `docker run` - Create and Start a Container

This is like ordering a meal from a restaurant using a recipe (image).

**Basic syntax:**
```bash
docker run [options] image_name
```

**Example - Run a simple web server:**
```bash
docker run -d -p 8080:80 nginx
```

**What this does:**
- `-d`: Run in background (detached)
- `-p 8080:80`: Connect port 8080 on your computer to port 80 in the container
- `nginx`: The image name (a web server)

After running this, visit http://localhost:8080 in your browser!

### 2. `docker ps` - List Running Containers

See what containers are currently running:

```bash
docker ps
```

**Sample output:**
```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   amazing_tesla
```

**To see all containers (including stopped ones):**
```bash
docker ps -a
```

### 3. `docker stop` - Stop a Running Container

Stop a container using its name or ID:

```bash
docker stop amazing_tesla
```

or

```bash
docker stop a1b2c3d4e5f6
```

### 4. `docker rm` - Remove a Container

Delete a stopped container:

```bash
docker rm amazing_tesla
```

**Pro tip:** Stop and remove in one command:
```bash
docker rm -f amazing_tesla
```

### 5. `docker images` - List Downloaded Images

See what images you have on your computer:

```bash
docker images
```

**Sample output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    1a2b3c4d5e6f   2 weeks ago   135MB
```

### 6. `docker rmi` - Remove an Image

Delete an image you no longer need:

```bash
docker rmi nginx
```

## Hands-On Mini-Project: Your First Web Application

Let's create a simple Python web application and containerize it!

### Step 1: Create Project Files

Create a new folder for your project:

```bash
mkdir my-docker-app
cd my-docker-app
```

### Step 2: Create a Simple Python Web App

Create a file called `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return """
    <h1>🐳 Hello from Docker!</h1>
    <p>This Python web app is running inside a Docker container!</p>
    <p>Congratulations on your first containerized application!</p>
    """

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 3: Create Requirements File

Create `requirements.txt`:

```
Flask==2.3.3
```

### Step 4: Create a Dockerfile

This is the recipe for building your container. Create a file called `Dockerfile` (no extension):

```dockerfile
# Start with a Python base image
FROM python:3.9-slim

# Set working directory inside the container
WORKDIR /app

# Copy requirements file
COPY requirements.txt .

# Install Python dependencies
RUN pip install -r requirements.txt

# Copy your application code
COPY app.py .

# Expose port 5000
EXPOSE 5000

# Command to run when container starts
CMD ["python", "app.py"]
```

### Step 5: Build Your Image

Build your custom image:

```bash
docker build -t my-python-app .
```

**What this does:**
- `-t my-python-app`: Give your image a name (tag)
- `.`: Use the current directory for build context

### Step 6: Run Your Container

Start your containerized app:

```bash
docker run -d -p 5000:5000 --name my-app my-python-app
```

Visit http://localhost:5000 to see your app running!

### Step 7: Verify It's Working

Check your running containers:

```bash
docker ps
```

View the logs:

```bash
docker logs my-app
```

### Step 8: Clean Up

When you're done experimenting:

```bash
docker stop my-app
docker rm my-app
docker rmi my-python-app
```

## Quick Reference Commands

Here's a cheat sheet for the commands we learned:

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run` | Create and start container | `docker run -d -p 8080:80 nginx` |
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers | `docker ps -a` |
| `docker stop` | Stop container | `docker stop container_name` |
| `docker rm` | Remove container | `docker rm container_name` |
| `docker images` | List images | `docker images` |
| `docker rmi` | Remove image | `docker rmi image_name` |
| `docker build` | Build image from Dockerfile | `docker build -t myapp .` |
| `docker logs` | View container logs | `docker logs container_name` |

## Common Flags and Options

- `-d`: Run in background (detached mode)
- `-p`: Port mapping (host:container)
- `--name`: Give container a custom name
- `-it`: Interactive mode with terminal
- `--rm`: Automatically remove container when it stops
- `-v`: Mount volumes (share files between host and container)

## Troubleshooting Tips

**Container won't start?**
- Check the logs: `docker logs container_name`
- Try running without `-d` to see error messages

**Port already in use?**
- Change the host port: `-p 8081:80` instead of `-p 8080:80`
- Stop conflicting services

**Permission denied on Linux?**
- Add yourself to docker group: `sudo usermod -aG docker $USER`
- Or use `sudo` before docker commands

**Can't connect to Docker daemon?**
- Make sure Docker Desktop is running (Windows/Mac)
- Start Docker service on Linux: `sudo systemctl start docker`

## What's Next?

Congratulations! You've learned Docker basics. Here are your next steps:

### 1. Docker Compose
Learn to manage multi-container applications with a single file:
```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  database:
    image: postgres
    environment:
      POSTGRES_PASSWORD: mypassword
```

### 2. Docker Hub
Share your images with the world:
- Create account at hub.docker.com
- Push images: `docker push username/image_name`
- Pull others' images: `docker pull username/image_name`

### 3. Advanced Topics to Explore
- **Volumes**: Persist data between container runs
- **Networks**: Connect containers together
- **Multi-stage builds**: Optimize image sizes
- **Docker in production**: Orchestration with Kubernetes or Docker Swarm

### 4. Practice Projects
- Containerize a database (MySQL, PostgreSQL)
- Create a multi-container app (web + database)
- Build a development environment for your favorite programming language

### 5. Learning Resources
- Docker's official documentation: docs.docker.com
- Docker's interactive tutorial: docker.com/play-with-docker
- Practice on Play with Docker (no installation needed)

## Final Thoughts

Docker might seem complex at first, but remember:
- **Containers are just isolated processes** running on your computer
- **Images are templates** for creating containers
- **Dockerfile is a recipe** for building images
- **Start simple** and gradually explore advanced features

You now have the foundation to start containerizing your applications and exploring the vast Docker ecosystem. The key is practice - try containerizing different applications and experimenting with various configurations.

Happy containerizing! 🐳
