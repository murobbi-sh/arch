# Arch Linux Installation Guide (UEFI)

> Panduan instalasi Arch Linux minimal berbasis UEFI.
> Cocok untuk pemula yang ingin setup manual tapi tetap rapi dan terstruktur.

---

## 1. Persiapan

### Download ISO

Ambil ISO dari website resmi Arch Linux:

<https://archlinux.org/download/>

### Buat Bootable USB

Gunakan salah satu:

- Rufus: <https://rufus.ie/id/>
- Ventoy: <https://www.ventoy.net>

### Boot ke Live ISO

Masuk BIOS/UEFI, lalu pilih USB installer.

Pilih:

```bash
Arch Linux install medium
```

---

## 2. Cek Mode Boot

Pastikan sistem boot dalam mode UEFI:

```bash
cat /sys/firmware/efi/fw_platform_size
```

Jika hasilnya:

```bash
64
```

berarti sistem berjalan dalam mode UEFI 64-bit.

Kalau folder `/sys/firmware/efi` tidak ada, berarti kamu boot dalam mode Legacy/BIOS. Jangan lanjut dulu. Boot ulang lewat mode UEFI.

---

## 3. Cek Koneksi Internet

Test koneksi:

```bash
ping archlinux.org
```

Jika pakai WiFi:

```bash
iwctl
```

Di dalam `iwctl`:

```bash
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "nama_wifi"
exit
```

Cek lagi:

```bash
ping archlinux.org
```

---

## 4. Sinkronisasi Waktu

```bash
timedatectl
```

Kalau internet sudah aktif, biasanya waktu akan sinkron otomatis.

---

## 5. Partisi & File System

Cek disk:

```bash
lsblk
```

Contoh target disk:

```bash
/dev/nvme0n1
```

> Ganti `/dev/nvme0n1` sesuai disk punya kamu. Jangan asal copy, nanti partisi Windows jadi korban.

---

## Skema Partisi UEFI

| Partisi | Ukuran | Tipe | Mount Point |
|---|---:|---|---|
| EFI | 1GiB | FAT32 | `/boot` |
| Root | Sisa disk | ext4 / btrfs | `/` |

---

## Buat Partisi

```bash
cfdisk /dev/nvme0n1
```

Buat partisi:

- `/dev/nvme0n1p1` → EFI System
- `/dev/nvme0n1p2` → Linux filesystem / Root

Setelah selesai:

1. Pilih `Write`
2. Ketik `yes`
3. Pilih `Quit`

---

## 6. Format Partisi

### Opsi ext4

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```

### Opsi btrfs

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -f /dev/nvme0n1p2
```

---

## 7. Mount Partisi

### Jika pakai ext4

```bash
mount /dev/nvme0n1p2 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

---

### Jika pakai btrfs dengan subvolume

Mount root dulu:

```bash
mount /dev/nvme0n1p2 /mnt
```

Buat subvolume:

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@snapshots
```

Unmount:

```bash
umount /mnt
```

Mount ulang:

```bash
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@ /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{boot,home,.snapshots,var/cache,var/log,tmp}

mount -o noatime,compress=zstd,ssd,discard=async,subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@cache /dev/nvme0n1p2 /mnt/var/cache
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@log /dev/nvme0n1p2 /mnt/var/log
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@tmp /dev/nvme0n1p2 /mnt/tmp

mount /dev/nvme0n1p1 /mnt/boot
```

---

## 8. Install Base System

```bash
pacstrap -K /mnt base linux linux-firmware nano sudo networkmanager git bash-completion
```

Penjelasan paket:

| Paket | Fungsi |
|---|---|
| `base` | Paket dasar sistem Arch Linux |
| `linux` | Kernel Linux utama |
| `linux-firmware` | Firmware untuk WiFi, GPU, bluetooth, dan hardware lain |
| `nano` | Text editor terminal yang gampang |
| `sudo` | Memberi akses admin ke user biasa |
| `networkmanager` | Mengelola internet, WiFi, dan ethernet |
| `git` | Version control untuk clone dan kelola repo |
| `bash-completion` | Auto-complete command di bash |

---

## 9. Generate fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Cek hasilnya:

```bash
cat /mnt/etc/fstab
```

Kalau mount point terlihat aneh, benerin dulu sebelum lanjut.

---

## 10. Chroot ke Sistem Baru

```bash
arch-chroot /mnt
```

---

## 11. Konfigurasi Sistem

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

Uncomment baris ini:

```txt
en_US.UTF-8 UTF-8
id_ID.UTF-8 UTF-8
```

Generate locale:

```bash
locale-gen
```

Set default locale:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

### Keyboard

Opsional, kalau pakai layout US:

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

```txt
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

### Buat User

Ganti `username` dengan nama user kamu:

```bash
useradd -m -G wheel -s /bin/bash username
passwd username
```

---

### Enable sudo

```bash
EDITOR=nano visudo
```

Uncomment baris ini:

```txt
%wheel ALL=(ALL:ALL) ALL
```

---

### Enable NetworkManager

```bash
systemctl enable NetworkManager
```

---

## 12. Pilih Boot Method

Sekarang pilih salah satu:

### Opsi 1 — UKI

Pakai UKI kalau kamu mau setup yang lebih modern, rapi, dan cocok buat Secure Boot.

Baca:

- [[Boot-UKI]]

### Opsi 2 — GRUB

Pakai GRUB kalau kamu mau bootloader umum, gampang dipahami, dan enak buat dual boot.

Baca:

- [[Boot-GRUB]]

> Pilih salah satu dulu. Jangan install dua-duanya kalau belum paham boot order, nanti BIOS jadi pasar malam.

---

## 13. Finalisasi

Setelah bootloader selesai dipasang, keluar dari chroot:

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

Cabut USB installer ketika sistem mulai restart.

---

## Catatan Lanjutan

Untuk setup lanjut:

- Desktop Environment: Hyprland, GNOME, KDE
- Audio: PipeWire
- GPU driver
- Secure Boot
- Snapshot btrfs

---

## Related

- Arch Wiki: <https://wiki.archlinux.org/>
- Installation Guide: <https://wiki.archlinux.org/title/Installation_guide>
