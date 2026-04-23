# Arch Linux Installation Guide (UEFI)

> Panduan instalasi Arch Linux minimal berbasis UEFI.
> Cocok untuk pemula yang ingin setup manual tapi tetap rapi dan terstruktur.

---

## 1. Persiapan

### Download ISO

* Ambil dari: [https://archlinux.org/download/](https://archlinux.org/download/)

### Buat Bootable USB

* Gunakan:

  * Rufus (Windows)
  * Ventoy (multi-ISO, recommended)

### Boot ke Live ISO

* Masuk BIOS/UEFI → pilih USB
* Pilih: `Arch Linux install medium`

---

## 2. Cek Koneksi Internet

### Test koneksi:

```bash
ping archlinux.org
```

### Jika pakai WiFi:

```bash
iwctl
```

Di dalam iwctl:

```bash
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "nama_wifi"
exit
```

---

## 3. Partisi & File System

### Cek disk:

```bash
lsblk
```

Contoh target:

```
/dev/nvme0n1
```

---

## 🔹 Skema Partisi (UEFI)

| Partisi | Ukuran | Tipe         |
| ------- | ------ | ------------ |
| EFI     | 512MB  | FAT32        |
| Root    | Sisa   | ext4 / btrfs |

---

## 🔸 Buat Partisi

```bash
cfdisk /dev/nvme0n1
```

* Buat:

  * `/dev/nvme0n1p1` → EFI
  * `/dev/nvme0n1p2` → Root
* Pilih **Write** → lalu Quit

---

## Format Partisi

### ext4:

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```

### btrfs:

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -f /dev/nvme0n1p2
```

---

## Mount Partisi

### ext4:

```bash
mount /dev/nvme0n1p2 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

---

### btrfs (dengan subvolume)

```bash
mount /dev/nvme0n1p2 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@snapshots
umount /mnt
```

Mount ulang:

```bash
mount -o compress=zstd,subvol=@ /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{boot,home,.snapshots,var/cache,var/log,tmp}

mount -o noatime,compress=zstd,ssd,discard=async,subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@cache /dev/nvme0n1p2 /mnt/var/cache
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@log /dev/nvme0n1p2 /mnt/var/log

mount /dev/nvme0n1p1 /mnt/boot
```

---

## 4. Install Base System

```bash
pacstrap /mnt base linux linux-firmware
"
base = Paket dasar dari sistem. Berisi core utilities (bash, coreutils, dll) agar sistem bisa boot
linux =Kernel utama Linux. Berfungsi sebagai penghubung antara hardware dan software.
linux-firmware = Firmware untuk berbagai perangkat keras (wifi, GPU, bluetooth, dll).
nano = Text editor sederhana berbasis terminal.
neovim = Text editor modern (fork dari vim).
sudo = Memberikan akses root sementara ke user biasa.
networkmanager = Service untuk mengelola koneksi jaringan (wifi & ethernet).
git = Version control system. untuk clone repository dan manajemen kode.
bash-completion = auto complete comand
pipewire         = Audio server modern (pengganti PulseAudio & JACK).
pipewire-alsa    = Layer kompatibilitas untuk ALSA.
pipewire-pulse   = Layer kompatibilitas untuk PulseAudio.
pipewire-jack    = Layer kompatibilitas untuk JACK (audio profesional).
"
```

---

## Generate fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Cek:

```bash
cat /mnt/etc/fstab
```

---

## 5. Chroot ke System

```bash
arch-chroot /mnt
```

---

## 6. Konfigurasi Sistem

### Timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
```

---

### Locale

Edit:

```bash
nano /etc/locale.gen
```

Uncomment:

```
en_US.UTF-8 UTF-8
id_ID.UTF-8 UTF-8
```

Generate:

```bash
locale-gen
```

Set default:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

### Keyboard (optional)

```bash
echo "KEYMAP=us" > /etc/vconsole.conf
```

---

### Hostname

```bash
echo "archlinux" > /etc/hostname
```

Edit hosts:

```bash
nano /etc/hosts
```

Isi:

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
```

---

### Root Password

```bash
passwd
```

---

### User

```bash
useradd -m -G wheel -s /bin/bash username
passwd username
```

---

### Enable sudo

```bash
EDITOR=nano visudo
```

Uncomment:

```
%wheel ALL=(ALL:ALL) ALL
```

---

### Enable NetworkManager

```bash
systemctl enable NetworkManager
```

---

## 7. Install Bootloader (GRUB - UEFI)

```bash
pacman -S grub efibootmgr
```

Install:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Generate config:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 8. Finalisasi

Keluar:

```bash
exit
```

Unmount:

```bash
umount -R /mnt
```

Reboot:

```bash
reboot
```

---

# Catatan

* Pastikan boot mode = **UEFI**, bukan Legacy
* Jika pakai SSD, disarankan gunakan:

  ```
  noatime,compress=zstd
  ```
* Untuk setup lanjut:

  * Desktop Environment (Hyprland, GNOME, KDE)
  * Audio (PipeWire)
  * GPU driver

---

# Related

* Arch Wiki: [https://wiki.archlinux.org/](https://wiki.archlinux.org/)
* Installation Guide: [https://wiki.archlinux.org/title/Installation_guide](https://wiki.archlinux.org/title/Installation_guide)

---

Kalau kamu mau next step, kita bisa lanjut bikin:

* 🔥 **Post-install (Hyprland setup clean)**
* ⚡ **Dotfiles structure biar scalable**
* 🧠 **Versi advanced (btrfs + snapshot + rollback)**
