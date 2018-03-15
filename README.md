# ArchLinux
Arch Install Directions
Misc Tweaks
Disable turbo boost:

# echo 1 > /sys/devices/system/cpu/intel_pstate/no_turbo
Disable systemd from handling lid close events:

Edit /etc/systemd/logind.conf and set HandleLidSwitch to ignore.
Installation procedures
Installation procedure (basic install):

    Use wifi-menu to connect to network

    Start ssh # systemctl start sshd

    Connect to machine via SSH

    Visit https://www.archlinux.org/mirrorlist/ on another computer, generate mirrorlist

    Edit /etc/pacman.d/mirrorlist on the Arch computer and paste the faster servers

    Update package indexes: # pacman -Syyy

    Create root partition:

    # fdisk /dev/sda

     * o (to create an empty DOS partition table)
     * n
     * 1
     * enter
     * +30G
     * w

    Create home partition:

    # fdisk /dev/sda

    * n
    * 2
    * enter
    * ebter
    * w

    mkfs.ext4 /dev/sda1

    # mkfs.ext4 /dev/sda2

    # mount /dev/sda1 /mnt

    # mkdir /mnt/home

    # mount /dev/sda2 /mnt/home

    # pacstrap -i /mnt base

    # genfstab -U -p /mnt >> /mnt/etc/fstab

    # arch-chroot /mnt

    # pacman -S openssh grub-bios linux-headers linux-lts linux-lts-headers

    If wireless: # pacman -S dialog network-manager-applet networkmanager networkmanager-openvpn wireless_tools wpa_supplicant wpa_actiond

    # mkinitcpio -p linux

    # mkinitcpio -p linux-lts

    # nano /etc/locale.gen (uncomment en_US.UTF-8)

    # locale-gen

    # passwd (for setting root password)

    # grub-install --target=i386-pc --recheck /dev/sda

    # cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo

    # grub-mkconfig -o /boot/grub/grub.cfg

    Create swap file:
        # fallocate -l 2G /swapfile
        # chmod 600 /swapfile
        # mkswap /swapfile
        # echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

    $ exit

    # umount -a

    # reboot

General Installation procedure (standard install on EFI):

    Use wifi-menu to connect to network

    Start ssh # systemctl start sshd

    Connect to machine via SSH

    Visit https://www.archlinux.org/mirrorlist/ on another computer, generate mirrorlist

    Edit /etc/pacman.d/mirrorlist on the Arch computer and paste the faster servers

    Update package indexes: # pacman -Syyy

    Create efi partition:

    # fdisk /dev/sda

     * g (to create an empty GPT partition table)
     * n
     * 1
     * enter
     * +300M
     * t
     * 1 (For EFI)
     * w

    Create root partition:

    # fdisk /dev/sda

    * n
    * 2
    * enter
    * +30G
    * w

    Create home partition:

    # fdisk /dev/sda

     * n
     * 3
     * enter
     * enter
     * w

    # mkfs.fat -F32 /dev/sda1

    # mkfs.ext4 /dev/sda2 # mkfs.ext4 /dev/sda3 # mount /dev/sda2 /mnt

    # mkdir /mnt/home

    # mount /dev/sda3 /mnt/home

    # pacstrap -i /mnt base

    # genfstab -U -p /mnt >> /mnt/etc/fstab

    # arch-chroot /mnt

    # pacman -S grub efibootmgr dosfstools openssh os-prober mtools linux-headers linux-lts linux-lts-headers

    # nano /etc/locale.gen (uncomment en_US.UTF-8)

    # locale-gen

    Enable root logon via ssh

    # systemctl enable sshd.service

    # passwd (for setting root password)

    # mkdir /boot/EFI

    # mount /dev/sda1 /boot/EFI

    # grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck

    # cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo

    # grub-mkconfig -o /boot/grub/grub.cfg

    Create swap file:
        # fallocate -l 2G /swapfile
        # chmod 600 /swapfile
        # mkswap /swapfile
        # echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab

    $ exit

    # umount -a

    # reboot

