# DigiRoomOS GNU/Linux — Remaster Xubuntu via Cubic (APT-only, tanpa Plymouth)

_Terakhir di-update: 2025-12-15_

Dokumen ini adalah panduan langkah-per-langkah (model tutorial) untuk meremaster **Xubuntu (base)** menjadi **DigiRoomOS (Digital Classroom)** memakai **Cubic**.

Fokus:
- Hapus bloatware (games/torrent/chat/music tertentu)
- Wallpaper **Desktop + Login (LightDM)** (metode “TOTAL WAR”)
- GRUB background untuk **boot menu ISO** (custom-disk)
- Fastfetch logo ASCII (pnj.png → txt via chafa)
- Paket “Digital Classroom” **tanpa Flatpak** (APT / .deb via `wget` + `apt install`)

> Catatan penting: Plymouth **tidak kita sentuh** (biarkan default).

---

## 0) Istilah penting (biar nggak salah tempat edit)

Di project Cubic ada 2 area:

1. **`custom-root/` (chroot / filesystem sistem)**  
   Ini isi OS yang nanti dipakai di Live dan setelah install (aplikasi, `/etc`, wallpaper, fastfetch, dll).

2. **`custom-disk/` (struktur ISO boot)**  
   Ini yang mengatur **tampilan menu GRUB saat boot ISO** (flashdisk/VM).  
   Jadi: GRUB background ISO **editnya di `custom-disk`**.

---

## 1) Persiapan aset (di dalam Cubic)

Siapkan 3 file berikut, lalu taruh ke folder ini di **Terminal Cubic** (chroot):

- `/root/assets/wallpaper.png`
- `/root/assets/pnj.png`
- `/root/assets/logo.png`

Cek cepat:

```bash
ls -lh /root/assets/
file /root/assets/wallpaper.png /root/assets/pnj.png /root/assets/logo.png
```

---

## 2) Tahap A — Customize di Terminal Cubic (chroot)

> Semua perintah di bagian ini dijalankan **di Terminal Cubic** (You have entered the virtual environment).
>
> Di Terminal Cubic biasanya kamu sudah `root`. Kalau suatu perintah gagal karena `sudo: command not found`, cukup **hapus `sudo`** dari baris tersebut.

### 2.1 Copy aset ke lokasi final DigiRoomOS

```bash
sudo install -d /usr/share/digiroom/wallpapers
sudo install -d /usr/share/digiroom/branding

sudo cp -f /root/assets/wallpaper.png /usr/share/digiroom/wallpapers/wallpaper.png
sudo cp -f /root/assets/pnj.png       /usr/share/digiroom/branding/pnj.png
sudo cp -f /root/assets/logo.png      /usr/share/digiroom/branding/logo.png
```

Cek:

```bash
ls -lh /usr/share/digiroom/wallpapers/wallpaper.png
ls -lh /usr/share/digiroom/branding/{pnj.png,logo.png}
```

---

### 2.2 Enable repo Universe/Multiverse (biar paket kreatif/edu banyak yang ada)

```bash
sudo apt-get update
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y universe
sudo add-apt-repository -y multiverse
sudo apt-get update
```

---

### 2.3 Install tools inti (wajib untuk project ini)

```bash
sudo apt-get install -y fastfetch chafa imagemagick papirus-icon-theme vlc
```

---

### 2.4 Hapus bloatware “aman” (silakan edit list)

```bash
sudo apt-get purge -y   gnome-mines gnome-sudoku sgt-puzzles sgt-launcher   transmission-gtk transmission-common   hexchat hexchat-common hexchat-plugins   rhythmbox rhythmbox-data rhythmbox-plugins   thunderbird   || true

sudo apt-get autoremove -y --purge
sudo apt-get clean
```

> Catatan: kalau kamu masih butuh LibreOffice, **jangan** purge `libreoffice*`.

---

### 2.5 Fastfetch — pnj.png → ASCII txt (chafa)

```bash
sudo chafa -c full -s 40x20 /usr/share/digiroom/branding/pnj.png | sudo tee /usr/share/digiroom/branding/pnj.txt >/dev/null
```

Cek:

```bash
head -n 20 /usr/share/digiroom/branding/pnj.txt
```

---

