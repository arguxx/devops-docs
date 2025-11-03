# Quick Start Guide

Get up and running with our DevOps environment in minutes!

## Prerequisites

Before you begin, ensure you have the following installed:

-   [x] **Git** - Version control system
-   [x] **Docker** - Container platform
-   [x] **kubectl** - Kubernetes CLI tool
-   [x] **Terraform** - Infrastructure as Code tool

## 🚀 5-Minute Setup

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/devops-project.git
cd devops-project
```

### 2. Set Up Environment

```bash
# Copy environment template
cp .env.example .env

# Edit with your values
vim .env
```

### 3. Start Local Development

```bash
# Start all services
docker-compose up -d

# Verify services are running
docker-compose ps
```

### 4. Access Applications

| Service     | URL                   | Description      |
| ----------- | --------------------- | ---------------- |
| Web App     | http://localhost:3000 | Main application |
| API         | http://localhost:8080 | REST API         |
| Database UI | http://localhost:8081 | Database admin   |

## ✅ Verification Steps

Run these commands to verify your setup:

```bash
# Check Docker containers
docker ps

# Test API health
curl http://localhost:8080/health

# Check logs
docker-compose logs -f app
```

## 🎯 Next Steps

1. **Explore the codebase** - Check out our [project structure](installation.md#project-structure)
2. **Set up CI/CD** - Follow our [pipeline guide](../guides/cicd.md)
3. **Deploy to staging** - Use our [deployment guide](../guides/deployment.md)

## 🆘 Troubleshooting

!!! warning "Common Issues"

    **Port conflicts?** Check if ports 3000, 8080, or 8081 are already in use:
    ```bash
    lsof -i :3000
    ```

    **Docker issues?** Restart Docker service:
    ```bash
    sudo systemctl restart docker
    ```

Need more help? Check our [detailed installation guide](installation.md) or [troubleshooting section](../troubleshooting.md).