Installation procedure (encrypted lvm on EFI):

    Use wifi-menu to connect to network

    Start ssh # systemctl start sshd

    Connect to machine via SSH

    Visit https://www.archlinux.org/mirrorlist/ on another computer, generate mirrorlist

    Edit /etc/pacman.d/mirrorlist on the Arch computer and paste the faster servers

    Update package indexes: # pacman -Syyy

    Create efi partition:

    # fdisk /dev/sda

     * g (to create an empty GPT partition table)
     * n
     * 1
     * enter
     * +300M
     * t
     * 1 (For EFI)
     * w

    Create boot partition:

    # fdisk /dev/sda

    * n
    * 2
    * enter
    * +400M
    * w

    Create LVM partition:

    # fdisk /dev/sda

     * n
     * 3
     * enter
     * enter
     * t
     * 3
     * 31
     * w

    # mkfs.fat -F32 /dev/sda1

    # mkfs.ext2 /dev/sda2

    Set up encryption
        # cryptsetup luksFormat /dev/sda3
        # cryptsetup open --type luks /dev/sda3 lvm

    Set up lvm:
        # pvcreate --dataalignment 1m /dev/mapper/lvm
        # vgcreate volgroup0 /dev/mapper/lvm
        # lvcreate -L 30GB volgroup0 -n lv_root
        # lvcreate -L 250GB volgroup0 -n lv_home
        # modprobe dm_mod
        # vgscan
        # vgchange -ay

    # mkfs.ext4 /dev/volgroup0/lv_root

    # mkfs.xfs /dev/volgroup0/lv_home

    # mount /dev/volgroup0/lv_root /mnt

    # mkdir /mnt/boot

    # mkdir /mnt/home

    # mount /dev/sda2 /mnt/boot

    # mount /dev/volgroup0/lv_home /mnt/home

    # pacstrap -i /mnt base

    # genfstab -U -p /mnt >> /mnt/etc/fstab

    # arch-chroot /mnt

    # pacman -S grub efibootmgr dosfstools openssh os-prober mtools linux-headers linux-lts linux-lts-headers

    Edit /etc/mkinitcpio.conf and add encrypt lvm2 in between block and filesystems

    # mkinitcpio -p linux

    # mkinitcpio -p linux-lts

    # nano /etc/locale.gen (uncomment en_US.UTF-8)

    # locale-gen

    Enable root logon via ssh

    # systemctl enable sshd.service

    # passwd (for setting root password)

    Edit /etc/default/grub: add cryptdevice=<PARTUUID>:volgroup0 to the GRUB_CMDLINE_LINUX_DEFAULT line If using standard device naming, the option will look like this: cryptdevice=/dev/sda3:volgroup0

    # mkdir /boot/EFI

    # mount /dev/sda1 /boot/EFI

    # grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck

    # cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo

    # grub-mkconfig -o /boot/grub/grub.cfg

    Create swap file:
        # fallocate -l 2G /swapfile
        # chmod 600 /swapfile
        # mkswap /swapfile
        # echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab

    $ exit

    # umount -a

    # reboot

