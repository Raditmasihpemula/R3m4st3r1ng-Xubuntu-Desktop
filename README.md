# DigiRoomOS GNU/Linux (Xubuntu remaster) — Cubic Guide

Panduan ini dibuat biar kamu bisa **remaster Xubuntu lewat Cubic** jadi **DigiRoomOS (Digital Classroom)** dengan hasil yang konsisten:
- Hapus bloatware
- Pasang wallpaper **desktop + login**
- Set **GRUB background** untuk boot ISO
- Setup **fastfetch** pakai ASCII logo (pnj.png → txt via chafa)
- (Plymouth **tidak disentuh** / pakai default)

> Catatan: Perintah di bawah ditulis “model tutorial” (diketik per baris), **bukan satu script panjang**.

---

## Struktur Cubic yang wajib paham

Di project Cubic ada 2 area penting:

1) **`custom-root/` (chroot)**  
Ini isi filesystem OS yang nanti jadi sistem live/installer (paket, config `/etc`, wallpaper, fastfetch, dll).

2) **`custom-disk/` (ISO boot files)**  
Ini khusus untuk **boot menu ISO** (GRUB saat boot dari flashdisk/VM).  
👉 Jadi **GRUB background harus diedit di `custom-disk`**, bukan di `/etc/default/grub` dalam chroot.

---

## Prasyarat

- Host Linux (contoh kamu: Kali).
- Cubic sudah jalan dan project sudah dibuat dari ISO Xubuntu.
- Kamu punya 3 aset (contoh nama file):
  - `wallpaper.png`
  - `pnj.png`
  - `logo.png`

---

## A. Taruh aset di Cubic (dalam chroot)

Di halaman **Cubic → Customize → Terminal** (yang prompt-nya `root@cubic`), pastikan folder assets ada:

```bash
mkdir -p /root/assets
ls -lah /root/assets
```

Lalu **drag & drop** file kamu ke folder `/root/assets/` (atau copy manual kalau kamu punya cara lain).

Cek:

```bash
ls -lah /root/assets
file /root/assets/wallpaper.png /root/assets/pnj.png /root/assets/logo.png
```

---

## B. Customisasi di Cubic Terminal (chroot)

### B1) Buat folder brand + salin aset ke lokasi final

```bash
install -d /usr/share/digiroom/wallpapers
install -d /usr/share/digiroom/branding

cp -f /root/assets/wallpaper.png /usr/share/digiroom/wallpapers/wallpaper.png
cp -f /root/assets/pnj.png       /usr/share/digiroom/branding/pnj.png
cp -f /root/assets/logo.png      /usr/share/digiroom/branding/logo.png

ls -lah /usr/share/digiroom/wallpapers
ls -lah /usr/share/digiroom/branding
```

---

### B2) Update repo + install tools dasar

```bash
apt-get update

apt-get install -y   fastfetch chafa imagemagick papirus-icon-theme vlc
```

---

### B3) Hapus bloatware (aman, bisa kamu edit)

```bash
apt-get purge -y   gnome-mines gnome-sudoku sgt-puzzles sgt-launcher   transmission-gtk transmission-common   hexchat hexchat-common hexchat-plugins   rhythmbox rhythmbox-data rhythmbox-plugins   thunderbird libreoffice*   || true

apt-get autoremove -y --purge
apt-get clean
```

---

### B4) Fastfetch: buat ASCII logo (pnj.png → pnj.txt)

```bash
chafa -c full -s 40x20 /usr/share/digiroom/branding/pnj.png > /usr/share/digiroom/branding/pnj.txt
wc -l /usr/share/digiroom/branding/pnj.txt
head -n 5 /usr/share/digiroom/branding/pnj.txt
```

---

### B5) Fastfetch config (untuk user baru + root)

Buat folder config:

```bash
install -d /etc/skel/.config/fastfetch
install -d /root/.config/fastfetch
```

Tulis config:

```bash
cat > /etc/skel/.config/fastfetch/config.jsonc <<'EOF'
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
cp -f /etc/skel/.config/fastfetch/config.jsonc /root/.config/fastfetch/config.jsonc
```

Test cepat:

```bash
fastfetch --show-errors
```

