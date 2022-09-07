# Archlinux Installation VirtualBox 08/2022

### Fix PGP Schluessel

    [$EDITOR] /etc/pacman.conf
SigLevel = Requeired DatabaseOptional to 

SigLevel = Never 
    sed -i 's/^SigLevel.*/SigLevel = Never/' /etc/pacman.conf
### Tastatur Layout aud Deutsch
    loadkeys de-latin1
    setfont ter-132n

### SSH Verbindung 
Üeberpruefen ob SSH aktiv ist 
    'systemctl status sshd'
Password fuer User:
    passwd

### Mirror Aktualisieren
    reflector -c Germany -a 6 --sort rate --save /etc/pacman.d/mirrorlist

Refresh Server
    pacman -Syy

### Partitionieren GPT oder MBR

    lsblk
    fdisk /dev/sda
EFI:
* 300M=/dev/sda1
* 1G=/dev/sda2
* 50G=/dev/sda3
* Rest=/dev/sda4

### Formatieren der Partitionen
    mkfs.fat -F32 /dev/sda
    mkswap /dev.sda2
    swapon /dev/sda2
    mkfs.ext4 /dev/sda3
    mkfs.ext4 /dev/sda4

    lsblk

### Partitionen Montieren
    mount /dev/sda3 /mnt
    mkdir -p /mnt/{boot/efi,home}
    mount /dev/sda1 /mnt/boot/efi
    mount /dev/sda4 /mnt/home
    lsblk

### Base Pakette
    pacstrap /mnt base linux linux-firmware git vim intel-ucode grub efibootmgr virtualbox-guest-utils lxde
### File Systemtable generieren
    genfstab -U /mnt >> /mnt/etc/fstab
### Root User in neuer Installation wechseln
    arch-chroot /mnt
### Time Zone
    ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
### Zeit Synchronisieren
    hwclock --systohc
### Locales
    echo 'de_DE.UTF-8 UTF-8' >>/mnt/etc/locale.gen
    echo 'en_US.UTF-8 UTF-8' >>/mnt/etc/locale.gen
    echo 'LANG=de_DE.UTF-8' >>/mnt/etc/locale.conf
    echo "arch" >> /etc/hostname
    echo "127.0.0.1 localhost" >> /etc/hosts
    echo "::1       localhost" >> /etc/hosts
    echo "127.0.1.1 arch.localdomain arch" >> /etc/hosts
### Root Password Aendern
    echo root:password | chpasswd

### Extras Paketten
    pacman -S sudo grub efibootmgr networkmanager network-manager-applet dialog wpa_supplicant mtools dosfstools base-devel linux-headers avahi xdg-user-dirs xdg-utils gvfs gvfs-smb nfs-utils inetutils dnsutils bluez bluez-utils cups hplip alsa-utils pipewire pipewire-alsa pipewire-pulse pipewire-jack bash-completion openssh rsync reflector acpi acpi_call tlp virt-manager qemu qemu-arch-extra edk2-ovmf bridge-utils dnsmasq vde2 openbsd-netcat iptables-nft ipset firewalld flatpak sof-firmware nss-mdns acpid os-prober ntfs-3g terminus-font

### Grub Installation

    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
    grub-mkconfig -o /boot/grub/grub.cfg
### Dienste Starten

    systemctl enable NetworkManager
    systemctl enable bluetooth
    systemctl enable cups.service
    systemctl enable sshd
    systemctl enable avahi-daemon
    systemctl enable tlp
    systemctl enable reflector.timer
    systemctl enable fstrim.timer
    systemctl enable libvirtd
    systemctl enable firewalld
    systemctl enable acpid
    systemctl enable lxdm
    systemctl enable vboxserservice'

### Neu User erstellen

    useradd -mG wheel "$USER"
    visudo
    %wheel ALL=(ALL)ALL

### Neustarten

    exit
    umount -a
    reboot
ISO entfernen
