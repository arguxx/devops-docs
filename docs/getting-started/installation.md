# Installation Guide

Detailed installation instructions for all required tools and dependencies.

## System Requirements

### Hardware Requirements

-   **CPU**: 2+ cores recommended
-   **RAM**: 8GB minimum, 16GB recommended
-   **Storage**: 20GB free space
-   **Network**: Stable internet connection

### Operating System Support

=== "macOS" - macOS 10.15+ (Catalina or newer) - Homebrew package manager recommended

=== "Linux" - Ubuntu 20.04+ / CentOS 8+ / RHEL 8+ - Root or sudo access required

=== "Windows" - Windows 10/11 with WSL2 - PowerShell 5.1+ or PowerShell Core 7+

## Core Tools Installation

### 1. Git

=== "macOS"
```bash # Using Homebrew
brew install git

    # Verify installation
    git --version
    ```

=== "Linux"
```bash # Ubuntu/Debian
sudo apt update && sudo apt install git

    # CentOS/RHEL
    sudo yum install git

    # Verify installation
    git --version
    ```

=== "Windows"
```powershell # Using Chocolatey
choco install git

    # Or download from https://git-scm.com/download/win
    ```

### 2. Docker

=== "macOS"
`bash
    # Download Docker Desktop from https://docker.com/products/docker-desktop
    # Or using Homebrew
    brew install --cask docker
    `

=== "Linux"
```bash # Ubuntu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

    # Add user to docker group
    sudo usermod -aG docker $USER
    newgrp docker
    ```

=== "Windows"
`powershell
    # Download Docker Desktop from https://docker.com/products/docker-desktop
    # Enable WSL2 integration
    `

### 3. Kubernetes CLI (kubectl)

```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Windows
choco install kubernetes-cli
```

### 4. Terraform

```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
sudo apt update && sudo apt install terraform

# Windows
choco install terraform
```

## Optional Tools

### AWS CLI

```bash
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows
choco install awscli
```

### Azure CLI

```bash
# macOS
brew install azure-cli

# Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Windows
winget install Microsoft.AzureCLI
```

## Development Environment Setup

### IDE Recommendations

| IDE               | Best For            | Extensions                    |
| ----------------- | ------------------- | ----------------------------- |
| **VS Code**       | General development | Docker, Kubernetes, Terraform |
| **IntelliJ IDEA** | Java/JVM apps       | Docker, Cloud Code            |
| **Vim/Neovim**    | Command-line lovers | coc-docker, vim-terraform     |

### Recommended VS Code Extensions

```json
{
    "recommendations": [
        "ms-vscode.vscode-docker",
        "ms-kubernetes-tools.vscode-kubernetes-tools",
        "hashicorp.terraform",
        "ms-vscode.azure-cli-tools",
        "amazonwebservices.aws-toolkit-vscode"
    ]
}
```

## Project Structure

```
devops-project/
├── .github/
│   └── workflows/          # CI/CD pipelines
├── infrastructure/
│   ├── terraform/          # Infrastructure as Code
│   └── kubernetes/         # K8s manifests
├── applications/
│   ├── frontend/           # React/Vue.js app
│   ├── backend/            # API service
│   └── database/           # Database configs
├── monitoring/
│   ├── prometheus/         # Metrics collection
│   └── grafana/           # Dashboards
├── scripts/
│   ├── deploy.sh          # Deployment scripts
│   └── setup.sh           # Environment setup
├── docker-compose.yml     # Local development
├── .env.example          # Environment template
└── README.md
```

## Verification

Run this script to verify all tools are installed correctly:

```bash
#!/bin/bash
echo "🔍 Verifying DevOps Tools Installation..."

tools=("git" "docker" "kubectl" "terraform")
for tool in "${tools[@]}"; do
    if command -v $tool &> /dev/null; then
        version=$($tool --version | head -n1)
        echo "✅ $tool: $version"
    else
        echo "❌ $tool: Not installed"
    fi
done

echo "🐳 Docker daemon status:"
if docker info &> /dev/null; then
    echo "✅ Docker daemon is running"
else
    echo "❌ Docker daemon is not running"
fi
```

## Troubleshooting

### Common Issues

!!! bug "Permission Denied"
`bash
    # Add user to docker group
    sudo usermod -aG docker $USER
    newgrp docker
    `

!!! bug "kubectl: command not found"
`bash
    # Add to PATH in ~/.bashrc or ~/.zshrc
    export PATH=$PATH:/usr/local/bin
    `

!!! bug "Terraform not found"
`bash
    # Verify installation path
    which terraform
    echo $PATH
    `

For more issues, see our [troubleshooting guide](../troubleshooting.md).