**Opsional (biar fastfetch auto jalan saat buka terminal):**

```bash
grep -q 'fastfetch' /etc/skel/.bashrc || echo 'fastfetch || true' >> /etc/skel/.bashrc
grep -q 'fastfetch' /root/.bashrc     || echo 'fastfetch || true' >> /root/.bashrc
```

---

### B6) Wallpaper: metode “TOTAL WAR” (versi yang terbukti nembak target)

> Ini narget file backdrops yang sering dipakai Xubuntu, jadi wallpaper default benar-benar keganti.

```bash
cp -f /root/assets/wallpaper.png /tmp/my-wall.png
```

Target utama (Questing):

```bash
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

Timpa “xubuntu-wallpaper.png” (kadang symlink/target):

```bash
rm -f /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
```

Jaga-jaga file lain:

```bash
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-development.png || true
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-plucky.png || true
```

Verifikasi ukuran (harus “masuk akal”, mirip ukuran wallpaper kamu):

```bash
ls -lh /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

---

### B7) LightDM greeter background (login screen)

```bash
WALL="/usr/share/digiroom/wallpapers/wallpaper.png"
GREETER="/etc/lightdm/lightdm-gtk-greeter.conf"

touch "$GREETER"
grep -q '^\[greeter\]' "$GREETER" || printf "[greeter]\n" >> "$GREETER"

if grep -q '^background=' "$GREETER"; then
  sed -i "s|^background=.*|background=$WALL|" "$GREETER"
else
  sed -i "/^\[greeter\]/a background=$WALL" "$GREETER"
fi
```

Cek:

```bash
grep -n '^\[greeter\]\|^background=' /etc/lightdm/lightdm-gtk-greeter.conf
```

---

### B8) Branding OS (opsional buat UAS)

```bash
cat > /etc/os-release <<'EOF'
NAME="DigiRoomOS GNU/Linux"
PRETTY_NAME="DigiRoomOS GNU/Linux"
ID=digiroomos
ID_LIKE=ubuntu
VERSION="1.0 (Xubuntu base)"
VERSION_ID="1.0"
HOME_URL="https://example.local"
SUPPORT_URL="https://example.local"
BUG_REPORT_URL="https://example.local"
EOF

ln -sf /etc/os-release /usr/lib/os-release
cat /etc/os-release
```

---

## C. Tambahan aplikasi “Digital Classroom” (opsional tapi lengkap)

Kamu punya 2 jalur:
- **APT** (lebih simple, tapi versi bisa lebih lama)
- **Flatpak/Flathub** (biasanya versi lebih baru, cocok buat aplikasi kreatif & IDE)

### C1) Setup Flatpak + Flathub (di chroot)

Ikuti standar Flatpak untuk Ubuntu: citeturn9view0

```bash
apt-get update
apt-get install -y flatpak
```

Tambah remote Flathub:

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak remotes
```

---

### C2) Paket rekomendasi (via Flatpak)

Aplikasi coding (open-source):
- **VSCodium** (open-source build dari VS Code) citeturn28view0

Aplikasi multimedia / kreatif:
- **OBS Studio** (rekam layar + streaming) citeturn27view0
- **Kdenlive** (video editor) citeturn26view3
- **Shotcut** (video editor) citeturn26view4
- **GIMP** (image editor) citeturn28view1
- **Inkscape** (vector) citeturn26view1
- **Audacity** (audio editor) citeturn28view2

Browser open-source (opsional):
- **Chromium** citeturn28view5

Install semuanya (sekali jalan):

```bash
flatpak install -y flathub   com.vscodium.codium   com.obsproject.Studio   org.kde.kdenlive   org.shotcut.Shotcut   org.gimp.GIMP   org.inkscape.Inkscape   org.audacityteam.Audacity   org.chromium.Chromium
```

Cek daftar:

```bash
flatpak list
```

---

### C3) Paket rekomendasi (via APT)

Coding & STEM:

```bash
apt-get update
apt-get install -y   build-essential gcc g++ make cmake   python3 python3-pip python3-venv   git curl wget   geany   scratch   stellarium   gcompris-qt
```

Catatan:
- `scratch`, `stellarium`, `gcompris-qt` ada di repo Ubuntu (cukup oke buat “Digital Classroom”).

---

### C4) Install Zoom (proprietary)

Buka halaman resmi Zoom untuk Linux, download paket `.deb` 64-bit: citeturn20view0

Contoh alur install (di chroot):

```bash
cd /tmp
# Download manual dari web Zoom (recommended), atau pakai wget kalau kamu sudah punya link .deb yang valid
# wget -O zoom.deb "PASTE_LINK_DEB_ZOOM_DI_SINI"

