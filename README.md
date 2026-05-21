# Dokumentasi Arch Linux

> Catatan instalasi dan setup Arch Linux berbasis UEFI, ditulis biar proses install nggak berubah jadi ritual pemanggilan error jam 2 pagi.

Repository ini berisi dokumentasi instalasi Arch Linux manual dengan struktur yang dipisah per topik. Cocok buat setup minimal, dual boot, Btrfs, Snapper, GRUB, UKI, dan persiapan Secure Boot.

---

## Daftar Isi

| Dokumen | Isi |
| ------- | --- |
| [Arch-Install.md](Arch-Install.md) | Panduan utama instalasi Arch Linux dari live ISO sampai konfigurasi dasar sistem |
| [Boot-GRUB.md](Boot-GRUB.md) | Setup bootloader GRUB untuk UEFI, termasuk catatan dual boot Windows |
| [Boot-UKI.md](Boot-UKI.md) | Setup Unified Kernel Image atau UKI, cocok untuk sistem modern dan Secure Boot |
| [Setup-btrfs.md](Setup-btrfs.md) | Setup Btrfs, subvolume, Snapper, snapshot, rollback, dan integrasi GRUB |

---

## Struktur Dokumentasi

```text
.
├── README.md
├── Arch-Install.md
├── Boot-GRUB.md
├── Boot-UKI.md
└── Setup-btrfs.md
```

---

## Rekomendasi Urutan Baca

Kalau kamu install dari nol, ikuti urutan ini:

```text
1. Arch-Install.md
2. Pilih salah satu:
   ├── Boot-GRUB.md
   └── Boot-UKI.md
3. Setup-btrfs.md
```

Jangan langsung lompat ke Snapper kalau sistemnya aja belum bisa boot. Itu namanya ambisi, bukan skill.

---

## Pilihan Boot

Ada dua opsi boot yang dipisah supaya dokumentasi tetap rapi.

### 1. GRUB

Pilih GRUB kalau kamu ingin:

- Setup yang umum dan gampang dicari solusinya
- Dual boot dengan Windows
- Menu boot yang jelas
- Integrasi mudah dengan Snapper lewat `grub-btrfs`

Baca: [Boot-GRUB.md](Boot-GRUB.md)

### 2. UKI

Pilih UKI kalau kamu ingin:

- Setup boot yang lebih modern
- File boot `.efi` yang lebih ringkas
- Alur yang lebih cocok untuk Secure Boot
- Sistem minimal tanpa menu bootloader besar

Baca: [Boot-UKI.md](Boot-UKI.md)

> Catatan: UKI bagus untuk Secure Boot, tapi tidak senyaman GRUB kalau target kamu adalah boot langsung ke snapshot Btrfs dari menu.

---

## Btrfs + Snapper

Dokumentasi Btrfs dan Snapper ada di:

[Setup-btrfs.md](Setup-btrfs.md)

Topik yang dibahas:

- Konsep snapshot
- Struktur subvolume
- Setup Snapper
- Auto snapshot
- Cleanup snapshot
- Rollback system
- Integrasi GRUB dengan `grub-btrfs`
- Catatan khusus jika memakai UKI

> Snapshot bukan backup.
> Kalau disk rusak, snapshot juga ikut hilang. Jangan sok aman cuma karena punya snapshot, nanti nangis di pojokan.

---

## Target Setup

Dokumentasi ini fokus ke:

- Arch Linux
- UEFI
- Root filesystem ext4 atau Btrfs
- GRUB atau UKI
- Sistem minimal yang bisa dikembangkan lagi
- Setup yang masih manusiawi buat dibaca ulang

---

## Catatan Penting

Sebelum mengikuti dokumentasi ini, pastikan kamu paham beberapa hal:

- Nama disk bisa berbeda, misalnya `/dev/nvme0n1`, `/dev/sda`, atau `/dev/vda`
- Salah format partisi bisa menghapus data
- Backup data penting sebelum install
- Jangan copy command partisi secara brutal
- Baca ulang command sebelum menekan Enter

Kalau kamu salah format disk Windows terus nyalahin dokumentasi, itu bukan bug. Itu skill issue.

---

## Setelah Install

Setelah sistem dasar berhasil boot, kamu bisa lanjut setup:

- Desktop Environment atau Window Manager
- GPU driver
- Audio dengan PipeWire
- Bluetooth
- Printer
- Firewall
- Dotfiles
- Secure Boot
- Backup eksternal

---

## Related Links

- [Arch Linux](https://archlinux.org/)
- [Arch Wiki](https://wiki.archlinux.org/)
- [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- [Btrfs - Arch Wiki](https://wiki.archlinux.org/title/Btrfs)
- [Snapper - Arch Wiki](https://wiki.archlinux.org/title/Snapper)
- [GRUB - Arch Wiki](https://wiki.archlinux.org/title/GRUB)
- [Unified Kernel Image - Arch Wiki](https://wiki.archlinux.org/title/Unified_kernel_image)

---

## Disclaimer

Dokumentasi ini dibuat sebagai catatan pribadi dan referensi belajar.

Gunakan dengan sadar. Baca command sebelum dijalankan. Kalau ragu, cek Arch Wiki.

Arch itu kuat, tapi dia nggak akan menyelamatkan orang yang copy-paste sambil merem.
