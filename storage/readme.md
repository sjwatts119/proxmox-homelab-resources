# Storage

## Introduction

## Prerequisites

## Setup

### Sharing Storage with an LXC

### Step 1: Access the Proxmox Root Shell
> [!IMPORTANT]
> Ensure your LXC has been shut down before proceeding.

### Step 2: Create Storage Bind Mount

```bash
nano /etc/pve/lxc/*YOUR_LXC_ID*.conf
```
> [!IMPORTANT]
> Replace `*YOUR_LXC_ID*` with the ID of the LXC.

Add the following line to the configuration file to create a new bind mount:

```plaintext
mp0: /mnt/pve/*YOUR_STORAGE_DIRECTORY*,mp=/mnt/*YOUR_MOUNT_POINT*
```
> [!IMPORTANT]
> Replace `*YOUR_STORAGE_DIRECTORY*` with the storage directory you set up.
> 
> Replace `*YOUR_MOUNT_POINT*` with the directory you will use to access the storage within the LXC. If you were to call it `nas`, you would access the storage at `/mnt/nas` within the LXC.

### Step 3: Access the LXC Shell
> [!IMPORTANT]
> Ensure your LXC is running before proceeding.

Enter the new LXC's shell via the Proxmox web interface or SSH:

### Verify the Bind Mount was Successful
```bash
ls -l /mnt/*YOUR_MOUNT_POINT*
```
> [!IMPORTANT]
> Replace `*YOUR_MOUNT_POINT*` with the mount point you specified in Step 2.
 
> [!TIP]
> If the bind mount was successful, you should see the contents of your storage directory listed.

### Step 4: Create Service User

If you haven't already created a dedicated user account on your LXC, you should do so now.

```bash
adduser *YOUR_USERNAME*
```
> [!IMPORTANT]
> Replace `*YOUR_USERNAME*` with a username of your choice.

> [!TIP]
> You can press Enter to skip optional fields like Full Name, Room Number, etc.

### Step 5: Create User Group
```bash
groupadd storagegroup -g *YOUR_GROUP_ID*
```
> [!IMPORTANT]  
> Ensure you replace `*YOUR_GROUP_ID*` with the group ID you set up for the Storage directory. Failure to do so may result in permission issues later.

### Step 6: Add Users to the NAS User Group
```bash
usermod -aG storagegroup root
usermod -aG storagegroup *YOUR_USERNAME*
```

> [!IMPORTANT]
> Replace *YOUR_USERNAME* with the username you created in Step 3

