# Cypher OS
Cypher is an independent Linux-based operating system that offers a couple of novel features.

## Installation Guide

**NOTE: If you are an experienced user who uses Arch Linux or Gentoo (or similar ones). You already know what to do. Just login with the given username password and install the OS.**

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
   - On Linux, run the command (replace /dev/sdx with the actual path of your drive) <br>
     
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
1. Once you boot into the OS, you will be prompted for username and password. A default user `neo` is provided with all the essential permissions already set to make the installation process easier. `neo` is a part of `wheel` group with sudo priviliges.

2. Login by entering `neo` as the username and `cypher` as the password. 

3. After you login successfully, try executing `hyprland` if you are using a intel device with i915 graphics driver. If not, you can continue in the tty environment for the installation process.

4. Make the font bigger by running,
   ```
   setfont latarcyrheb-sun32.psfu.gz
   ```
5. You can connect to the Internet using NetworkManager
   

#### Disk Partitioning
1. Identify your target drive: use `lsblk` or `fdisk -l` or a tool like `cfdisk`. For beginners, `cfdisk` is recommended.
2. Start Partitioning: `fdisk /dev/sdX` (replace sdX with your drive)
3. Create Partitions (Example Partitions):
   - EFI Partition (`/dev/sdX1`): maybe `512MB` (Type: EFI System). Only create this if you don't already have an EFI system. Don't create or overwrite it.
   - Swap Partition (`/dev/sdX2`): Equal to your RAM (Type: Linux Swap) If it already exists, leave it as is.
   - Root Partition (`/dev/sdX3`): Remaining space (Type: Linux Filesystem)
4. Format the partitions.
   ```
   mkfs.ext4 /dev/sdX3     # Linux Filesystem
   ```
   If you've downloaded the experimental version or if you want to try `tenet` you need a `btrfs` filesystem, run this command instead
   ```
   mkfs.btrfs /dev/sdX3
   ```
   
6. Mount the partitions.
   ```
   mount /dev/sdX3 /mnt/fs
   ```
   Ensure `/mnt/fs` is mounted correctly before going to the next step.

7. Copy the root filesystem to your drive. This is the crucial phase of the installation. Do it correctly.
   ```
   rsync -aAXHv --progress --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /mnt/fs/
   ```

Now you have successfully installed Cypher to your drive. 

### Chroot and Setup
1. Bind mount the essential pseudo filesystems
   ```
   sudo mount -v --bind /dev /mnt/fs/dev
   
   sudo mount -vt devpts devpts -o gid=5,mode=0620 /mnt/fs/dev/pts
   sudo mount -vt proc proc /mnt/fs/proc
   sudo mount -vt sysfs sysfs /mnt/fs/sys
   sudo mount -vt tmpfs tmpfs /mnt/fs/run

   if [ -h /mnt/fs/dev/shm ]; then
     sudo install -v -d -m 1777 /mnt/fs$(realpath /dev/shm)
   else
    sudo mount -vt tmpfs -o nosuid,nodev tmpfs /mnt/fs/dev/shm
   fi
   ```
2. Chroot into your drive. (root's password is `root`)
   
   ```
   sudo chroot /mnt/fs /usr/bin/env -i   \
   HOME=/root                 \
   TERM="xterm"                \
   PS1='(cypher chroot) \u:\w\$ ' \
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
   /bin/bash --login
   ```

3. Set your timezone. (The default is set to India.)
   - List the available timezones and choose your timezone

      ```
      ls /usr/share/zoneinfo/
      ```
   
   - For example, if you are in Casey, Antarctica (weird place to live) the file path would be <br>
   `/usr/share/zoneinfo/Antarctica/Casey`

   - Set the timezone using a symlink
     ```
     ln -sf /usr/share/zoneinfo/Antarctica/Casey /etc/localtime
     ```
   - Update the hardware clock
     ```
     hwclock --systohc
     ```
5. Install Bootloader and Configure GRUB
   - For UEFI Systems
      ```
      grub-install --target=x86_64-efi --bootloader-id=Cypher
      ```
   - For BIOS Systems
      ```
      grub-install --target=i386-pc /dev/sdX
      ```
   - Ensure the `/etc/default/grub` file is present, if not create and edit it. You can clone this repo, and use the given file. And, run the command
     
     ```
     grub-mkconfig -o /boot/grub/grub.cfg
     ```
     
7. Update the grub.cfg file to enable dual boot mode. You can refer the given grub.cfg file in the repo.

8. Update the /etc/fstab file.
   ```
   genfstab -U /mnt >> /mnt/etc/fstab
   ```
   or you can do it manually,
   ```
   blkid
   vi /mnt/install/etc/fstab
   ```
9. You can continue using the system with the user `neo` (it's cool). But if you want to create a new user, run the essential commands for it and verify you have done it correctly.

10. Set your desired root password.
    ```
    passwd
    ```

11. Ensure you have set everything right.

12. Exit and reboot.
    ```
    exit
    umount -R /mnt
    reboot
    ```

You can now start using Cypher. <br>

### Installing Cypher Components
Once you've set up the system and have hyprland running, you can install Cypher's own graphical components from [Cypher-shell's Releases Page](https://github.com/Cypher-OS/Cypher-Shell/releases/tag/v1.0.0) and follow the installation guide given there. These applications weren't included in the final .iso image to ensure minimalism. 

   

   
   
   
