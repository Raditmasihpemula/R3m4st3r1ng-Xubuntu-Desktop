# DigiRoomOS (Xubuntu Remaster via Cubic) — Step-by-step (APT only, no Plymouth)

Target: bikin ISO Xubuntu jadi **DigiRoomOS (Digital Classroom)** dengan:
- Hapus bloat aman
- Wallpaper **desktop + login (LightDM)** jadi wallpaper kamu
- GRUB background (menu boot ISO) jadi wallpaper kamu
- Fastfetch logo ASCII dari `pnj.png` (via `chafa` → `.txt`)
- **Tanpa** utak-atik Plymouth (biarin default)

> Catatan penting: banyak error “package tidak ditemukan” itu **bukan karena paketnya nggak ada**, tapi karena **copy‑paste kamu nyatuin beberapa command jadi satu baris**, sehingga apt mengira ada paket bernama `ffmpegsudo`, `stellariumsudo`, dll.

---

## A. Persiapan Aset (di Cubic project)

Pastikan kamu sudah taruh aset ini di **custom-root** (Cubic → bagian file manager/chroot):
- `/root/assets/wallpaper.png`
- `/root/assets/pnj.png`
- `/root/assets/logo.png`

Cek:
```bash
ls -lh /root/assets/
```

---

## B. Customize di Cubic Terminal (chroot / custom-root)

> Jalankan di **Terminal Cubic** (yang masuk chroot). Biasanya prompt-nya `root@cubic:~#`.

### B1) Buat folder branding & copy aset
```bash
install -d /usr/share/digiroom/wallpapers
install -d /usr/share/digiroom/branding

cp -f /root/assets/wallpaper.png /usr/share/digiroom/wallpapers/wallpaper.png
cp -f /root/assets/pnj.png       /usr/share/digiroom/branding/pnj.png
cp -f /root/assets/logo.png      /usr/share/digiroom/branding/logo.png
```

### B2) Update repo & install tools inti (untuk branding + utilitas)
```bash
apt-get update
apt-get install -y fastfetch chafa imagemagick papirus-icon-theme
```

### B3) Hapus bloat “aman” (boleh kamu tambah/kurangin)
> Jangan purge LibreOffice/Thunderbird kalau kamu mau install buat kelas.
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

### B4) Fastfetch: convert `pnj.png` → ASCII txt
```bash
chafa -c full -s 40x20 /usr/share/digiroom/branding/pnj.png > /usr/share/digiroom/branding/pnj.txt
```

Cek hasilnya:
```bash
head -n 15 /usr/share/digiroom/branding/pnj.txt
```

### B5) Fastfetch config untuk user baru + root
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

Copy ke root:
```bash
cp -f /etc/skel/.config/fastfetch/config.jsonc /root/.config/fastfetch/config.jsonc
```

Test:
```bash
fastfetch --show-errors
```

---

## C. Wallpaper Desktop (metode “TOTAL WAR” yang terbukti keubah)

Metode ini **menimpa file backdrop Xubuntu** yang biasanya dipakai default.

### C1) Siapkan “peluru”
```bash
cp -f /root/assets/wallpaper.png /tmp/my-wall.png
```

### C2) Timpa target utama (Questing)
```bash
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-questing.png
```

### C3) Timpa juga file `xubuntu-wallpaper.png` (sering jadi symlink/target)
```bash
rm -f /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-wallpaper.png
```

### C4) Jaga-jaga mode lain
```bash
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-development.png || true
cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-plucky.png      || true
```

### C5) Konfirmasi ukuran file target
```bash
ls -lh /usr/share/xfce4/backdrops/xubuntu-questing.png
```

---

## D. Wallpaper Login Screen (LightDM GTK Greeter)

```bash
WALL="/usr/share/digiroom/wallpapers/wallpaper.png"
GREETER="/etc/lightdm/lightdm-gtk-greeter.conf"

touch "$GREETER"

grep -q "^\[greeter\]" "$GREETER" || printf "\n[greeter]\n" >> "$GREETER"

if grep -q "^background=" "$GREETER"; then
  sed -i "s|^background=.*|background=$WALL|" "$GREETER"
else
  sed -i "/^\[greeter\]/a background=$WALL" "$GREETER"
fi
```