### 2.6 Fastfetch config (default untuk user baru + root)

```bash
sudo install -d /etc/skel/.config/fastfetch
sudo install -d /root/.config/fastfetch
```

Buat file config:

```bash
sudo tee /etc/skel/.config/fastfetch/config.jsonc >/dev/null <<'EOF'
{
  "logo": { "type": "file", "source": "/usr/share/digiroom/branding/pnj.txt" },
  "display": { "separator": " : " },
  "modules": [
    "title", "separator",
    "os", "kernel", "uptime",
    "packages", "shell", "terminal",
    "cpu", "gpu", "memory", "disk", "localip"
  ]
}
EOF
```

Copy ke root:

```bash
sudo cp -f /etc/skel/.config/fastfetch/config.jsonc /root/.config/fastfetch/config.jsonc
```

Test:

```bash
fastfetch --show-errors
```

---

### 2.7 Wallpaper Desktop — metode “TOTAL WAR” (yang kamu pakai & sudah terbukti)

> Metode ini menimpa file backdrop bawaan Xubuntu, jadi wallpaper “nempel” walau XFCE config beda-beda.

```bash
sudo cp -f /root/assets/wallpaper.png /tmp/my-wall.png
```

Target utama (Questing):

```bash
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

Timpa `xubuntu-wallpaper.png` (kadang symlink / file target):

```bash
sudo rm -f /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
```

Jaga-jaga (varian lain):

```bash
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-development.png || true
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-plucky.png || true
```

Cek ukuran:

```bash
ls -lh /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

---

### 2.8 Wallpaper Login (LightDM GTK Greeter)

```bash
WALL="/usr/share/digiroom/wallpapers/wallpaper.png"
GREETER="/etc/lightdm/lightdm-gtk-greeter.conf"

sudo touch "$GREETER"
```

Pastikan section `[greeter]` ada:

```bash
grep -q "^\[greeter\]" "$GREETER" || echo "[greeter]" | sudo tee -a "$GREETER" >/dev/null
```

Set background:

```bash
if grep -q "^background=" "$GREETER"; then
  sudo sed -i "s|^background=.*|background=$WALL|" "$GREETER"
else
  sudo sed -i "/^\[greeter\]/a background=$WALL" "$GREETER"
fi
```

Cek:

```bash
grep -n "^\[greeter\]\|^background=" "$GREETER"
```

---

### 2.9 Branding OS (cara aman — **jangan ubah ID**)

Beberapa installer bisa error kalau `ID=` di `/etc/os-release` diganti dari `ubuntu`.
Jadi kita ubah **NAME** dan **PRETTY_NAME** saja.

(Opsional) reinstall base-files dulu:

```bash
sudo apt-get install -y --reinstall base-files
```

Ubah `/etc/os-release`:

```bash
sudo sed -i   -e 's/^NAME=.*/NAME="DigiRoomOS GNU\/Linux"/'   -e 's/^PRETTY_NAME=.*/PRETTY_NAME="DigiRoomOS GNU\/Linux (Xubuntu base)"/'   /etc/os-release
```

Ubah `/etc/lsb-release` (kalau ada):

```bash
sudo sed -i   's/^DISTRIB_DESCRIPTION=.*/DISTRIB_DESCRIPTION="DigiRoomOS GNU\/Linux (Xubuntu base)"/'   /etc/lsb-release || true
```

Pastikan symlink:

```bash
sudo ln -sf /etc/os-release /usr/lib/os-release
```

Cek:

```bash
grep -E '^(NAME|PRETTY_NAME|ID|ID_LIKE)=' /etc/os-release
```

---

## 3) Tahap B — Paket “Digital Classroom” (APT / .deb)

> Semua perintah di bagian ini juga dijalankan di **Terminal Cubic (chroot)**.

### 3.1 Produktivitas & dokumen

```bash
sudo apt-get install -y libreoffice
sudo apt-get install -y okular evince
sudo apt-get install -y hunspell-id
```

### 3.2 Coding & Dev tools (open-source)

```bash
sudo apt-get install -y build-essential git curl wget
sudo apt-get install -y python3 python3-pip
sudo apt-get install -y default-jdk
sudo apt-get install -y nodejs npm
sudo apt-get install -y geany geany-plugins
sudo apt-get install -y codeblocks
sudo apt-get install -y vim nano
sudo apt-get install -y thonny
sudo apt-get install -y cmake gdb valgrind
```

