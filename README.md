# � Instalar-Arch

🔹 Sobre la guia
Esta guia de instalacion fue hecha unicamente solo para mi uso, si usted desea hacer uso de ella tiene que hacerlo bajo su responsabilidad, realmente lo adapte para mis necesidades ya que lleva en si gnome como escritorio principal y un mayor enfoque para intel
---

## � Características Principales  
- *Guia de instalación manual*   
- *Escritorio*: Gnome 
- *Enfoque principal*: Lo mas vanilla posible
- *Región*: America/Guayaquill UTC -5
- *Paquetes Instalados despues del primer boot*: 400-600
- *Tanto para GPT y MBR*


![Arch Linux](https://img.shields.io/badge/Arch_Linux-1793D1?logo=arch-linux&logoColor=white)
![UEFI/BIOS](https://img.shields.io/badge/UEFI%2FBIOS-Compatible-blueviolet)
![GNOME](https://img.shields.io/badge/GNOME-4A86CF?logo=gnome&logoColor=white)

## Conexión a Internet (Común para ambos sistemas)
```bash
rfkill unblock all
ip link set wlan0 up
iwctl
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect "TU_RED"
exit
ping -c 4 archlinux.org
```
## Particionar disco (GPT)
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

## Instalar Sistema Base
# Paquetes comunes para ambos sistemas:
```bash
pacstrap -K /mnt base linux-zen linux-zen-headers linux-firmware intel-ucode \
networkmanager grub sudo nano gdm gnome-shell gnome-control-center gnome-tweaks \
xdg-user-dirs flatpak bash-completion fastfetch firewalld
```
# Específico para UEFI:
```bash
pacstrap -K /mnt efibootmgr
```
# Específico para BIOS:
```bash
pacstrap -K /mnt os-prober
```
## Configuración básica del sistema
# Zona horaria (Ecuador)
```bash
ln -sf /usr/share/zoneinfo/America/Guayaquil /etc/localtime
```
# Configuración regional
```bash
echo "LANG=es_EC.UTF-8" > /etc/locale.conf
```
# Crear usuario
```bash
useradd -m -G wheel -s /bin/bash tu_usuario
passwd tu_usuario
```
# Configurar sudo
```bash
EDITOR=nano visudo  # Descomentar: %wheel ALL=(ALL:ALL) ALL
```
# Instalar GRUB
## Para UEFI:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
```
## Para BIOS:
```bash
grub-install --target=i386-pc /dev/sda
```

```bash