Cek:
```bash
grep -nE "^\[greeter\]|^background=" /etc/lightdm/lightdm-gtk-greeter.conf
```

---

## E. Branding OS (opsional, buat identitas di UAS)

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
```

---

## F. Install Aplikasi “Digital Classroom” (APT only)

> Semua di bawah ini **harus dipisah per baris**. Jangan sampai `... || true sudo apt-get ...` nyatu.

### F1) Office & PDF
```bash
apt-get install -y libreoffice
apt-get install -y okular evince
apt-get install -y hunspell-id || true
```

### F2) Coding / Development (open-source)
```bash
apt-get install -y build-essential git curl wget
apt-get install -y python3 python3-pip
apt-get install -y default-jdk
apt-get install -y nodejs npm
apt-get install -y geany geany-plugins
apt-get install -y codeblocks
apt-get install -y vim nano kate
apt-get install -y meld
apt-get install -y thonny || true
```

### F3) Multimedia (grafis, audio, video)
```bash
apt-get install -y vlc
apt-get install -y gimp inkscape
apt-get install -y audacity
apt-get install -y kdenlive
apt-get install -y blender
apt-get install -y ffmpeg
```

### F4) Rekam layar + screenshot (buat demo UAS)
```bash
apt-get install -y obs-studio
apt-get install -y flameshot
```

### F5) Edu apps
```bash
apt-get install -y scratch
apt-get install -y stellarium
apt-get install -y gcompris-qt || true
apt-get install -y tuxpaint tuxmath tuxtyping || true
```

### F6) Browser
```bash
apt-get install -y firefox || true
apt-get install -y chromium || true
```

> **Google Chrome & Zoom**: biasanya **tidak ada di repo Ubuntu resmi** (butuh repo vendor / download `.deb`).
> Karena kamu minta **APT bawaan saja**, dua itu **nggak aku cantumin** di langkah utama.

---

## G. GRUB Background untuk Boot Menu ISO (di host / custom-disk)

> Ini dikerjakan di **host (Kali/Ubuntu host)** pada folder project Cubic kamu (misal `~/izin`), **bukan** di chroot.

### G1) Copy wallpaper ke folder GRUB ISO
```bash
cd ~/izin
sudo cp -f custom-root/usr/share/digiroom/wallpapers/wallpaper.png custom-disk/boot/grub/wallpaper.png
ls -lh custom-disk/boot/grub/wallpaper.png
```

### G2) Edit `custom-disk/boot/grub/grub.cfg`
Buka:
```bash
nano custom-disk/boot/grub/grub.cfg
```

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

### G3) Edit `custom-disk/boot/grub/loopback.cfg`
Buka:
```bash
nano custom-disk/boot/grub/loopback.cfg
```

Tambahkan blok yang sama **di paling atas** (sebelum `menuentry` pertama):
```cfg
insmod all_video
insmod gfxterm
insmod gfxterm_background
insmod png
set gfxmode=auto
terminal_output gfxterm
background_image /boot/grub/wallpaper.png
```

### G4) Verifikasi cepat
```bash
grep -n "background_image" custom-disk/boot/grub/grub.cfg custom-disk/boot/grub/loopback.cfg
```

---

## H. Build ISO & Testing

1. Balik ke Cubic → **Generate** ISO.
2. Test di VirtualBox:
   - GRUB menu ISO ada background ✅
   - Desktop wallpaper sudah berubah ✅
   - Login screen background berubah ✅
   - `fastfetch` tidak error ✅

---

## I. “Package tidak ditemukan” yang kamu alami (akar masalah + perbaikan)

Dari log kamu, apt sempat cari paket ini:
- `hunspell-idsudo`
- `nanosudo`
- `ffmpegsudo`
- `obs-studiosudo`
- `stellariumsudo`
- bahkan `apt-get` dan `install`

Itu **bukan nama paket asli**. Itu terjadi karena beberapa command kamu **ketempel jadi satu baris**.

✅ Nama paket yang bener:
- `hunspell-id`
- `nano`
- `ffmpeg`
- `obs-studio`
- `stellarium`

Tips: kalau mau cek paket ada/enggak sebelum install:
```bash
apt-cache policy obs-studio
apt-cache policy stellarium
apt-cache policy ffmpeg
```
