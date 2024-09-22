# Creating a Persistent Disk

## Overview

In this lab, you will learn how to create a persistent disk in Google Cloud Platform (GCP) and attach it to a Compute Engine virtual machine (VM). Persistent disks are durable storage devices that function similarly to physical hard drives, with the benefit of existing independently from the VM to which they are attached. This ensures that the data on the disk persists even if the VM is deleted.

## Objectives

By the end of this lab, you will:

- Create a new Compute Engine VM instance.
- Create and attach a persistent disk to the VM.
- Format and mount the persistent disk on the VM.
- Configure the disk to automatically mount on system restart.

## Prerequisites

- Basic understanding of GCP and Compute Engine.
- Familiarity with Linux commands and text editors (e.g., `nano`).
  
## Estimated Time

Approximately **30 minutes**.

## Instructions

### Task 1: Create a New Compute Engine Instance

1. **Set the default zone**:
   Open Cloud Shell and set the default zone and region:
   ```bash
   gcloud config set compute/zone us-east4-b
   gcloud config set compute/region us-east4
   ```

2. **Create a new VM instance**:
   Run the following command to create a new VM instance named `gcelab`:
   ```bash
   gcloud compute instances create gcelab --zone $ZONE --machine-type e2-standard-2
   ```
   - This creates a new VM in the specified zone with the machine type `e2-standard-2`.

### Task 2: Create a New Persistent Disk

1. **Create a 200 GB persistent disk**:
   Run the following command to create a persistent disk named `mydisk` with a size of 200 GB:
   ```bash
   gcloud compute disks create mydisk --size=200GB --zone $ZONE
   ```
   - The newly created disk will be available for attachment to the VM.

### Task 3: Attach the Disk to the VM

1. **Attach the persistent disk**:
   Attach the disk `mydisk` to the VM instance `gcelab`:
   ```bash
   gcloud compute instances attach-disk gcelab --disk mydisk --zone $ZONE
   ```

2. **SSH into the VM**:
   SSH into the VM instance to format and mount the disk:
   ```bash
   gcloud compute ssh gcelab --zone $ZONE
   ```

3. **Verify the disk**:
   List the attached disk devices:
   ```bash
   ls -l /dev/disk/by-id/
   ```
   - The attached disk will appear as `scsi-0Google_PersistentDisk_persistent-disk-1`.

### Task 4: Format and Mount the Disk

1. **Create a mount point**:
   Create a directory to serve as the mount point for the disk:
   ```bash
   sudo mkdir /mnt/mydisk
   ```

2. **Format the disk**:
   Format the disk with the `ext4` filesystem:
   ```bash
   sudo mkfs.ext4 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1
   ```

3. **Mount the disk**:
   Mount the disk to the `/mnt/mydisk` directory:
   ```bash
   sudo mount -o discard,defaults /dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1 /mnt/mydisk
   ```

### Task 5: Automatically Mount the Disk on Reboot

1. **Edit the `fstab` file**:
   Open the `/etc/fstab` file for editing:
   ```bash
   sudo nano /etc/fstab
   ```

2. **Add the disk entry**:
   Add the following line to ensure the disk is automatically mounted on system restart:
   ```
   /dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1 /mnt/mydisk ext4 defaults 1 1
   ```

3. **Save and exit**:
   Press `CTRL+O`, then `ENTER` to save the changes, and press `CTRL+X` to exit `nano`.

### Task 6: Test Your Knowledge (Quiz)

1. **Question 1: Can you prevent the destruction of an attached persistent disk when the instance is deleted?**
   - Yes, you can prevent the deletion by deselecting the option **Delete boot disk when instance is deleted** during instance creation.

2. **Question 2: Reorder the following steps for migrating data from a persistent disk to another region**:
   - The correct order is: Unmount file system(s), Create snapshot, Create disk, Attach disk, Create instance.