Installation procedure (lvm with no encryption):

    Use wifi-menu to connect to network

    Start ssh # systemctl start sshd

    Connect to machine via SSH

    Visit https://www.archlinux.org/mirrorlist/ on another computer, generate mirrorlist

    Edit /etc/pacman.d/mirrorlist on the Arch computer and paste the faster servers

    Update package indexes: # pacman -Syyy

    Create efi partition:

    # fdisk /dev/sda

     * g (to create an empty GPT partition table)
     * n
     * 1
     * enter
     * +300M
     * t
     * 1 (For EFI)
     * w

    Create LVM partition

    # fdisk /dev/sda

     * n
     * 2
     * enter
     * enter
     * t
     * 2
     * 31
     * w

    # mkfs.fat -F32 /dev/sda1

    Set up lvm:
        Non-SSD: # pvcreate /dev/sda2
        SSD: # pvcreate --dataalignment 1m /dev/sda2
        # vgcreate volgroup0 /dev/sda2
        # lvcreate -L 30GB volgroup0 -n lv_root
        # lvcreate -L 250GB volgroup0 -n lv_home
        # modprobe dm_mod
        # vgscan
        # vgchange -ay

    # mkfs.ext4 /dev/volgroup0/lv_root

    # mkfs.xfs /dev/volgroup0/lv_home

    # mount /dev/volgroup0/lv_root /mnt

    # mkdir /mnt/home

    # mount /dev/volgroup0/lv_home /mnt/home

    # pacstrap -i /mnt base

    # genfstab -U -p /mnt >> /mnt/etc/fstab

    # arch-chroot /mnt

    # pacman -S grub efibootmgr dosfstools openssh os-prober mtools linux-headers linux-lts linux-lts-headers

    Edit /etc/mkinitcpio.conf and add lvm2 in between block and filesystems

    # mkinitcpio -p linux

    # mkinitcpio -p linux-lts

    # nano /etc/locale.gen (uncomment en_US.UTF-8)

    # locale-gen

    Enable root logon via ssh

    # systemctl enable sshd.service

    # passwd (for setting root password)

    # mkdir /boot/EFI

    # mount /dev/sda1 /boot/EFI

    # grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck

    # cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo

    # grub-mkconfig -o /boot/grub/grub.cfg

    Create swap file:
        # fallocate -l 2G /swapfile
        # chmod 600 /swapfile
        # mkswap /swapfile
        # echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab

    $ exit

    `# umount -a

    # reboot

Installation procedure (lvm, mbr):

    Use wifi-menu to connect to network

    Start ssh # systemctl start sshd

    Connect to machine via SSH

    Visit https://www.archlinux.org/mirrorlist/ on another computer, generate mirrorlist

    Edit /etc/pacman.d/mirrorlist on the Arch computer and paste the faster servers

    Update package indexes: # pacman -Syyy

    Partition disk, create lvm partition: # fdisk /dev/sda
        o
        enter
        w
        n
        p
        1
        enter
        enter
        t
        1
        8E
        w

    Create lvm:
        Non-SSD: # pvcreate /dev/sda1
        SSD: # pvcreate --dataalignment 1m /dev/sda1
        # vgcreate volgroup0 /dev/sda1
        # lvcreate -L 30GB volgroup0 -n lv_root
        # lvcreate -L 250GB volgroup0 -n lv_home
        # modprobe dm_mod
        # vgscan
        # vgchange -ay

    # mkfs.ext4 /dev/volgroup0/lv_root

    # mkfs.ext4 /dev/volgroup0/lv_home

    # mount /dev/volgroup0/lv_root /mnt

    # mkdir /mnt/home

    # mount /dev/volgroup0/lv_home /mnt/home

    # pacstrap -i /mnt base

    # genfstab -U -p /mnt >> /mnt/etc/fstab

    # arch-chroot /mnt

    # pacman -S openssh grub-bios linux-headers linux-lts linux-lts-headers

    Edit /etc/mkinitcpio.conf and add lvm2 in between block and filesystems

    # mkinitcpio -p linux

    # mkinitcpio -p linux-lts

    # nano /etc/locale.gen (uncomment en_US.UTF-8)

    # locale-gen

    # ln -s /usr/share/zoneinfo/America/Detroit /etc/localtime

    # hwclock --systohc --utc

    Enable root logon via ssh

    # systemctl enable sshd.service

    # passwd (for root)

    # grub-install --target=i386-pc --recheck /dev/sda

    # cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo

    # grub-mkconfig -o /boot/grub/grub.cfg

    Create swap file:
        # fallocate -l 2G /swapfile
        # chmod 600 /swapfile
        # mkswap /swapfile
        # echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab

    $ exit

    # umount /mnt/home

    # umount /mnt

    # reboot

Post installation steps:

    Fix GNOME app issues: # localectl set-locale LANG="en_US.UTF-8"
    Add to fstab: tmpfs /tmp tmpfs nodev,nosuid,size=2G 0 0
    If ssd, add discard to fstab. Example: UUID=<UUID> / ext4 defaults,noatime,discard 0 2

    © 2018 GitHub, Inc.
    Terms
    Privacy
    Security
    Status
    Help

