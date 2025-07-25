# Samba File Server

## Introduction

## Prerequisites

This guide assumes you have followed the **[Storage Setup Guide](TODO ADD LINK)** and have configured storage permissions on the Proxmox host properly. This will allow you to have a fully modular storage setup which is entirely external of this LXC.

## Installation

### Step 1: Install Docker LXC Template

Install the **[Docker LXC Template](https://community-scripts.github.io/ProxmoxVE/scripts?id=docker)** from the Proxmox Community Scripts site. Alternatively, spin up an Ubuntu LXC and install Docker Compose manually.

**Recommended specifications:**
- CPU: 1 core
- RAM: 1024MB
- Network: Static IP (note this for later if you don't manage your own DHCP server)

### Step 2: Create Storage Bind Mount
Follow the **[Sharing Storage with an LXC](TODO ADD LINK)** steps to create a storage bind mount for your LXC. This will allow the LXC to access the storage directory you set up.

### Step 3: Access the LXC Shell
> [!IMPORTANT]
> Ensure your new LXC is running before proceeding.

### Step 4: Add your User to the Docker Group
```bash
usermod -aG docker *YOUR_USERNAME*
```
> [!IMPORTANT]
> Replace *YOUR_USERNAME* with the username you created in Step 3

### Step 5: Create Docker Compose File
Create a `docker-compose.yml` file in your LXC's home directory or any other directory you prefer. This file will define the services you want to run, such as Samba.

Refer to the [Samba Docker Compose Example](ADD LINK) for a working example.

### Step 6: Create .env File
Create a `.env` file in the same directory as your `docker-compose.yml`. This file will contain a number of environment variables that will be used by the Docker Compose file.

Refer to the [Samba .env Example](ADD LINK) for a working example.

> [!CAUTION]
> The credentials provided in this file will be the ones you use to access the NAS. Ensure you set a strong password and keep this file secure.

> [!IMPORTANT]
> Ensure the USER_GID variable in the `.env` file matches the GID of the storagegroup you created in step 2. Failing to do so will result in you not being able to write files to the NAS.

### Step 7: Start the Docker Container
Navigate to the directory where your `docker-compose.yml` and `.env` files are located, and run the following command to start the Samba service:

```bash
docker-compose up -d
```

> [!TIP]
> You can use `docker-compose logs -f` to follow the logs of the running container.

### Step 8: Access the Samba Share
You can now access the Samba share from any device on your network. Use the IP address of your LXC and the credentials you set in the `.env` file to connect.

### Step 10: Verify Permissions
Ensure you can create, read, and delete files in the Samba share. If you encounter permission issues, double-check the GID in your `.env` file and ensure it matches the GID of the storage group you created earlier.