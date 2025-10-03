# Docker - Containerization Platform

## Overview

Docker is an open-source platform for developing, shipping, and running applications in containers. Containers package up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.

### Key Features
- **Lightweight**: Containers share the host system's kernel
- **Portable**: Run anywhere Docker is installed
- **Isolated**: Containers are isolated from each other and the host
- **Version Control**: Docker images can be versioned
- **Ecosystem**: Large ecosystem of tools and services
- **DevOps Integration**: Integrates well with CI/CD pipelines

## Core Concepts

### Docker Architecture
- **Docker Engine**: Core component that creates and runs containers
- **Docker Daemon**: Background service that manages Docker objects
- **Docker Client**: Command-line interface (CLI) for interacting with Docker
- **Docker Registry**: Service for storing and distributing Docker images
- **Docker Images**: Read-only templates used to create containers
- **Docker Containers**: Runnable instances of Docker images

### Key Components
- **Images**: Immutable files that contain the application and its dependencies
- **Containers**: Running instances of images
- **Dockerfile**: Text file with instructions for building an image
- **Volumes**: Persistent storage for containers
- **Networks**: Enable containers to communicate with each other
- **Registry**: Stores and distributes Docker images

### Docker Objects
- **Images**: Templates for creating containers
- **Containers**: Running instances of images
- **Networks**: Virtual networks for containers
- **Volumes**: Persistent data storage
- **Services**: Define multi-container applications
- **Stacks**: Group of interrelated services

## Installation & Setup

### System Requirements
- **OS**: Linux, macOS, Windows
- **CPU**: 64-bit processor
- **Memory**: Minimum 4GB RAM (8GB recommended)
- **Virtualization**: Enabled in BIOS
- **Storage**: Minimum 25GB free space

### Installation Methods

#### Ubuntu/Debian
```bash
# Update package index
sudo apt update

# Install prerequisites
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker repository
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
sudo docker run hello-world
```

#### CentOS/RHEL
```bash
# Install prerequisites
sudo yum install -y yum-utils

# Add Docker repository
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo yum install docker-ce docker-ce-cli containerd.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
sudo docker run hello-world
```

#### macOS
```bash
# Download Docker Desktop for Mac
# Visit: https://www.docker.com/products/docker-desktop

# Install Docker Desktop
# Follow the installation wizard

# Verify installation
docker --version
docker run hello-world
```

#### Windows
```bash
# Download Docker Desktop for Windows
# Visit: https://www.docker.com/products/docker-desktop

# Enable Hyper-V and Containers features
# Run PowerShell as Administrator

# Install Docker Desktop
# Follow the installation wizard

# Verify installation
docker --version
docker run hello-world
```

## Dockerfile Basics

### Dockerfile Structure
```dockerfile
# Base image
FROM ubuntu:20.04

# Metadata
LABEL maintainer="your-email@example.com"
LABEL version="1.0"
LABEL description="My application container"

# Environment variables
ENV APP_HOME=/app
ENV NODE_ENV=production

# Working directory
WORKDIR $APP_HOME

# Copy files
COPY package*.json ./
COPY . .

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# Install application dependencies
RUN npm install

# Expose port
EXPOSE 3000

# Create user
RUN useradd -m appuser && chown -R appuser:appuser $APP_HOME
USER appuser

# Command to run
CMD ["npm", "start"]
```

### Dockerfile Instructions
- **FROM**: Specifies the base image
- **LABEL**: Adds metadata to the image
- **ENV**: Sets environment variables
- **WORKDIR**: Sets the working directory
- **RUN**: Executes commands during build
- **COPY**: Copies files from host to container
- **ADD**: Copies files and can extract archives
- **EXPOSE**: Exposes ports to the outside
- **CMD**: Default command to execute
- **ENTRYPOINT**: Configures container to run as executable
- **VOLUME**: Creates a mount point
- **USER**: Sets the user for the container
- **ARG**: Defines build-time variables

## Docker Commands

### Image Management
```bash
# Pull an image
docker pull nginx:latest

# List images
docker images
docker image ls

# Build an image
docker build -t my-app:1.0 .

# Build with Dockerfile in different location
docker build -f /path/to/Dockerfile -t my-app:1.0 .

# Tag an image
docker tag my-app:1.0 my-app:latest

# Remove an image
docker rmi my-app:1.0
docker image rm my-app:1.0

# Push image to registry
docker push my-app:1.0

# Save image to tar file
docker save -o my-app.tar my-app:1.0

# Load image from tar file
docker load -i my-app.tar

# Inspect image
docker inspect nginx:latest

# View image history
docker history nginx:latest
```

