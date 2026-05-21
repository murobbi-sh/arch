# Boot dengan GRUB UEFI

> GRUB cocok kalau kamu ingin bootloader yang umum, fleksibel, dan gampang dipakai untuk dual boot.

---

## Apa Itu GRUB?

GRUB adalah bootloader yang umum dipakai di Linux.

Dia bertugas menampilkan menu boot dan menjalankan kernel Linux. GRUB juga sering dipakai untuk dual boot karena bisa mendeteksi sistem operasi lain seperti Windows.

---

## Kapan Pakai GRUB?

Pakai GRUB kalau:

- Kamu masih pemula.
- Kamu dual boot dengan Windows.
- Kamu ingin menu boot yang jelas.
- Kamu sering gonta-ganti kernel.
- Kamu belum mau ribet Secure Boot.

GRUB itu bukan paling modern, tapi paling “yaudah jalan dulu lah”.

---

## Kekurangan GRUB

GRUB juga bukan malaikat.

Kekurangannya:

- Setup Secure Boot lebih ribet dibanding UKI.
- Konfigurasinya lebih banyak.
- Kadang menu boot double kalau salah install atau bekas entry lama belum dibersihin.

---

## 1. Install Paket GRUB

Di dalam `arch-chroot`:

```bash
pacman -S grub efibootmgr
```

Kalau pakai Intel CPU:

```bash
pacman -S intel-ucode
```

Kalau pakai AMD CPU:

```bash
pacman -S amd-ucode
```

---

## 2. Install GRUB ke EFI

Pastikan EFI mounted di `/boot`.

Cek:

```bash
lsblk
```

Install GRUB:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Generate config:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 3. Jika Pakai Dual Boot Windows

Install `os-prober`:

```bash
pacman -S os-prober
```

Edit config GRUB:

```bash
nano /etc/default/grub
```

Cari atau tambahkan:

```txt
GRUB_DISABLE_OS_PROBER=false
```

Generate ulang:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Kalau Windows terdeteksi, nanti akan muncul entry Windows Boot Manager di menu GRUB.

---

## 4. Reboot

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

### Error: ESP Doesn't Look Like an EFI Partition

Artinya partisi EFI kemungkinan:

- Belum diformat FAT32.
- Salah mount.
- Bukan EFI System Partition.

Cek:

```bash
lsblk -f
```

Pastikan EFI mounted di `/boot`:

```bash
mount | grep boot
```

---

### GRUB Tidak Muncul di BIOS

Cek boot entry:

```bash
efibootmgr
```

Kalau tidak ada entry GRUB, install ulang:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

---

### Windows Tidak Muncul

Pastikan Windows Boot Manager masih ada di EFI:

```bash
ls /boot/EFI/Microsoft/Boot
```

Install `os-prober`:

```bash
pacman -S os-prober
```

Aktifkan:

```bash
nano /etc/default/grub
```

Isi:

```txt
GRUB_DISABLE_OS_PROBER=false
```

Generate ulang:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Secure Boot dengan GRUB

GRUB bisa dipakai dengan Secure Boot, tapi setup-nya lebih ribet daripada UKI.

Kalau target kamu Secure Boot yang rapi, lebih enak pakai UKI.

Kalau target kamu gampang boot dulu, dual boot dulu, hidup dulu, waras dulu, pakai GRUB.

---

## Kesimpulan

Pakai GRUB kalau kamu ingin cara yang umum, fleksibel, dan gampang buat dual boot.

Pakai UKI kalau kamu ingin setup modern dan lebih enak buat Secure Boot.

---

## Related

- [Arch-Install.md](./Arch-Install.md)
- [Boot-GRUB.md](./Boot-GRUB.md)
- [Boot-UKI.md](./Boot-UKI.md)
