# � Instalar-Arch

🔹
Esta guia de instalacion fue hecha unicamente solo para mi uso, si usted desea hacer uso de ella tiene que hacerlo bajo su propio riesgo, realmente lo adapte para mis necesidades ya que lleva un mayor enfoque para intel
---

# � Características Principales  
- *Guia de instalación manual*   
- *Escritorio*: Gnome 
- *Enfoque principal*: Lo mas vanilla posible
- *Región*: America/Guayaquill UTC -5
- *Paquetes Instalados despues del primer boot*: 400-600
- *Tanto para GPT y MBR*


![Arch Linux](https://img.shields.io/badge/Arch_Linux-1793D1?logo=arch-linux&logoColor=white)
![UEFI/BIOS](https://img.shields.io/badge/UEFI%2FBIOS-Compatible-blueviolet)
![GNOME](https://img.shields.io/badge/GNOME-4A86CF?logo=gnome&logoColor=white)

# Conexión a Internet
```bash
rfkill unblock all
ip link set wlan0 up
iwctl
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect "TU_RED"
exit
ping -c 4 archlinux.org
```
# Particionar disco (GPT)
```bash
lsblk
cfdisk /dev/nvme0n1  # (ajustar según tu disco)
```
| Partición | Tamaño    | Tipo (GPT)       | Montaje |
|-----------|-----------|------------------|---------|
| nvme0n1p1 | 550 MiB   | EFI System (ef00)| /boot   |
| nvme0n1p2 | 8-16 GiB  | Linux swap       | [SWAP]  |
| nvme0n1p3 | 30-50 GiB | Linux filesystem | /       |
| nvme0n1p4 | Resto     | Linux filesystem | /home   |

| Partición | Tamaño    | Tipo (MBR)          | Montaje |
|-----------|-----------|---------------------|---------|
| sda1      | 512 MiB   | Bootable, Linux     | /boot   |
| sda2      | 8-16 GiB  | Linux swap          | [SWAP]  |
| sda3      | 30-50 GiB | Linux filesystem    | /       |
| sda4      | Resto     | Linux filesystem    | /home   |

## Formatear particiones (gpt)
```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
mkfs.ext4 /dev/nvme0n1p3
mkfs.ext4 /dev/nvme0n1p4
```
## Montar particiones (gpt)
```bash
mount /dev/nvme0n1p3 /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
mkdir -p /mnt/home
mount /dev/nvme0n1p4 /mnt/home
```
## Formatear particiones (mbr)
```bash
mkfs.ext4 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
```
## Montar particiones (mbr)
```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
swapon /dev/sda2
mkdir -p /mnt/home
mount /dev/sda4 /mnt/home
```
# Instalar Sistema Base
```bash
pacstrap -K /mnt base linux-zen linux-zen-headers linux-firmware intel-ucode \
networkmanager grub sudo nano gdm gnome-shell gnome-control-center gnome-tweaks \
xdg-user-dirs flatpak bash-completion fastfetch firewalld
```
Específico para UEFI:
```bash
pacstrap -K /mnt efibootmgr
```
Específico para BIOS:
```bash
pacstrap -K /mnt os-prober
```
Generar fstab:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
# Entrar al sistema (chroot)
```bash
arch-chroot /mnt
```
# Configuración básica del sistema
Zona horaria (Ecuador)
```bash
ln -sf /usr/share/zoneinfo/America/Guayaquil /etc/localtime
timedatectl set-ntp true
timedatectl set-local-rtc 0
hwclock --systohc
nano /etc/locale.gen  # Descomentar es_EC.UTF-8 UTF-8
locale-gen
```
Configuración regional
```bash
echo "LANG=es_EC.UTF-8" > /etc/locale.conf
```
Configuración de red y hostname
```bash
echo "archlinux" > /etc/hostname
nano /etc/hosts

127.0.0.1    localhost
::1          localhost
127.0.1.1    archlinux.localdomain archlinux
```
# Habilitación de servicios
```bash
systemctl enable gdm
systemctl enable NetworkManager
systemctl enable firewalld
```
# Configurar usuario y sudo
Contraseña root
```bash
passwd
```
Crear usuario
```bash
useradd -m -G wheel -s /bin/bash tu_usuario
passwd tu_usuario
```
Configurar sudo
```bash
EDITOR=nano visudo  # Descomentar: %wheel ALL=(ALL:ALL) ALL
```
## Instalar GRUB
Para UEFI:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
```
Para BIOS:
```bash
grub-install --target=i386-pc /dev/sda
```
#< Finalizar instalación
```bash
exit
umount -R /mnt
reboot
```

# Post-Install, eso creo
0. Conectarse a internet
1. Actualizar sistema
```bash
sudo pacman -Syu
```
3. Configurar Flatpak
```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```
4. Habilitar Multilib (para Steam/WINE)
```bash
sudo nano /etc/pacman.conf  # Descomentar [multilib]
sudo pacman -Syu
```
5. Instalar yay (AUR Helper)
```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
```
6. Fastfetch en terminal
```bash
echo -e "\nclear\nfastfetch" >> ~/.bashrc
```
7. Configurar firewall
```bash
sudo firewall-cmd --set-default-zone=home
sudo firewall-cmd --complete-reload
```