### Container Management
```bash
# Run a container
docker run -d --name my-container -p 8080:80 nginx:latest

# Run container interactively
docker run -it --name my-container ubuntu:latest /bin/bash

# List running containers
docker ps
docker container ls

# List all containers (including stopped)
docker ps -a
docker container ls -a

# Stop a container
docker stop my-container

# Start a container
docker start my-container

# Restart a container
docker restart my-container

# Remove a container
docker rm my-container
docker container rm my-container

# Force remove a running container
docker rm -f my-container

# Execute command in running container
docker exec -it my-container /bin/bash

# View container logs
docker logs my-container

# Follow container logs
docker logs -f my-container

# View container resource usage
docker stats my-container

# Inspect container
docker inspect my-container
```

### Volume Management
```bash
# Create a volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Run container with volume
docker run -d --name my-container -v my-volume:/app/data nginx:latest

# Run container with host mount
docker run -d --name my-container -v /host/path:/container/path nginx:latest
```

### Network Management
```bash
# Create a network
docker network create my-network

# List networks
docker network ls

# Inspect network
docker network inspect my-network

# Remove network
docker network rm my-network

# Run container with network
docker run -d --name my-container --network my-network nginx:latest

# Connect container to network
docker network connect my-network my-container

# Disconnect container from network
docker network disconnect my-network my-container
```

## Docker Compose

### Docker Compose File
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

### Docker Compose Commands
```bash
# Start services
docker-compose up

# Start services in detached mode
docker-compose up -d

# Stop services
docker-compose down

# Stop services and remove volumes
docker-compose down -v

# List services
docker-compose ps

# View service logs
docker-compose logs

# Follow service logs
docker-compose logs -f

# Build services
docker-compose build

# Rebuild services
docker-compose build --no-cache

# Run command in service
docker-compose exec web bash

# Scale services
docker-compose up -d --scale web=3
```

## Docker Networking

### Network Types
- **bridge**: Default network for containers
- **host**: Container shares host's network stack
- **none**: No network connectivity
- **overlay**: Multi-host networking
- **macvlan**: Assign MAC address to container

### Network Configuration
```bash
# Create bridge network
docker network create --driver bridge my-bridge-network

# Create overlay network
docker network create --driver overlay my-overlay-network

# Create network with subnet
docker network create --subnet=192.168.0.0/24 my-network

# Connect container to multiple networks
docker run -d --name my-container --network network1 nginx:latest
docker network connect network2 my-container
```

## Docker Storage

### Volume Types
- **Named Volumes**: Managed by Docker
- **Bind Mounts**: Host file or directory mounted into container
- **Tmpfs Mounts**: Stored in host memory

### Volume Management
```bash
# Create named volume
docker volume create my-data-volume

# Create volume with driver options
docker volume create --opt type=tmpfs --opt device=tmpfs my-tmpfs-volume

# Mount volume in container
docker run -d --name my-container -v my-data-volume:/data nginx:latest

# Mount read-only volume
docker run -d --name my-container -v my-data-volume:/data:ro nginx:latest

# Mount multiple volumes
docker run -d --name my-container \
  -v volume1:/data1 \
  -v volume2:/data2 \
  nginx:latest
```

## Docker Security

### Security Best Practices
- **Use Official Images**: Prefer official Docker images
- **Minimal Base Images**: Use minimal base images like Alpine
- **Non-root User**: Run containers as non-root user
- **Read-only Filesystem**: Mount filesystem as read-only when possible
- **Resource Limits**: Set memory and CPU limits
- **Network Isolation**: Use custom networks
- **Secrets Management**: Use Docker secrets for sensitive data

### Security Commands
```bash
# Run container with user
docker run -d --name my-container -u 1000:1000 nginx:latest

# Run container with read-only filesystem
docker run -d --name my-container --read-only nginx:latest

# Run container with memory limit
docker run -d --name my-container --memory=512m nginx:latest

# Run container with CPU limit
docker run -d --name my-container --cpus=0.5 nginx:latest

# Scan image for vulnerabilities
docker scan nginx:latest
```