Opsional (kalau ada):

```bash
sudo apt-get install -y kate || true
sudo apt-get install -y meld || true
sudo apt-get install -y eclipse || true
```

### 3.3 Multimedia & kreatif (open-source)

```bash
sudo apt-get install -y gimp inkscape
sudo apt-get install -y audacity
sudo apt-get install -y kdenlive
sudo apt-get install -y openshot || true
sudo apt-get install -y blender
sudo apt-get install -y ffmpeg
sudo apt-get install -y mpv || true
```

### 3.4 Rekam layar & presentasi

```bash
sudo apt-get install -y obs-studio
sudo apt-get install -y simplescreenrecorder || true
sudo apt-get install -y flameshot || true
```

### 3.5 Edukasi / Digital Classroom

```bash
sudo apt-get install -y scratch
sudo apt-get install -y stellarium
```

Opsional (kalau tersedia):

```bash
sudo apt-get install -y gcompris-qt || true
sudo apt-get install -y tuxpaint tuxmath || true
```

### 3.6 Browser (repo Ubuntu)

Xubuntu biasanya sudah punya Firefox. Kalau kamu butuh browser kedua **tanpa Snap**, biasanya pilihan aman adalah pakai Firefox saja.
`chromium/chromium-browser` di Ubuntu sering diarahkan ke Snap, jadi **tidak saya rekomendasikan** untuk target “APT-only”.

---

### 3.7 Zoom Desktop (download .deb + `apt install`)

Zoom tidak ada di repo Ubuntu default, jadi opsi paling “rapi” adalah install dari `.deb` resmi:

```bash
cd /tmp
wget -O zoom_amd64.deb https://zoom.us/client/latest/zoom_amd64.deb
sudo apt-get update
sudo apt install -y ./zoom_amd64.deb
```

Cek:

```bash
command -v zoom || true
```

Kalau kamu mau **tanpa aplikasi Zoom**, alternatifnya: pakai **Zoom Web** di Firefox/Chrome.

---

### 3.8 Google Chrome (download .deb + `apt install`)

Chrome juga bukan repo default. Install dari `.deb` resmi:

```bash
cd /tmp
wget -O google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt-get update
sudo apt install -y ./google-chrome.deb
```

Cek:

```bash
command -v google-chrome || true
```

---

## 4) Tahap C — Shortcut Desktop per link (untuk Live + setelah Install)

Agar shortcut muncul di Desktop user live dan user baru setelah install, taruh `.desktop` di **`/etc/skel/Desktop`**.

Buat folder:

```bash
sudo install -d /etc/skel/Desktop
```

### 4.1 Template launcher (pakai Chrome kalau ada, fallback ke Firefox)

> Ini bikin shortcut tetap jalan meskipun Chrome belum diinstall.

**Academia PNJ**

```bash
sudo tee /etc/skel/Desktop/academia-pnj.desktop >/dev/null <<'EOF'
[Desktop Entry]
Type=Application
Name=Academia PNJ
Comment=Portal akademik PNJ
Exec=sh -c 'if command -v google-chrome >/dev/null 2>&1; then google-chrome "https://academia.pnj.ac.id/"; else firefox "https://academia.pnj.ac.id/"; fi'
Icon=web-browser
Terminal=false
Categories=Network;Education;
EOF
```

**Website PNJ**

```bash
sudo tee /etc/skel/Desktop/website-pnj.desktop >/dev/null <<'EOF'
[Desktop Entry]
Type=Application
Name=Website PNJ
Comment=Website resmi PNJ
Exec=sh -c 'if command -v google-chrome >/dev/null 2>&1; then google-chrome "https://pnj.ac.id/"; else firefox "https://pnj.ac.id/"; fi'
Icon=web-browser
Terminal=false
Categories=Network;Education;
EOF
```

**Google Scholar**