# Install:
apt-get install -y ./zoom.deb || apt-get -f install -y
```

> Karena link Zoom kadang berubah, cara paling aman: **download .deb dari website Zoom**, lalu drag & drop ke Cubic Terminal (ke `/tmp/`) sebelum install.

---

### C5) Install Google Chrome (proprietary)

Google menyediakan download Chrome untuk Linux (pilih **64-bit .deb (Debian/Ubuntu)**): citeturn19view0turn15search0

Contoh alur install (di chroot):

```bash
cd /tmp
# Download manual dari halaman Chrome (recommended), atau pakai wget kalau kamu sudah pakai link direct .deb:
# wget -O chrome.deb "PASTE_LINK_DEB_CHROME_DI_SINI"

apt-get install -y ./chrome.deb || apt-get -f install -y
```

---

## D. GRUB background untuk boot ISO (dikerjakan di HOST, bukan di chroot)

> Ini dikerjakan di host kamu (Kali), pada folder project Cubic yang ada `custom-root/` dan `custom-disk/`.

Masuk folder project Cubic (contoh kamu `~/izin`):

```bash
cd ~/izin
ls
```

### D1) Copy wallpaper ke ISO boot folder

```bash
sudo cp -f custom-root/usr/share/digiroom/wallpapers/wallpaper.png custom-disk/boot/grub/wallpaper.png
ls -lh custom-disk/boot/grub/wallpaper.png
```

### D2) Edit `custom-disk/boot/grub/grub.cfg`

Buka file:

```bash
nano custom-disk/boot/grub/grub.cfg
```

Cari bagian:

```cfg
loadfont unicode
```

Lalu **setelah itu**, tambahkan blok ini:

```cfg
insmod all_video
insmod gfxterm
insmod gfxterm_background
insmod png
set gfxmode=auto
terminal_output gfxterm
background_image /boot/grub/wallpaper.png
```

Save (Ctrl+O) → Enter → keluar (Ctrl+X).

### D3) Edit `custom-disk/boot/grub/loopback.cfg`

```bash
nano custom-disk/boot/grub/loopback.cfg
```

Tambahkan blok yang sama **paling atas sebelum menuentry pertama**:

```cfg
insmod all_video
insmod gfxterm
insmod gfxterm_background
insmod png
set gfxmode=auto
terminal_output gfxterm
background_image /boot/grub/wallpaper.png
```

Save.

### D4) Verifikasi cepat

```bash
grep -n "background_image" custom-disk/boot/grub/grub.cfg custom-disk/boot/grub/loopback.cfg
```

Harus muncul 2 baris.

---

## E. Build ISO

Di Cubic:
1) Pastikan semua step chroot sudah beres
2) Klik **Generate**
3) Test ISO di VirtualBox/VMWare

---

## F. Checklist testing (wajib buat bukti UAS)

1) Boot ISO → **GRUB menu** muncul + background wallpaper ✅  
2) Masuk Live Desktop → wallpaper sudah sesuai ✅  
3) Login screen (LightDM) background sudah sesuai ✅  
4) Buka terminal → `fastfetch` aman ✅  
5) Pastikan aplikasi yang kamu pilih muncul (VLC, Scratch, Stellarium, dll) ✅

---

## Troubleshooting singkat

### Wallpaper masih default
- Cek file target bener ada:
  ```bash
  ls -lh /usr/share/xfce4/backdrops/xubuntu-questing.png
  ```
- Pastikan kamu memang build ISO setelah perubahan.

### GRUB background tidak muncul
- Pastikan file ada di:
  ```bash
  ls -lh custom-disk/boot/grub/wallpaper.png
  ```
- Pastikan `grub.cfg` **dan** `loopback.cfg` sama-sama punya `background_image`.

---

Selesai ✅
