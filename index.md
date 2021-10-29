<h1 align="center">Arch Linux Installation Documentation</h1>

### Linux Pre-Installation

1. **Download an HTTP link from https://archlinux.org/download/** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: if the iso has split into two files, it has not downloaded correctly. The MD5/SHA1 will not match.*</br>

2. **To verify the signature:**
    * Type “shasum -a 1 filepath” into terminal to output the SHA1 Checksum \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: You can drag the file into the terminal window to automatically input the path.* 
    * Copy the outputted SHA1 Checksum into the Text Editor Mac application
    * Copy the official/given Checksum into the Text Editor
    * Press *command + F*
    * Paste the official/given Checksum into the search bar and click enter
    * Both SHA1 Checksums should match \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: The same process can be performed using the “MD5 filepath” terminal command.* 

3. **Install/Boot into Arch Linux via VM**
    * Enter the VM library with the *^ + command + L* keys
    * Select “new” and drag the iso file into the VM window
    * Choose “other Linux 5.x kernel 64-bit” for the operating system \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: the kernel release number can be found under the “release info” section on the downloads page.*
    * Choose UEFI – The installation image uses systemd-boot for UEFI
    * Click “customize settings” to change RAM allocation – I used 2048MB
    * Press the start button
    * Select Arch Linux Installation Medium and hit enter \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: After following these steps, the screen should contain an empty command line. The default keyboard layout is US.*  

4. **Verify the boot method**
    * ls /sys/firmware/efi/efivars <- If this command does not work/throws error, the system may have booted into BIOS mode. 

5. **Update the system clock**
    * Enter the “timedatectl set-ntp true” command
    * Check that the time was successfully updated with the “timedatectl status” command. Make sure that NTP = active 