```bash
sudo tee /etc/skel/Desktop/google-scholar.desktop >/dev/null <<'EOF'
[Desktop Entry]
Type=Application
Name=Google Scholar
Comment=Cari jurnal & referensi
Exec=sh -c 'if command -v google-chrome >/dev/null 2>&1; then google-chrome "https://scholar.google.com/"; else firefox "https://scholar.google.com/"; fi'
Icon=web-browser
Terminal=false
Categories=Network;Education;
EOF
```

**Google Meet**

```bash
sudo tee /etc/skel/Desktop/google-meet.desktop >/dev/null <<'EOF'
[Desktop Entry]
Type=Application
Name=Google Meet
Comment=Ruang meeting online
Exec=sh -c 'if command -v google-chrome >/dev/null 2>&1; then google-chrome "https://meet.google.com/landing"; else firefox "https://meet.google.com/landing"; fi'
Icon=web-browser
Terminal=false
Categories=Network;Education;
EOF
```

**E-learning PNJ** (ganti URL sesuai e-learning yang kamu pakai)

```bash
sudo tee /etc/skel/Desktop/elearning-pnj.desktop >/dev/null <<'EOF'
[Desktop Entry]
Type=Application
Name=E-learning PNJ
Comment=LMS / e-learning PNJ
Exec=sh -c 'if command -v google-chrome >/dev/null 2>&1; then google-chrome "https://elearning.pnj.ac.id/"; else firefox "https://elearning.pnj.ac.id/"; fi'
Icon=web-browser
Terminal=false
Categories=Network;Education;
EOF
```

Bikin executable:

```bash
sudo chmod +x /etc/skel/Desktop/*.desktop
```

> Di XFCE, kalau shortcut belum bisa diklik, biasanya tinggal **Right click → Allow Launching**.

---

## 5) Tahap D — GRUB background untuk Boot ISO (dikerjakan di HOST, bukan chroot)

> Bagian ini kamu jalankan di **host OS** (Kali/Ubuntu host), di folder project Cubic kamu (contoh: `~/izin`).

### 5.1 Copy wallpaper ke custom-disk

```bash
cd ~/izin
sudo cp -f custom-root/usr/share/digiroom/wallpapers/wallpaper.png custom-disk/boot/grub/wallpaper.png
ls -lh custom-disk/boot/grub/wallpaper.png
```

### 5.2 Edit `custom-disk/boot/grub/grub.cfg`

Tambahkan blok ini **setelah** `loadfont unicode`:

```cfg
insmod all_video
insmod gfxterm
insmod gfxterm_background
insmod png
set gfxmode=auto
terminal_output gfxterm
background_image /boot/grub/wallpaper.png
```

### 5.3 Edit `custom-disk/boot/grub/loopback.cfg`

Tambahkan blok yang sama **di paling atas file** (sebelum menuentry pertama).

### 5.4 Verifikasi cepat

```bash
grep -n "background_image" custom-disk/boot/grub/grub.cfg custom-disk/boot/grub/loopback.cfg
```

Harus muncul 2 baris (satu di `grub.cfg`, satu di `loopback.cfg`).

---

## 6) Build ISO + Test

1. Kembali ke Cubic → lanjut ke langkah **Generate** (build ISO).
2. Test di VirtualBox/QEMU:
   - GRUB menu ISO tampil + background muncul
   - Live Desktop wallpaper sudah berubah
   - Login screen (LightDM) background berubah
   - Buka terminal → `fastfetch` tidak error
   - Coba **Install** (kalau installer error, lihat catatan di bawah)

---

## 7) Troubleshooting cepat

### 7.1 Installer “Something went wrong” (ubuntu-desktop-bootstrap)

Hal yang paling sering bikin installer crash di ISO remaster:
- Mengganti `ID=` di `/etc/os-release` jadi bukan `ubuntu`
- Menghapus paket yang ternyata dipakai installer (jarang, tapi bisa)

Solusi aman (yang kamu sudah buktikan jalan):
- Biarkan `ID=ubuntu`
- Ubah hanya `NAME` dan `PRETTY_NAME` pakai `sed` (lihat Step 2.9)

Kalau masih error:
- klik **Show log** di installer, screenshot/log-nya simpan untuk analisis
- coba install ulang paket inti (di chroot) yang berkaitan dengan installer:
  ```bash
  sudo apt-get install -y --reinstall base-files
  ```

---

Selesai. ✅
