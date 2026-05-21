# Boot dengan UKI

> UKI cocok kalau kamu mau setup Arch Linux yang modern, bersih, dan enak dipasangkan dengan Secure Boot.

---

## Apa Itu UKI?

UKI adalah singkatan dari **Unified Kernel Image**.

Singkatnya, UKI menggabungkan beberapa komponen boot menjadi satu file `.efi`, biasanya berisi:

- kernel Linux
- initramfs
- microcode
- kernel command line

File `.efi` ini bisa langsung diboot oleh firmware UEFI atau boot manager.

---

## Kapan Pakai UKI?

Pakai UKI kalau:

- Kamu ingin setup Secure Boot.
- Kamu ingin boot langsung dari file `.efi`.
- Kamu ingin sistem boot yang lebih simpel tanpa menu bootloader ribet.
- Kamu tidak butuh fitur kompleks GRUB.

UKI cocok banget untuk setup Arch modern karena kernel dan initramfs bisa dikemas jadi satu file EFI.

---

## Kekurangan UKI

UKI bukan berarti selalu lebih gampang.

Kekurangannya:

- Setup awal lebih sensitif.
- Kernel command line harus benar.
- Kalau salah root UUID atau rootflags, sistem bisa gagal boot.
- Untuk dual boot, kadang perlu atur boot entry UEFI dengan lebih teliti.

Kalau kamu masih sering bongkar partisi kayak orang gabut, GRUB kadang lebih santai.

---

## 1. Install Paket yang Dibutuhkan

Di dalam `arch-chroot`:

```bash
pacman -S systemd
```

Biasanya `systemd` sudah ikut terinstall dari base system, tapi command ini aman buat memastikan.

Kalau pakai Intel CPU:

```bash
pacman -S intel-ucode
```

Kalau pakai AMD CPU:

```bash
pacman -S amd-ucode
```

---

## 2. Buat Kernel Command Line

Cek UUID root:

```bash
blkid
```

Cari UUID dari partisi root, contoh:

```txt
/dev/nvme0n1p2: UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Buat file:

```bash
nano /etc/kernel/cmdline
```

---

## Jika Root Pakai ext4

Isi:

```txt
root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw
```

---

## Jika Root Pakai btrfs

Isi:

```txt
root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rootflags=subvol=@ rw
```

> Jangan pakai UUID EFI. Pakai UUID partisi root. Ini kesalahan klasik yang bikin boot gagal terus.

---

## 3. Edit Preset mkinitcpio

Buka preset kernel:

```bash
nano /etc/mkinitcpio.d/linux.preset
```

Ubah menjadi seperti ini:

```bash
# mkinitcpio preset file for the 'linux' package

ALL_kver="/boot/vmlinuz-linux"

PRESETS=('default' 'fallback')

default_uki="/boot/EFI/Linux/arch-linux.efi"
default_options="--splash=/usr/share/systemd/bootctl/splash-arch.bmp"

fallback_uki="/boot/EFI/Linux/arch-linux-fallback.efi"
fallback_options="-S autodetect"
```

Pastikan folder tujuan ada:

```bash
mkdir -p /boot/EFI/Linux
```

Generate UKI:

```bash
mkinitcpio -P
```

Cek hasilnya:

```bash
ls -lah /boot/EFI/Linux
```

Harus ada file seperti:

```txt
arch-linux.efi
arch-linux-fallback.efi
```

---

## 4. Buat Entry UEFI Langsung

Cek disk dan partisi EFI:

```bash
lsblk
```

Contoh:

- Disk: `/dev/nvme0n1`
- EFI: partisi pertama, berarti `--part 1`

Buat boot entry:

```bash
efibootmgr --create \
  --disk /dev/nvme0n1 \
  --part 1 \
  --label "Arch Linux UKI" \
  --loader '\EFI\Linux\arch-linux.efi'
```

Cek boot entry:

```bash
efibootmgr
```

---

## 5. Secure Boot

UKI enak untuk Secure Boot karena yang perlu ditandatangani cukup file `.efi` hasil gabungan tadi.

Konsepnya:

1. Buat UKI.
2. Sign file UKI.
3. Daftarkan key Secure Boot.
4. Firmware hanya menjalankan file EFI yang sudah valid/sign.

Untuk setup Secure Boot biasanya bisa pakai `sbctl`.

Install:

```bash
pacman -S sbctl
```

Cek status:

```bash
sbctl status
```

Buat key:

```bash
sbctl create-keys
```

Enroll key:

```bash
sbctl enroll-keys -m
```

Sign UKI:

```bash
sbctl sign -s /boot/EFI/Linux/arch-linux.efi
sbctl sign -s /boot/EFI/Linux/arch-linux-fallback.efi
```

Cek file yang sudah terdaftar:

```bash
sbctl list-files
```

Setiap kernel update, UKI akan dibuat ulang. Pastikan file UKI yang baru tetap signed.

---

## 6. Reboot

Keluar dari chroot:

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

## Troubleshooting

### UKI Tidak Muncul di BIOS

Cek apakah file ada:

```bash
ls -lah /boot/EFI/Linux
```

Cek boot entry:

```bash
efibootmgr
```

Kalau belum ada entry, buat ulang:

```bash
efibootmgr --create \
  --disk /dev/nvme0n1 \
  --part 1 \
  --label "Arch Linux UKI" \
  --loader '\EFI\Linux\arch-linux.efi'
```

---

### Boot Gagal Karena Root Tidak Ketemu

Cek `/etc/kernel/cmdline`:

```bash
cat /etc/kernel/cmdline
```

Pastikan:

- UUID yang dipakai adalah UUID root.
- Kalau btrfs, ada `rootflags=subvol=@`.
- Tidak typo.
- Tidak pakai tanda kutip.

Generate ulang:

```bash
mkinitcpio -P
```

---

### Secure Boot Gagal

Cek status:

```bash
sbctl status
```

Cek file signed:

```bash
sbctl verify
```

Kalau UKI belum signed:

```bash
sbctl sign -s /boot/EFI/Linux/arch-linux.efi
```

---

## Kesimpulan

Pakai UKI kalau kamu mau boot modern dan siap main Secure Boot.

UKI lebih bersih dari GRUB, tapi kalau salah konfigurasi, error-nya bisa bikin pengen ngomong kasar ke laptop sendiri.

---

## Related

- Arch Wiki UKI: <https://wiki.archlinux.org/title/Unified_kernel_image>
- Arch Wiki mkinitcpio: <https://wiki.archlinux.org/title/Mkinitcpio>
