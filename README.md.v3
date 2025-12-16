# DigiRoomOS GNU/Linux — Remaster Xubuntu via Cubic (Tanpa Flatpak, Tanpa Plymouth)
Panduan ini memecah langkah jadi **2 bagian utama**:
- **A. Chroot (Terminal di Cubic)** → isi sistem (app, wallpaper, fastfetch, login)
- **B. ISO Boot (custom-disk di host Kali/Linux)** → **GRUB background** saat boot ISO

> Catatan: Di Terminal Cubic biasanya kamu sudah **root**, jadi **tanpa `sudo`**.

---

## 0) Struktur aset (wajib)
Pastikan di environment chroot (Terminal Cubic) ada file berikut:

- `/root/assets/wallpaper.png`
- `/root/assets/pnj.png`
- `/root/assets/logo.png` *(opsional buat branding, tapi **tidak dipakai untuk Plymouth** di panduan ini)*

Cek:

```bash
ls -lh /root/assets/
```

---

# A) BAGIAN CHROOT — Jalankan di Terminal Cubic

## 1) Copy aset ke lokasi DigiRoomOS
```bash
install -d /usr/share/digiroom/wallpapers
install -d /usr/share/digiroom/branding

cp -f /root/assets/wallpaper.png /usr/share/digiroom/wallpapers/wallpaper.png
cp -f /root/assets/pnj.png       /usr/share/digiroom/branding/pnj.png
cp -f /root/assets/logo.png      /usr/share/digiroom/branding/logo.png
```

---

## 2) Update repo & install tools wajib
```bash
apt-get update
apt-get install -y fastfetch chafa imagemagick papirus-icon-theme vlc
```

---

## 3) Hapus bloat (aman)
> Kalau ada paket yang tidak ada, `apt` akan kasih error. Itu normal—hapus dari list.

```bash
apt-get purge -y \
  gnome-mines gnome-sudoku sgt-puzzles sgt-launcher \
  transmission-gtk transmission-common \
  hexchat hexchat-common hexchat-plugins \
  rhythmbox rhythmbox-data rhythmbox-plugins \
  || true

apt-get autoremove -y --purge
apt-get clean
```

---

## 4) Fastfetch: pnj.png → ASCII txt (chafa)
```bash
chafa -c full -s 40x20 /usr/share/digiroom/branding/pnj.png > /usr/share/digiroom/branding/pnj.txt
```

Cek hasilnya:

```bash
head -n 25 /usr/share/digiroom/branding/pnj.txt
```

---

## 5) Fastfetch config (skel + root)
```bash
install -d /etc/skel/.config/fastfetch
install -d /root/.config/fastfetch
```

Buat config:

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

Copy untuk root:

```bash
cp -f /etc/skel/.config/fastfetch/config.jsonc /root/.config/fastfetch/config.jsonc
```

Tes:

```bash
fastfetch --show-errors
```

---

## 6) Wallpaper: metode “TOTAL WAR” (yang terbukti work)
Ini menarget file wallpaper yang sering dipakai Xubuntu.

```bash
cp -f /root/assets/wallpaper.png /tmp/my-wall.png
```

Target utama:

```bash
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

Timpa file “xubuntu-wallpaper.png” (kadang dipakai/symlink):

```bash
rm -f /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
```

Jaga-jaga:

```bash
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-development.png || true
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-plucky.png || true
```

Konfirmasi ukuran:

```bash
ls -lh /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

---

## 7) LightDM (login screen) pakai wallpaper DigiRoomOS
```bash
WALL="/usr/share/digiroom/wallpapers/wallpaper.png"
GREETER="/etc/lightdm/lightdm-gtk-greeter.conf"

touch "$GREETER"
```

Pastikan ada section `[greeter]`:

```bash
grep -q "^\[greeter\]" "$GREETER" || printf "\n[greeter]\n" >> "$GREETER"
```

Set background:

```bash
if grep -q "^background=" "$GREETER"; then
  sed -i "s|^background=.*|background=$WALL|" "$GREETER"
else
  sed -i "/^\[greeter\]/a background=$WALL" "$GREETER"
fi
```

Cek:

```bash
grep -nE "^\[greeter\]|^background=" "$GREETER"
```

---

## 8) Branding OS (opsional)
```bash
cat > /etc/os-release <<'EOF'
NAME="DigiRoomOS GNU/Linux"
PRETTY_NAME="DigiRoomOS GNU/Linux"
ID=digiroomos
ID_LIKE=ubuntu
VERSION="1.0 (Xubuntu base)"
VERSION_ID="1.0"
EOF

ln -sf /etc/os-release /usr/lib/os-release
```

---

# B) BAGIAN GRUB ISO — Jalankan di HOST (Kali/Linux), folder project Cubic

> Contoh lokasi project kamu: `~/izin`

## 9) Copy wallpaper ke custom-disk
```bash
cd ~/izin
sudo cp -f custom-root/usr/share/digiroom/wallpapers/wallpaper.png custom-disk/boot/grub/wallpaper.png
ls -lh custom-disk/boot/grub/wallpaper.png
```

---

## 10) Edit GRUB config (boot menu ISO)
Edit `custom-disk/boot/grub/grub.cfg` dan `custom-disk/boot/grub/loopback.cfg`.

Tambahkan blok ini **setelah `loadfont unicode`** di `grub.cfg`, dan **di paling atas** di `loopback.cfg`:

```cfg
insmod all_video
insmod gfxterm
insmod gfxterm_background
insmod png
set gfxmode=auto
terminal_output gfxterm
background_image /boot/grub/wallpaper.png
```

