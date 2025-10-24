# Instalación de Arch Linux

Esta guía de instalación fue hecha únicamente para mi uso. Si deseas hacer uso de ella, hazlo bajo tu propio riesgo. La adapté para mis necesidades con un mayor enfoque en Intel.

---

## Características Principales

- **Guía de instalación**: Manual
- **Escritorio**: GNOME
- **Enfoque principal**: Lo más vanilla posible
- **Región**: America/Guayaquil UTC -5
- **Paquetes instalados después del primer boot**: 600-700
- **Compatibilidad**: GPT y MBR

![Arch Linux](https://img.shields.io/badge/Arch_Linux-1793D1?logo=arch-linux&logoColor=white)
![UEFI/BIOS](https://img.shields.io/badge/UEFI%2FBIOS-Compatible-blueviolet)
![GNOME](https://img.shields.io/badge/GNOME-4A86CF?logo=gnome&logoColor=white)

---

## Conexión a Internet

```bash
rfkill unblock all
# Verifica tu red con: ip link
ip link set wlan0 up
iwctl
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect "TU_RED"
exit
ping -c 4 archlinux.org
```

---

## Particionar Disco

```bash
lsblk
cfdisk /dev/nvme0n1  # Ajusta según tu disco
```

### Esquema de particiones GPT

| Partición | Tamaño    | Tipo              | Montaje |
|-----------|-----------|-------------------|---------|
| nvme0n1p1 | 550 MiB   | EFI System (ef00) | /boot   |
| nvme0n1p2 | 8-16 GiB  | Linux swap        | [SWAP]  |
| nvme0n1p3 | 30-50 GiB | Linux filesystem  | /       |
| nvme0n1p4 | Resto     | Linux filesystem  | /home   |

### Esquema de particiones MBR

| Partición | Tamaño    | Tipo             | Montaje |
|-----------|-----------|------------------|---------|
| sda1      | 512 MiB   | Bootable, Linux  | /boot   |
| sda2      | 8-16 GiB  | Linux swap       | [SWAP]  |
| sda3      | 30-50 GiB | Linux filesystem | /       |
| sda4      | Resto     | Linux filesystem | /home   |

---

## Formatear y Montar Particiones

### Para GPT

**Formatear:**
```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
mkfs.ext4 /dev/nvme0n1p3
mkfs.ext4 /dev/nvme0n1p4
```

**Montar:**
```bash
mount /dev/nvme0n1p3 /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
mkdir -p /mnt/home
mount /dev/nvme0n1p4 /mnt/home
```

### Para MBR

**Formatear:**
```bash
mkfs.ext4 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
```

**Montar:**
```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
swapon /dev/sda2
mkdir -p /mnt/home
mount /dev/sda4 /mnt/home
```

---

## Instalar Sistema Base

```bash
pacstrap -K /mnt \
    base base-devel \
    bash-completion bluez bluez-utils \
    eog fastfetch firefox firewalld flatpak \
    gdm git \
    gnome-calculator gnome-control-center gnome-shell gnome-software \
    gnome-terminal gnome-tweaks \
    grub intel-ucode \
    lib32-vulkan-intel linux-firmware linux-zen \
    mesa nano nautilus networkmanager noto-fonts-cjk \
    os-prober power-profiles-daemon \
    python python-pip python-virtualenv \
    steam ttf-arphic-uming vulkan-intel wqy-zenhei xdg-user-dirs \
    --needed
```

### Específico para UEFI

```bash
pacstrap -K /mnt efibootmgr
```

### Generar fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

---

## Entrar al Sistema (chroot)

```bash
arch-chroot /mnt
```

---

## Configuración Básica del Sistema

### Zona horaria (Ecuador)

```bash
ln -sf /usr/share/zoneinfo/America/Guayaquil /etc/localtime
timedatectl set-ntp true
timedatectl set-local-rtc 0
hwclock --systohc
```

### Configurar idioma

```bash
nano /etc/locale.gen  # Descomentar es_EC.UTF-8 UTF-8
locale-gen
echo "LANG=es_EC.UTF-8" > /etc/locale.conf
```

### Configurar hostname

```bash
echo "arch" > /etc/hostname
nano /etc/hosts
```

Añade estas líneas:
```
127.0.0.1    localhost
::1          localhost
127.0.1.1    arch.localdomain arch
```

---

## Habilitar Servicios

```bash
systemctl enable NetworkManager
systemctl enable gdm
systemctl enable firewalld
systemctl enable bluetooth
systemctl enable power-profiles-daemon
```

---

## Configurar Usuario y Sudo

### Contraseña root

```bash
passwd
```

### Crear usuario

```bash
useradd -m -G wheel -s /bin/bash tu_usuario
passwd tu_usuario
```

### Configurar sudo

```bash
EDITOR=nano visudo  # Descomentar: %wheel ALL=(ALL:ALL) ALL
```

---

## Instalar GRUB (Dualboot con Windows)

### Para UEFI

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
nano /etc/default/grub  # Descomentar: GRUB_DISABLE_OS_PROBER=false
grub-mkconfig -o /boot/grub/grub.cfg
```

### Para BIOS

```bash
grub-install --target=i386-pc /dev/sda
nano /etc/default/grub  # Descomentar: GRUB_DISABLE_OS_PROBER=false
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Finalizar Instalación

```bash
exit
umount -R /mnt
reboot
```

---

## Post-Instalación

### 1. Conectarse a internet y actualizar

```bash
sudo pacman -Syu
```

### 2. Configurar Flatpak

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
reboot
```

### 3. Configurar .bashrc para Fastfetch

```bash
nano ~/.bashrc
```

Añade al final:
```bash
if [ -t 1 ]; then
    echo -n "¿Quieres actualizar el sistema? (s/n): "
    read respuesta
    if [[ "$respuesta" == "s" || "$respuesta" == "S" ]]; then
        sudo pacman -Syu && yay -Syu
        clear
        fastfetch
    else
        clear
        fastfetch
    fi
fi
```

### 4. Activar Multilib (para Steam/WINE)

```bash
sudo nano /etc/pacman.conf
```

Descomentar:
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

```bash
sudo pacman -Syu
```

### 5. Configurar firewall

```bash
sudo firewall-cmd --set-default-zone=home
sudo firewall-cmd --complete-reload
```

### 6. Botones de ventana

```bash
gsettings set org.gnome.desktop.wm.preferences button-layout "appmenu:minimize,maximize,close"
```

### 7. Instalar Yay (AUR helper)

```bash
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
```

### 8. Instalar OpenJDK (Temurin)

```bash
yay -S jdk21-temurin
```
---

### 9. Instalar la extension de gnome-shell-integration en el navegador

```bash
https://addons.mozilla.org/en-US/firefox/addon/gnome-shell-integration/
```

## Configuración para Uso Diario

### Extensiones de GNOME

- Bluetooth Battery Meter
- Dash to Dock
- User Avatar In Quick Settings
- User Themes
- Quick Setting Audio Panel

### Temas

- **Cursor**: Bibata Modern Ice
- **Iconos**: Tela
- **GNOME Shell**: Graphite-Dark
- **Aplicaciones heredadas**: Graphite-Dark

### Terminal transparente

```bash
yay -S gnome-terminal-transparency
```

### Steam Proton-GE

```bash
yay -S proton-ge-custom-bin
```

---

## Instalar Waydroid con Root Magisk

### 1. Instalar Waydroid

```bash
yay -S waydroid
```

### 2. Preparar imágenes

Descarga de SourceForge los archivos system.img y vendor.img.

```bash
sudo mkdir -p /usr/share/waydroid-extra/images/
cd /usr/share/waydroid-extra/images/
sudo mv /ruta/de/donde/esten/los/.img /usr/share/waydroid-extra/images/
sudo waydroid init
```

### 3. Configurar firewall

```bash
sudo waydroid session stop
sudo waydroid container stop
firewall-cmd --zone=trusted --add-port=67/udp
firewall-cmd --zone=trusted --add-port=53/udp
firewall-cmd --zone=trusted --add-forward
firewall-cmd --zone=trusted --add-interface=waydroid0
```

### 4. Instalar root

Sigue las instrucciones en: https://github.com/mistrmochov/WaydroidSU

---


## Recomendaciones y Mantenimiento

### Personalización de GNOME Shell

- Los temas de iconos van en: `~/.icons`
- Los temas de shell/GTK van en: `~/.themes`
- Para editar el tema actual:
  ```bash
  nano ~/.local/share/themes/Thema/gnome-shell/gnome-shell.css
  ```
  Cambiar `.popup-menu-content` para ajustar el color (debe estar en rgba).

### Limpieza del sistema

```bash
# Paquetes huérfanos
sudo pacman -Rns $(pacman -Qdtq)

# Caché
sudo pacman -Sc
yay -Scc

# Actualización completa
sudo pacman -Syu && yay -Syu

# Limpiar caché de paquetes antiguos
sudo paccache -rk2
```

### Solución de problemas

- Si el tema no aplica: Alt+F2, luego 'r' para reiniciar GNOME Shell
- Para restaurar configuración: `rm -rf ~/.config/dconf/user`

---

## Configuraciones Específicas

### FastFlags de Sober

En el directorio `~/.var/app/org.vinegarhq.Sober/config/sober` se encuentran los FastFlags.

Añade estos FastFlags en el archivo `config.json`:

```json
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
```

### Configuración del teclado

Configura la distribución de teclado en Configuración del sistema.

### Atajos de teclado

Configura los atajos de teclado en Configuración del sistema.
