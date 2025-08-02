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
## Específico para UEFI:
```bash
pacstrap -K /mnt efibootmgr
```
## Específico para BIOS:
```bash
pacstrap -K /mnt os-prober
```
## Generar fstab:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
# Entrar al sistema (chroot):
```bash
arch-chroot /mnt
```
# Configuración básica del sistema
## Zona horaria (Ecuador):
```bash
ln -sf /usr/share/zoneinfo/America/Guayaquil /etc/localtime
timedatectl set-ntp true
timedatectl set-local-rtc 0
hwclock --systohc
nano /etc/locale.gen  # Descomentar es_EC.UTF-8 UTF-8
locale-gen
```
## Configuración regional:
```bash
echo "LANG=es_EC.UTF-8" > /etc/locale.conf
```
## Configuración de red y hostname:
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
## Contraseña root:
```bash
passwd
```
## Crear usuario:
```bash
useradd -m -G wheel -s /bin/bash tu_usuario
passwd tu_usuario
```
## Configurar sudo:
```bash
EDITOR=nano visudo  # Descomentar: %wheel ALL=(ALL:ALL) ALL
```
# Instalar GRUB:
## Para UEFI:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
```
## Para BIOS:
```bash
grub-install --target=i386-pc /dev/sda
```
# Finalizar instalación
```bash
exit
umount -R /mnt
reboot
```
-----
# Post-Install, eso creo
0. Conectarse a internet
1. Actualizar sistema:
   sudo pacman -Syu

2. Configurar Flatpak:
   flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
   reboot

3. Instalar gnome-browser-connector:
   sudo pacman -S gnome-browser-connector

4. Configurar .bashrc para Fastfetch:
   nano ~/.bashrc
   # Añadir al final del texto:
clear
	fastfetch

5. Activar Multilib (para Steam/WINE):
   sudo nano /etc/pacman.conf
   # Descomentar:
   [multilib]
   Include = /etc/pacman.d/mirrorlist
   sudo pacman -Syu

6. Configurar firewall:
   sudo firewall-cmd --set-default-zone=home
   sudo firewall-cmd --complete-reload

7. Botones de ventana:
   gsettings set org.gnome.desktop.wm.preferences button-layout "appmenu:minimize,maximize,close"

8. Yay (AUR helper):
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
-----
# Mi uso diario

### Navegador:----------
En gnome software descargar brave

# Herramientas adicionales:----------
sudo pacman -S gnome-system-monitor eog gnome-calculator


# Java:----------
yay -S jdk21-temurin


# Python:----------
sudo pacman -S python python-pip python-virtualenv

# Instalar Fonts faltantes:----------
sudo pacman -S wqy-zenhei ttf-arphic-uming noto-fonts-cjk

# Extensiones usadas:----------
Bluetooth Battery Meter
Blur my Shell 
Dash to Dock 
Just Perfection 
User Avatar In Quick Settings
User Themes 
V-Shell 


# Paquetes de retoques:----------
Cursor: Bibata Modern Ice
Iconos: Tela 
GNOME Shell: Layan-Dark 
Aplicaciones heredadas: Layan-Dark


# Perfil de Power Options:----------
sudo pacman -S power-profiles-daemon
sudo systemctl enable power-profiles-daemon.service


# Dual boot con windows:----------
    #Ver la UUID de la particion efi
    sudo blkid /dev/nvme0n1p1
    
    #Editar opciones de arranque
    sudo nano /etc/grub.d/40_custom
    
    #Agregar esto al final de el texto:

menuentry "Windows" {
  insmod part_gpt
  insmod fat
  insmod chain
  search --fs-uuid --set=root 1234-5678 #Reemplazar 1234-5678 con la UUID real de tu partición EFI de Windows
  chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
    #Generar grub con todos los cambios
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    reboot


# Terminal de gnome transparente:----------
yay gnome-terminal-transparency


# Steam:----------
sudo pacman -S steam lib32-vulkan-intel
yay -S proton-ge-custom-bin

# Instalar Sober:----------
    En la gnome-software esta la el Sober

# Instalar SKlauncher:----------
    (Descargar el .jar de la pag oficial en el apartado de linux: https://skmedix.pl/downloads)
    (Descargar un icon.png sobre sklauncher)

    #Crear una carpeta llamada .SKlauncher en /home/user

    #Mover ahi los dos archivos (.jar y .png)

    #Crear la aplicacion de sklauncher
    nano ~/.local/share/applications/sklauncher.desktop

    #Agregar dentro de esa app lo siguiente:

[Desktop Entry]
Name=SKLauncher
Comment=Lanzador de Minecraft
Exec=java -jar /home/sijie/.sklauncher/SKlauncher-3.2.12.jar
Icon=/home/sijie/.sklauncher/sklauncher.png
Terminal=false
Type=Application
Categories=Game;
StartupNotify=true

    #Puedes cambiar los directorios de el .jar y .png


# Instalar Waydroid con root magisk:----------

       yay -S waydroid
    
    a. desargar de soruce forge el system y vendor
       sudo mkdir -p /usr/share/waydroid-extra/images/
       cd /usr/share/waydroid-extra/images/

    b. Extraer el system.img y vendor.img
       sudo mv /ruta/de/donde/esten/los/.img /usr/share/waydroid-extra/images/
       sudo waydroid init

    c. Abrir Waydroid

       sudo waydroid session stop
       sudo waydroid container stop 

       firewall-cmd --zone=trusted --add-port=67/udp
       firewall-cmd --zone=trusted --add-port=53/udp
       firewall-cmd --zone=trusted --add-forward
       firewall-cmd --zone=trusted --add-interface=waydroid0
    
    d. Ahora root

        https://github.com/mistrmochov/WaydroidSU

# Activar Bluetooth:----------
sudo pacman -S bluez bluez-utils
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service


##### RECOMENDACIONES Y MANTENIMIENTO

# Para personalización avanzada de GNOME Shell:
# 1. Los temas de iconos van en: /home/user/.icons
# 2. Los temas de shell/GTK van en: /home/user/.themes
# 3. Para editar el tema actual:
#    nano ~/.local/share/themes/Thema/gnome-shell/gnome-shell.css (cambiar el .popup-menu-content para ajustar el color, debe de estar rgba)

# Limpieza del sistema:
# - Paquetes huérfanos: sudo pacman -Rns $(pacman -Qdtq)
# - Caché: sudo pacman -Sc
# - Actualización completa: sudo pacman -Syu && yay -Syu

# Solución de problemas:
# - Si el tema no aplica: Alt+F2, luego 'r' para reiniciar GNOME Shell
# - Para restaurar configuración: rm -rf ~/.config/dconf/user

# Limpiar caché de paquetes antiguos:
# - sudo paccache -rk2

======================================================================

##### CONFIGURACIONES

#Sober FFLAGS
    #En el directorior ~/.var/app/org.vinegarhq.Sober/config/sober se encuentra los fflags

    #Añadir los fastflags en el archivo config.json

        "DFFlagDebugDisableTimeoutDisconnect": "True",
        "DFFlagDebugPauseVoxelizer": "True",
        "DFFlagDisableDPIScale": "True",
        "DFFlagOrder66": "True",
        "DFIntConnectionMTUSize": "900",
        "DFIntMaxFrameBufferSize": "4",
        "DFIntMaxLoadableAudioChannelCount": "1",
        "DFIntPerformanceControlTextureQualityBestUtility": "-1",
        "FFlagAdServiceEnabled": "False",
        "FFlagDebugDisableTelemetryEphemeralCounter": "True",
        "FFlagDebugDisableTelemetryEphemeralStat": "True",
        "FFlagDebugDisableTelemetryEventIngest": "True",
        "FFlagDebugDisableTelemetryPoint": "True",
        "FFlagDebugDisableTelemetryV2Counter": "True",
        "FFlagDebugDisableTelemetryV2Event": "True",
        "FFlagDebugDisableTelemetryV2Stat": "True",
        "FFlagDebugForceChatDisabled": "false",
        "FFlagDebugGraphicsDisableDirect3D11": "True",
        "FFlagDebugGraphicsPreferOpenGL": "True",
        "FFlagDisablePostFx": "True",
        "FFlagOptimizeNetwork": "True",
        "FFlagOptimizeNetworkRouting": "True",
        "FFlagOptimizeNetworkTransport": "True",
        "FIntFRMMaxGrassDistance": "0",
        "FIntFRMMinGrassDistance": "0",
        "FIntRakNetResendBufferArrayLength": "128",
        "FIntRenderGrassDetailStrands": "0",
        "FIntRenderGrassHeightScaler": "0",
        "FIntRenderShadowIntensity": "0",
        "FIntRomarkStartWithGraphicQualityLevel": "1",
        "FIntTerrainArraySliceSize": "4"

#Configurar distribución de teclado en configuración 

#Configurar los atajos del teclado en configuración
