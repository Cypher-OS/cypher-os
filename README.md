# Cypher OS
Cypher is an independent Linux-based operating system that offers a couple of novel features.

## Installation Guide

**NOTE: If you are an experienced user who uses Arch Linux or Gentoo (or similar ones). You already know what to do bro. Just login with the given username password and install the OS.**

### Download files & verify integrity
1. Download the given cypher.iso file and sha256sum.txt
2. After downloading, verify the file with the given sha256sum.
   
   ```
   sha256sum -c sha256sum.txt
   ``` 
   If it says,
   ```
   cypher.iso: OK
   ```
   You're good to go. If not, delete both the files and download them again from the official 'Releases' page.

### Prepare the installation media
1. You need a USB Drive (minimum 4GB) to create a bootable media.
   
2. Use a tool to flash the drive. <br>
   - On Linux, run the command (replace /dev/sdx with the actual path of your drive)
   ```
   sudo dd if=cypher.iso of=/dev/sdX bs=4M status=progress oflag=sync
   ```
   - On Windows, use a graphical tool like **BalenaEtcher** or **Rufus** by selecting the downloaded iso file as source image.

### BIOS/UEFI Setup and Boot
1. Insert the newly created bootable USB drive into the target computer.
   
2. Reboot the computer.

3. Immediately press the key to enter the Boot Menu or BIOS/UEFI Setup. 
    - Common keys are `F2`, `F10`, `F12`, `Del`, or `Esc`.

4. In the Boot Menu, select the option for your USB drive (it might be listed under its brand name or as "UEFI USB Drive").

5. Your computer should now boot into the Cypher OS Live Environment.

### Manual OS installation
#### Login 
1. Once you boot into the OS, you will be prompted for username and password. We provide a default user `neo` with all the essential permissions already set to make the installation process easier.
2. Login by entering `neo` as the username and `cypher` as the password.
3. After you login successfully, try executing `hyprland` if you are using a intel device with i915 graphics driver. If not, you can continue in the tty environment for the installation process.
4. Make the font bigger by running,
   ```
   setfont latarcyrheb-sun32.psfu.gz
   ```

#### Disk Partitioning
1. Identify your target drive:
   ```
   fdisk -l
   ```
2. Start Partitioning: `fdisk /dev/sdX` (replace sdX with your drive)
3. Create Partitions (Example Partitions):
   - EFI Partition (`/dev/sdX1`): maybe `512MB` (Type: EFI System). Only create this if you don't have already have an EFI system. Don't create it or overwrite it.
   - Swap Partition (`/dev/sdX2`): Equal to your RAM (Type: Linux Swap) If it already exists, leave it as is.
   - Root Partition (`/dev/sdX3`): Remaining space (Type: Linux Filesystem)
4. Format the partitions.
   ```
   mkfs.ext4 /dev/sdX3       # Root
   ```
5. Mount the partitions.
   ```
   mount /dev/sdX3 /mnt/fs
   ```
   
   
   