## Docker Multi-Stage Builds

### Multi-Stage Dockerfile
```dockerfile
# Build stage
FROM node:16-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Docker Registry

### Private Registry Setup
```bash
# Run private registry
docker run -d -p 5000:5000 --name registry registry:2

# Push to private registry
docker tag my-app:1.0 localhost:5000/my-app:1.0
docker push localhost:5000/my-app:1.0

# Pull from private registry
docker pull localhost:5000/my-app:1.0
```

## Docker Monitoring

### Monitoring Commands
```bash
# View container resource usage
docker stats

# View system-wide information
docker info

# View Docker events
docker events

# View container logs
docker logs my-container

# View container inspect
docker inspect my-container
```

## Interview Questions

### Beginner Level
1. **What is Docker?**
   - Docker is a containerization platform that packages applications and their dependencies into containers

2. **What is the difference between Docker and virtual machines?**
   - Docker containers share the host OS kernel, while VMs have their own OS kernel

3. **What is a Docker image?**
   - A Docker image is a read-only template used to create containers

4. **What is a Docker container?**
   - A Docker container is a runnable instance of a Docker image

### Intermediate Level
1. **What is the difference between CMD and ENTRYPOINT?**
   - CMD provides default command and parameters, ENTRYPOINT configures container to run as executable

2. **What is Docker Compose?**
   - Docker Compose is a tool for defining and running multi-container applications

3. **What is the difference between COPY and ADD in Dockerfile?**
   - COPY copies files from host to container, ADD can also extract archives and download from URLs

4. **What are Docker volumes?**
   - Docker volumes are persistent storage that survives container restarts

### Advanced Level
1. **What is multi-stage build in Docker?**
   - Multi-stage builds use multiple FROM statements in a single Dockerfile to create smaller, more secure images

2. **How does Docker networking work?**
   - Docker provides various network drivers (bridge, host, overlay, etc.) for container communication

3. **What is the difference between Docker Swarm and Kubernetes?**
   - Docker Swarm is Docker's native orchestration tool, while Kubernetes is a more feature-rich container orchestration platform

4. **How do you secure Docker containers?**
   - Use non-root users, read-only filesystems, resource limits, network isolation, and regular security scanning

## Best Practices

### Dockerfile Best Practices
- **Use Specific Tags**: Avoid using `latest` tag
- **Minimize Layers**: Combine RUN commands to reduce layers
- **Use .dockerignore**: Exclude unnecessary files
- **Clean Up**: Remove cache and temporary files
- **Use Multi-Stage Builds**: Reduce final image size
- **Order Instructions**: Put frequently changing instructions at the end

### Container Management
- **Resource Limits**: Set memory and CPU limits
- **Health Checks**: Implement health checks
- **Logging**: Use proper logging strategies
- **Monitoring**: Monitor container performance
- **Backup**: Regular backup of volumes and data

### Security Best Practices
- **Minimal Images**: Use minimal base images
- **Regular Updates**: Keep images updated
- **Security Scanning**: Scan images for vulnerabilities
- **Access Control**: Use proper access controls
- **Network Security**: Use network policies and firewalls

## Troubleshooting

### Common Issues
```bash
# Check Docker daemon status
sudo systemctl status docker

# Restart Docker daemon
sudo systemctl restart docker

# Check Docker logs
sudo journalctl -u docker

# Check disk space
df -h

# Clean up unused images, containers, networks
docker system prune -a

# Check container logs
docker logs my-container

# Check container status
docker inspect my-container
```

### Debugging Commands
```bash
# Run container in debug mode
docker run -it --entrypoint /bin/bash nginx:latest

# Check container processes
docker top my-container

# Check container network configuration
docker exec my-container ip addr

# Check container disk usage
docker exec my-container df -h

# Check container memory usage
docker stats my-container
```

## Resources

### Official Documentation
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Registry Documentation](https://docs.docker.com/registry/)

### Learning Resources
- [Docker Tutorial](https://docs.docker.com/get-started/)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Security](https://docs.docker.com/engine/security/)

### Community
- [Docker Community](https://www.docker.com/community)
- [Docker Forums](https://forums.docker.com/)
- [Docker GitHub](https://github.com/docker)