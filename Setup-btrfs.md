# Btrfs + Snapper Setup  
> Automatic snapshot & rollback system untuk Arch Linux (atau distro lain berbasis Btrfs)

---

## Konsep Singkat

- **Btrfs** menggunakan sistem *Copy-on-Write (CoW)*  
- Snapshot **bukan full copy**, tapi hanya menyimpan perubahan  
- Snapshot:
  - ⚡ cepat dibuat
  - 💾 hemat storage
- **Snapper** = tool untuk otomatisasi snapshot & cleanup

> ⚠️ Snapshot ≠ Backup  
> Jika disk rusak, semua snapshot ikut hilang

---

## Install Package

Jalankan setelah install system:

```bash
pacman -S snapper btrfs-progs
````

---

## Struktur Subvolume

Pastikan kamu punya struktur seperti ini:

```
@
@home
@snapshots
@cache
@log
```

Cek dengan:

```bash
btrfs subvolume list /
```

Kalau belum ada:

```bash
btrfs subvolume create /.snapshots
```

---

## Setup Snapper (Root)

Buat konfigurasi:

```bash
snapper -c root create-config /
```

Ini akan:

- Membuat config di `/etc/snapper/configs/root`
- Setup `.snapshots` (bisa override manual setup)

---

## 🔐 Fix Permission

```bash
chmod 750 /.snapshots
chown :wheel /.snapshots
```

---

## ⏱️ Enable Auto Snapshot

```bash
systemctl enable --now snapper-timeline.timer
systemctl enable --now snapper-cleanup.timer
```

### Default Behavior

- Snapshot:
    - tiap jam
    - harian
    - mingguan
    - bulanan
- Cleanup otomatis berdasarkan limit
    

---

## Konfigurasi (Optional)

Edit:

```bash
nano /etc/snapper/configs/root
```

Contoh:

```ini
TIMELINE_CREATE="yes"
TIMELINE_CLEANUP="yes"

TIMELINE_LIMIT_HOURLY="5"
TIMELINE_LIMIT_DAILY="7"
TIMELINE_LIMIT_WEEKLY="4"
TIMELINE_LIMIT_MONTHLY="3"
```

---

## Snapshot Manual

### Buat snapshot

```bash
snapper -c root create --description "before update"
```

### List snapshot

```bash
snapper list
```

---

## Rollback System

```bash
snapper rollback
```

📌 Yang terjadi:

- Snapshot dijadikan root baru
- Root lama tetap disimpan sebagai snapshot
- Perlu reboot

> ⚠️ Perubahan setelah snapshot bisa hilang

---

## Integrasi GRUB (Recommended)

Install:

```bash
pacman -S grub-btrfs inotify-tools
```

Enable daemon:

```bash
systemctl enable --now grub-btrfsd
```

Generate config:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

✅ Hasil:

- Snapshot muncul di menu boot
- Bisa rollback tanpa masuk system

> ⚠️ Jika daemon mati, snapshot baru tidak muncul di GRUB

---

## fstab (Penting)

Pastikan subvolume di-mount dengan benar:

```fstab
UUID=xxx / btrfs subvol=@,compress=zstd,noatime 0 0
UUID=xxx /home btrfs subvol=@home 0 0
UUID=xxx /.snapshots btrfs subvol=@snapshots 0 0
```

---

## Best Practice

✔ Snapshot sebelum update:

```bash
snapper create --description "pre update"
```

✔ Pisahkan `/home`:

```bash
snapper -c home create-config /home
```

✔ Jaga free space disk

> Btrfs butuh ruang kosong untuk performa optimal

---

## Common Mistakes

- Tidak mount `.snapshots` sebagai subvolume
- Disk terlalu penuh
- Salah config `fstab`
- Mengira snapshot = backup

---

## Workflow yang Direkomendasikan

```
1. Update system
2. Kalau error → rollback
3. Kalau aman → lanjut kerja
```

---

## Struktur Snapshot (Ilustrasi)

```
@snapshots
├── 1/
├── 2/
├── 3/
```

---

## Related

- Snapper
- Btrfs
- GRUB