6. **Partition the disks** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: cfdisk command can be used to create the partition tables <- This is a graphical version of fdisk that is MUCH more &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;user-friendly.*
    * Type “fdisk -l” to identify the block disks. *Rom, loop, or airoot can be ignored*.
    * Type “fdisk /dev/sda” to enter the edit mode
    * Type n to create a new partition (redo for each)
      * Click enter for default partition number
      * Click enter (leave blank) for default first sector
      * Type “+ size” for the second sector (ex. “+260M”)
        * EFI system partition -> at least 260 MiB
        * Linux swap -> more than 512 MiB
        * Linux root (x86-64) -> remainder 
    * Type t to change partition type
      * Enter the partition number
      * Enter alias (1 = EFI System, 19 = Linux Swap, 23 = Linux root (x86-64)
    * Type w to write changes
    * Type “fdisk -l /dev/sda” to double-check partitions were successfully created
    
7. **Format the partitions**
    * EFI System <- “mkfs.vfat /dev/efi_system_partition” (/dev/sda1)
    * Linux Swap <- “mkswap /dev/swap_partition” (/dev/sda2)
    * Linux root (x86-64) <- “mkfs.ext4 /dev/root_partition” (/dev/sda3)

8. **Mount the filesystems**
    * Type “mount /dev/root_partition /mnt” to mount the root partition
    * Type “mkdir /boot/EFI”
    * Type “mount /dev/efi_system_partition /boot/EFI” to mount the EFI partition \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: DO NOT mount to /mnt/EFI because the later instructions will not work correctly. If this causes problems – CAN  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOUNT LATER.*  
    * Type “swapon /dev/swap_partition” to enable swap

### Linux Installation - Essential Packages/Text Editor

1. **Select the mirrors (w/ reflector)**
    * Type “pacman -Sy reflector” to install the reflector package
    * Type “reflector –verbose -l 200 -p http –sort rate –save /etc/pacman.d/mirrorlist

2. **Install essential packages**
    * Type “pacstrap /mnt base linux linux-firmware”
    * Type "pacman -S nano" to install the nano text editor

### Configure 

1. **Fstab**
    * Type “genfstab -U  /mnt >> /mnt/etc/fstab” \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: Check the resulting /mnt/etc/fstab file, and edit it in case of errors.*

2. **Change root into the new system**
    * Type “arch-chroot /mnt”

3. **Adjust the time zone**
    * Type “ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime” to change the time zone
    * Check that this was successful with the Date command

4. **Localization**
    * Type “pacman -S nano"
    * Type “nano /etc/locale.gen” and uncomment en_US.UTF-8 UTF-8
    * Type “locale-gen”
    * Type “echo LANG=en_US.UTF-8 > /etc/locale.conf” to create/set the locale.conf file. 

5. **Network configuration**
    * Type “echo hostname > /etc/hostname” 

6. **Root password**
    * Type “passwd”… and then enter desired password 

7. **Install bootloader**
    * Type “pacman -S grub”
    * Type “pacman -S efibootmgr dosfstools os-prober mtools”
    * Type “mkdir /boot/EFI”
    * Type “mount /dev/sda1 /boot/EFI”
    * Type “grub-install --target=x86_64-efi --bootloader-id=grub_uefi –recheck” 
    * Type “grub-mkconfig -o /boot/grub/grub.cfg”

8. **Install networking manager**
    * Type “pacman -S dhcpcd.”
    * Type “systemctl enable dhcpcd.service" \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: if this service is not working, type “dhcpcd” to start.*

9. **Reboot**
    * Type exit command to leave chroot environment
    * Type *shutdown now* to shutdown
    * Go to VM library settings and remove boot disc – and restart linux VM

### Post-Installation Guide

1. **Install sudo/edit file**
    * Type “pacman -S sudo” to install sudo capabilities
    * Enable sudo powers for members of the wheel group:
      * Type “EDITOR=nano visudo” to edit the sudoers file with nano
      * Uncomment the “%wheel ALL=(ALL) ALL” line

2. **Add users/give sudo powers**
    * Type “useradd -m name” to create a new user
    * Type “passwd name” to add a password for each created user \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: Can type “passwd -e name” to create expiring password* \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: Can check password expiration date by typing “sudo chage -l name”*
    * Type “usermod -aG wheel name” to add users to the sudo-group
    
3. **Create/install DE (gnome) <- Did not like Budgie or LXDE**
    * Type “pacman -S xorg gnome gnome-extra gdm”  <- use defaults
    * Type “systemctl enable gdm” to enable the display manager
    * Type *reboot*

4. **Install another shell (zsh)**
    * Type “pacman -S fish” to install the zsh package
    * Type zsh to run the new user install setup \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: can be manually started with “autoload -Uz zsh-newuser-install zsh-newuser-install -f”*
    * Type “chsh -s shell path” to change the default user shell
    
5. **Install OpenSSH**
    * Type “pacman -Sy openssh”
    * Log-in to the student gateway 
      * ssh -p53997 username@129.244.245.21
      * ssh -p53997 username@192.168.2.XX

6. **Change color of the terminal output (bash)**
    * Type “sudo cp /etc/bash.bashrc /etc/bash.bashrc.backup” to back up the bashrc file (to avoid issues)
    * Type “sudo nano /etc/bash.bashrc” to edit the bashrc file \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: Editing this file edits the color schedule for all users!*
    * Type `"\[\e[30m\][\[\e[m\]\[\e[1;34m\]\u\[\e[m\]\[\e[1;33m\]@\[\e[m\]\[\e[34m\]\h\[\e[m\]:\[\e[1;33m\]\w\[\e[m\]\[\e[30m\]]\[\e[m\]\[\e[1;33m\]\\$\[\e[m\]"` to change the bash prompt to TU colors \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: If the colors don’t change for all users, update each of the home .bashrc files.*

7. **Add aliases**
    * Use the cd command to enter home folder
    * Type “nano .bashrc” to edit the bashrc file
    * Add alias with the following syntax “alias name=’command’” Example: alias c=’clear’
    * Reboot the system to see the changes \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Note: Editing the /etc/bash.bashrc file will add aliases for all users.*