Cek cepat:

```bash
grep -n "background_image" custom-disk/boot/grub/grub.cfg custom-disk/boot/grub/loopback.cfg
```

---

# C) Paket “Digital Classroom” (APT)
> Semua ini via `apt-get install`. **Tanpa Flatpak**.  
> Catatan: beberapa paket (misal `chromium`) di Ubuntu kadang jadi paket transisi ke Snap; kalau kamu mau “murni deb”, skip bagian itu.

## 11) Produktivitas & dokumen
```bash
apt-get update
apt-get install -y libreoffice
apt-get install -y okular evince
apt-get install -y hunspell-id
```

## 12) Coding & Dev tools (open-source)
```bash
apt-get install -y build-essential git curl wget
apt-get install -y python3 python3-pip
apt-get install -y default-jdk
apt-get install -y nodejs npm
apt-get install -y geany geany-plugins
apt-get install -y codeblocks
apt-get install -y vim nano
```

Opsional (kalau ada di repo):

```bash
apt-get install -y kate || true
apt-get install -y meld || true
```

## 13) Multimedia & kreatif (open-source)
```bash
apt-get install -y gimp inkscape
apt-get install -y audacity
apt-get install -y kdenlive
apt-get install -y blender
apt-get install -y ffmpeg
```

## 14) Rekam layar & presentasi
```bash
apt-get install -y obs-studio
```

Opsional:

```bash
apt-get install -y flameshot || true
```

## 15) Edukasi / Digital Classroom
```bash
apt-get install -y scratch
apt-get install -y stellarium
```

Opsional:

```bash
apt-get install -y gcompris-qt || true
apt-get install -y tuxpaint tuxmath || true
```

## 16) Browser via APT (opsional)
```bash
apt-get install -y chromium || true
```

---

# D) (OPSIONAL) Install app vendor via wget + .deb (tanpa Flatpak)
> Ini **bukan** dari repo Ubuntu default, tapi masih “install pakai apt” dari file `.deb`.

## 17) Google Chrome (stable)
```bash
wget -O /tmp/chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
apt-get install -y /tmp/chrome.deb
```

Cek:

```bash
google-chrome --version || true
```

## 18) Zoom Desktop (Linux .deb)
```bash
wget -O /tmp/zoom.deb https://zoom.us/client/latest/zoom_amd64.deb
apt-get install -y /tmp/zoom.deb
```

Cek:

```bash
zoom --version || true
```

---

# E) Shortcut Desktop per link (untuk user baru)
Kita taruh shortcut di `/etc/skel/Desktop/` supaya:
- **Live session** (biasanya) ikut kebawa
- User baru setelah install juga otomatis punya shortcut

## 19) Buat folder Desktop di skel
```bash
install -d /etc/skel/Desktop
```

> Pilih salah satu: pakai **Firefox** (default) atau **Chrome** (kalau kamu install Chrome).

---

## 20) Shortcut Firefox (1 link = 1 shortcut)

### 20.1 Academia PNJ
```bash
cat > /etc/skel/Desktop/academia-pnj.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Academia PNJ
Comment=Buka Academia PNJ
Exec=firefox --new-window "https://academia.pnj.ac.id/"
Icon=firefox
Terminal=false
Categories=Network;WebBrowser;
EOF
```

### 20.2 Website PNJ
```bash
cat > /etc/skel/Desktop/pnj-website.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Website PNJ
Comment=Buka situs resmi PNJ
Exec=firefox --new-window "https://pnj.ac.id/"
Icon=firefox
Terminal=false
Categories=Network;WebBrowser;
EOF
```

### 20.3 Google Scholar
```bash
cat > /etc/skel/Desktop/google-scholar.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Google Scholar
Comment=Buka Google Scholar
Exec=firefox --new-window "https://scholar.google.com/"
Icon=firefox
Terminal=false
Categories=Network;WebBrowser;
EOF
```

### 20.4 Google Meet
```bash
cat > /etc/skel/Desktop/google-meet.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Google Meet
Comment=Buka Google Meet
Exec=firefox --new-window "https://meet.google.com/landing"
Icon=firefox
Terminal=false
Categories=Network;WebBrowser;
EOF
```

Biar bisa diklik:

```bash
chmod +x /etc/skel/Desktop/*.desktop
```

---

## 21) Shortcut Chrome (kalau Chrome terpasang)
Kalau kamu install Chrome, buat versi Chrome juga (opsional). Contoh Academia:

```bash
cat > /etc/skel/Desktop/academia-pnj-chrome.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Academia PNJ (Chrome)
Comment=Buka Academia PNJ via Google Chrome
Exec=google-chrome --new-window "https://academia.pnj.ac.id/"
Icon=google-chrome
Terminal=false
Categories=Network;WebBrowser;
EOF

chmod +x /etc/skel/Desktop/academia-pnj-chrome.desktop
```

> Ulangi pola yang sama untuk link lain jika mau semuanya versi Chrome.

---

# F) Build ISO & Test
1. Kembali ke Cubic → **Generate** ISO
2. Test di VM (VirtualBox):
   - GRUB background muncul
   - Live desktop wallpaper berubah
   - Login screen background berubah
   - `fastfetch` normal

---

## Troubleshooting cepat
- Jika `.desktop` muncul tapi tidak bisa diklik: pastikan `chmod +x` sudah, lalu di desktop klik kanan → **Allow Launching**.
- Jika `apt` bilang “Unable to locate package …”: pastikan perintah dipaste **per baris**, jangan ketempel jadi `hunspell-idsudo` dsb.
