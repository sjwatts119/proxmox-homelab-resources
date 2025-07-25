# Storage

## Introduction

## Prerequisites

## Setup

### Step 1: Create a ZFS Volume
In your Proxmox web interface, navigate to the storage section and create a new ZFS volume. This will be the storage directory that you will share with your LXC.

You can choose multiple drives in a RAID configuration, or a single drive. Ensure that the ZFS volume is large enough to accommodate the data you plan to store.

> [!IMPORTANT]
> Choose a good name for your ZFS volume as you will need to reference it later in this setup.

### Step 2: Access Proxmox Host Shell

### Step 3: Ensure the Storage Directory is Mounted

The ZFS volume should be available at the root directory of your Proxmox host. You can check this by running the following command:

```bash
ls /
```

You should see a folder with the name of your ZFS volume listed.

### Step 4: Create a Storage Directory
It is recommended to create a dedicated directory for your storage. This will help keep things organized and make it easier to manage permissions.

```bash
mkdir /*YOUR_ZFS_VOLUME*/storage
```
> [!IMPORTANT]
> Replace `*YOUR_ZFS_VOLUME*` with the folder name of your mounted ZFS volume as seen in Step 3.

### Step 5: Set Permissions for the Storage Directory

> [!IMPORTANT]
> Proxmox maps UIDs/GIDs inside the LXC to different ones in the host. This is to ensure that host level permissions are never acquirable from inside an unprivileged LXC, even if it is compromised.
> 
> Proxmox will map any IDs inside an unprivileged LXC to a range between 100000-165535 on the host.
> 
> For example, if you have a group in the LXC with a GID of `1234`, the permissions of this group will be mapped to the GID of `101234` on the host.

Here is where we will decide on the required GID for accessing the storage directory. For this example, we will use `2468` as the GID inside our LXCs, but you can choose any GID you prefer as long as it does not conflict with existing groups.

This means we will need to set permissions on the host as GID `102468` for this example to account for the Proxmox UID/GID mapping.

Now we know the GID we want to use, we can recursively set the permissions for the storage directory:
```bash
chown -R 102468:102468 /*YOUR_ZFS_VOLUME*/storage
chmod -R 770 /*YOUR_ZFS_VOLUME*/storage
```

> [!IMPORTANT]
> Replace `*YOUR_ZFS_VOLUME*` with the folder name of your mounted ZFS volume as seen in Step 3.

We should also ensure that any new files created in this directory will inherit the same permissions. This can be done by setting the setgid bit on the directory:

```bash
find /*YOUR_ZFS_VOLUME*/storage -type d -exec chmod g+s {} \;
```
> [!IMPORTANT]
> As above, replace `*YOUR_ZFS_VOLUME*` with the folder name of your mounted ZFS volume.

### Step 6: Done!
You have now set up a storage directory on your Proxmox host that can be shared with an unprivileged LXC. The next steps will guide you through sharing this storage with your LXC.

## Sharing Storage with an Unprivileged LXC

### Step 1: Access the Proxmox Host Shell
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
> Replace `*YOUR_USERNAME*` with the username you created in Step 3

### Step 7: Test Permissions
To ensure everything is set up correctly, you can test the permissions by creating a file in the storage directory:
```bash
touch /