# Samba File Server

## Introduction

Setting up a NAS was one of the primary purposes I had for my homelab, but I ran into quite a few issues with sharing the data between other containers.

This setup allows you to mount your NAS storage directory directly in any other LXC you create without any permissions issues. 

This is particularly useful for media servers like Jellyfin or Plex where you might want to upload media files directly to the NAS and have them automatically available in your media server.

I'm using the **[dockurr/samba](https://github.com/dockur/samba)** Docker image for a quick and easy setup.

## Prerequisites

This guide assumes you have followed the **[Storage Setup Guide](../storage/readme.md)** and have configured storage permissions on the Proxmox host properly. This will allow you to have a fully modular storage setup which is entirely external of this LXC.

## Setup

### Step 1: Set Up A New LXC

Install the **[Docker LXC Template](https://community-scripts.github.io/ProxmoxVE/scripts?id=docker)** from the Proxmox Community Scripts site. Alternatively, spin up an Ubuntu LXC and install Docker Compose manually.

**Recommended specifications:**
- CPU: 1 core
- RAM: 2048MB
- Network: Static IP (optional, but recommended for easier access if you don't manage your own DHCP server)
- Unprivileged

### Step 2: Set up Storage Bind Mount and Configure User Permissions
Follow the **[Sharing Storage with an LXC](../storage/readme.md#sharing-storage-with-an-lxc)** steps to create a storage bind mount for your LXC. This will allow the LXC to access the storage directory you set up.

Ensure you have created a user and group for the LXC that matches the GID of the storage group you created in **[Step 5 of "Sharing Storage with an LXC"](../storage/readme.md#step-5-create-user-group)**.

### Step 3: Access the LXC Shell
> [!IMPORTANT]
> Ensure your new LXC is running before proceeding.

### Step 4: Add Your User To The Docker Group
```bash
usermod -aG docker *YOUR_USERNAME*
```
> [!IMPORTANT]
> Replace `*YOUR_USERNAME*` with the username you created in Step 3

### Step 5: Create Docker Compose File
Create a `docker-compose.yml` file in your LXC's home directory or any other directory you prefer. This file will define the services you want to run, such as Samba.

```bash
nano docker-compose.yml
```

Refer to the [Samba Docker Compose Example](../smb-nas/docker-compose.yml) for a working example.

### Step 6: Create .env File
Create a `.env` file in the same directory as your `docker-compose.yml`. This file will contain a number of environment variables that will be used by the Docker Compose file.

```bash
nano .env
```

Refer to the [Samba .env Example](../smb-nas/.env) for a working example.

> [!CAUTION]
> The credentials provided in this file will be the ones you use to access the NAS. Ensure you set a strong password and keep this file secure.

> [!IMPORTANT]
> Ensure the USER_GID variable in the `.env` file matches the GID of the storage group you created in **[Step 5 of "Sharing Storage with an LXC"](../storage/readme.md#step-5-create-user-group)**. Failing to do so will result in your NAS being read-only.

### Step 7: Start The Docker Container
Navigate to the directory where your `docker-compose.yml` and `.env` files are located, and run the following command to start the Samba service:

```bash
docker-compose up -d
```

> [!TIP]
> You can use `docker-compose logs -f` to follow the logs of the running container.

### Step 8: Access The Samba Share
You can now access the Samba share from any device on your network. Use the IP address of your LXC and the credentials you set in the `.env` file to connect.

### Step 9: Verify Permissions
Ensure you can create, read, and delete files in the Samba share. If you encounter permission issues, double-check the GID in your `.env` file and ensure it matches the GID of the storage group you created earlier.