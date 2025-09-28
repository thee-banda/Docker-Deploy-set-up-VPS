# Docker Deployment Toolkit for Ubuntu VPS

Clear, repeatable instructions for installing Docker Engine and the Docker Compose plugin on an Ubuntu virtual private server. Use this playbook when you want a ready-to-deploy host for multiple containerised workloads (Node.js, React, Nginx, databases, and more).

## Why This Guide
This handbook turns a fresh Ubuntu VPS into a reliable Docker host. Each step aligns with Docker's official documentation while highlighting the context you need to understand what the command is doing.

> **Tip:** Run the commands over SSH with a sudo-capable user. Keep the session open until you have confirmed Docker is working.

## Table of Contents
- [Before You Start](#before-you-start)
- [At-a-Glance Checklist](#at-a-glance-checklist)
- [Installation Workflow](#installation-workflow)
- [Post-install Tasks](#post-install-tasks)
- [Deploy Containers](#deploy-containers)
- [Maintenance Commands](#maintenance-commands)
- [Troubleshooting](#troubleshooting)
- [Useful References](#useful-references)

## Before You Start
- Ubuntu 20.04, 22.04, or newer with outbound internet access
- User account that can run commands with `sudo`
- Basic familiarity with terminal usage and SSH

## At-a-Glance Checklist
- **Step 1:** `sudo apt update && sudo apt upgrade -y` (refresh package metadata and apply security updates)
- **Step 2:** `sudo apt install -y ca-certificates curl gnupg lsb-release` (install dependencies needed during Docker setup)
- **Step 3:** `sudo install -m 0755 -d /etc/apt/keyrings` (create a secure keyring directory for Docker's GPG key)
- **Step 4:** `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg` (download and store Docker's signing key)
- **Step 5:** `sudo chmod a+r /etc/apt/keyrings/docker.gpg` (allow apt to read the key file)
- **Step 6:** `echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null` (register Docker's stable repository)
- **Step 7:** `sudo apt update` (refresh apt metadata after adding the repository)
- **Step 8:** `sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` (install Docker Engine, CLI tooling, Buildx, and the Compose plugin)

> Copy the commands above if you only need the essentials, or follow the detailed workflow below for more context.

## Installation Workflow
### Step 1 - Refresh the package index
```bash
sudo apt update && sudo apt upgrade -y
```
Keeps the system patched before installing new packages.

### Step 2 - Install setup dependencies
```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```
Provides certificate authorities, HTTP transfer tools, and distribution details required in later steps.

### Step 3 - Add Docker's GPG key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
Creates a dedicated key directory, downloads Docker's public key, and exposes it to the package manager.

### Step 4 - Register Docker's apt repository
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Adds the official Docker repository matching your Ubuntu release to the apt sources list.

### Step 5 - Install Docker Engine and Compose
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Installs Docker Engine, the CLI, containerd runtime, Buildx, and the Compose plugin.

### Step 6 - Verify the installation
```bash
docker --version
docker compose version
```
Both commands should report versions without errors.

## Post-install Tasks
- Add your user to the `docker` group so you can run Docker without `sudo`:
  ```bash
  sudo usermod -aG docker $USER
  newgrp docker
  ```
- Run the hello-world image to confirm the engine can pull and execute containers:
  ```bash
  docker run hello-world
  ```
- (Optional) Enable Docker at boot:
  ```bash
  sudo systemctl enable docker
  ```

## Deploy Containers
1. Create or copy a `docker-compose.yml` file into your project directory.
2. Start services in detached mode:
   ```bash
   docker compose up -d
   ```
3. Inspect container health and logs:
   ```bash
   docker ps
   docker compose logs -f
   ```
4. Shut down and clean up when required:
   ```bash
   docker compose down
   ```

## Maintenance Commands
- Check service status: `sudo systemctl status docker`
- Review daemon logs: `sudo journalctl -u docker --since "1 hour ago"`
- Upgrade Docker packages: `sudo apt update && sudo apt install --only-upgrade docker-ce docker-ce-cli containerd.io`

## Troubleshooting
- `permission denied while trying to connect to the Docker daemon`: confirm your user is in the `docker` group, then log out/in or run `newgrp docker`.
- `certificate has expired` or GPG errors: verify system time with `timedatectl status`, then repeat the key download commands from Step 3.
- `docker compose` not found: check that the `docker-compose-plugin` package installed successfully; rerun Step 5 if needed.
- Slow downloads from Docker registry: switch to a mirror closer to your region or configure a pull-through cache.

## Useful References
- Docker Engine (Ubuntu): https://docs.docker.com/engine/install/ubuntu/
- Post-install steps: https://docs.docker.com/engine/install/linux-postinstall/
- Docker Compose plugin: https://docs.docker.com/compose/
