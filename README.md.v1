# DigiRoomOS GNU/Linux (Xubuntu base) — Remaster via Cubic

Dokumen ini adalah **panduan langkah-per-langkah** (bukan script sekali jalan) untuk meremaster ISO Xubuntu menjadi **DigiRoomOS (Digital Classroom)** menggunakan **Cubic**.

Target yang dicapai:
- Hapus bloatware aman
- Wallpaper **desktop (XFCE)** + **login screen (LightDM)** jadi wallpaper kamu
- GRUB background **boot ISO** (menu “Try or Install …”) jadi wallpaper kamu
- Fastfetch pakai **logo ASCII** (pnj.png → txt via chafa)
- **Plymouth: dibiarkan default** (tidak disentuh)

> Catatan: Ada 2 “dunia” dalam project Cubic:
> - **custom-root (chroot)**: isi sistem Linux yang nanti dipasang (packages, config, wallpaper, fastfetch).
> - **custom-disk**: struktur boot ISO (GRUB menu, loopback.cfg, dll).  
>   **GRUB background ISO harus edit di custom-disk.**

---

## 0) Persiapan aset

Taruh 3 file aset kamu di dalam environment Cubic (di **Terminal Cubic / chroot**), lokasinya:

- `/root/assets/wallpaper.png`
- `/root/assets/pnj.png`
- `/root/assets/logo.png` *(buat branding saja; tidak dipakai plymouth)*

Cek:

```bash
ls -lh /root/assets/
file /root/assets/wallpaper.png /root/assets/pnj.png /root/assets/logo.png
```

---

## 1) Copy aset ke lokasi final (di Terminal Cubic / chroot)

Buat folder brand DigiRoomOS + copy aset:

```bash
sudo install -d /usr/share/digiroom/wallpapers
sudo install -d /usr/share/digiroom/branding

sudo cp -f /root/assets/wallpaper.png /usr/share/digiroom/wallpapers/wallpaper.png
sudo cp -f /root/assets/pnj.png       /usr/share/digiroom/branding/pnj.png
sudo cp -f /root/assets/logo.png      /usr/share/digiroom/branding/logo.png
```

Verifikasi:

```bash
ls -lh /usr/share/digiroom/wallpapers/wallpaper.png
ls -lh /usr/share/digiroom/branding/pnj.png /usr/share/digiroom/branding/logo.png
```

---

## 2) Update & install tools dasar (di Terminal Cubic / chroot)

Update index repo:

```bash
sudo apt-get update
```

Install tools yang kepake untuk branding & kebutuhan dasar:

```bash
sudo apt-get install -y fastfetch chafa imagemagick papirus-icon-theme vlc
```

---

## 3) Hapus bloatware yang aman (di Terminal Cubic / chroot)

Purge “games/torrent/chat/music” yang umum dan aman:

```bash
sudo apt-get purge -y \
  gnome-mines gnome-sudoku sgt-puzzles sgt-launcher \
  transmission-gtk transmission-common \
  hexchat hexchat-common hexchat-plugins \
  rhythmbox rhythmbox-data rhythmbox-plugins \
  || true
```

Rapikan dependency:

```bash
sudo apt-get autoremove -y --purge
sudo apt-get clean
```

> Jika kamu mau **hapus LibreOffice/Thunderbird**, lakukan terpisah (opsional):
```bash
sudo apt-get purge -y thunderbird "libreoffice*" || true
sudo apt-get autoremove -y --purge
sudo apt-get clean
```

---

## 4) Fastfetch — buat logo ASCII (pnj.png → pnj.txt)

Convert pnj.png jadi teks (ASCII) agar fastfetch stabil:

```bash
sudo chafa -c full -s 40x20 /usr/share/digiroom/branding/pnj.png | sudo tee /usr/share/digiroom/branding/pnj.txt >/dev/null
```

Cek hasil:

```bash
head -n 20 /usr/share/digiroom/branding/pnj.txt
```

---

## 5) Fastfetch — set config untuk user baru & root

Buat folder config:

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

Copy untuk root:

```bash
sudo cp -f /etc/skel/.config/fastfetch/config.jsonc /root/.config/fastfetch/config.jsonc
```

Test di chroot:

```bash
fastfetch --show-errors
```

---

## 6) Wallpaper — metode “TOTAL WAR” (XFCE backdrop)

Metode ini menimpa file wallpaper bawaan Xubuntu yang paling sering dipakai.

Siapkan “peluru”:

```bash
sudo cp -f /root/assets/wallpaper.png /tmp/my-wall.png
```

Tembak target utama:

```bash
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

Tembak “xubuntu-wallpaper.png” (kadang symlink / kadang dipanggil langsung):

```bash
sudo rm -f /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-wallpaper.png || true
```

Jaga-jaga file lain:

```bash
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-development.png || true
sudo cp -f /tmp/my-wall.png /usr/share/xfce4/backdrops/xubuntu-plucky.png || true
```

Verifikasi ukuran:

```bash
ls -lh /usr/share/xfce4/backdrops/xubuntu-questing.png || true
```

---

## 7) Wallpaper Login Screen — LightDM GTK Greeter

Set background LightDM ke wallpaper DigiRoomOS:

```bash
WALL="/usr/share/digiroom/wallpapers/wallpaper.png"
GREETER="/etc/lightdm/lightdm-gtk-greeter.conf"

sudo touch "$GREETER"
```

Pastikan ada section `[greeter]`:

```bash
if ! grep -q "^\[greeter\]" "$GREETER"; then
  echo "[greeter]" | sudo tee -a "$GREETER" >/dev/null
fi
```

Set/replace `background=`:

```bash
if grep -q "^background=" "$GREETER"; then
  sudo sed -i "s|^background=.*|background=$WALL|" "$GREETER"
else
  sudo sed -i "/^\[greeter\]/a background=$WALL" "$GREETER"
fi
```

Cek:

```bash
grep -nE "^\[greeter\]|^background=" /etc/lightdm/lightdm-gtk-greeter.conf
```

---

## 8) Branding OS (opsional)

Ubah `/etc/os-release`:

```bash
sudo tee /etc/os-release >/dev/null <<'EOF'
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
```

Samakan `os-release` system:

```bash
sudo ln -sf /etc/os-release /usr/lib/os-release
```

---

## 9) Paket “Digital Classroom” (APT-only, tanpa Flatpak)

Di bawah ini **semua via `apt`**.  
Catatan penting: **Zoom Desktop dan Google Chrome** biasanya **tidak tersedia** di repo Ubuntu default, jadi untuk *benar-benar apt-only* kita pakai alternatif:
- Browser: **Firefox bawaan** + **Chromium** (kalau tersedia)
- Video conference: pakai **web app di browser** (Zoom Web / Google Meet / Microsoft Teams Web) atau **Jitsi via web**

### 9.1 Produktivitas & dokumen

```bash
sudo apt-get install -y libreoffice
sudo apt-get install -y okular evince
sudo apt-get install -y hunspell-id
```

### 9.2 Coding & Dev tools (open-source)

```bash
sudo apt-get install -y build-essential git curl wget
sudo apt-get install -y python3 python3-pip
sudo apt-get install -y default-jdk
sudo apt-get install -y nodejs npm
sudo apt-get install -y geany geany-plugins
sudo apt-get install -y codeblocks
sudo apt-get install -y vim nano
```

*(Opsional editor lain, kalau ada di repo kamu):*
```bash
sudo apt-get install -y kate || true
sudo apt-get install -y meld || true
```

### 9.3 Multimedia & kreatif (open-source)

```bash
sudo apt-get install -y gimp inkscape
sudo apt-get install -y audacity
sudo apt-get install -y kdenlive
sudo apt-get install -y blender
sudo apt-get install -y ffmpeg
```

### 9.4 Rekam layar & presentasi

```bash
sudo apt-get install -y obs-studio
```

*(Opsional screenshot/annotate):*
```bash
sudo apt-get install -y flameshot || true
```

### 9.5 Edukasi / Digital Classroom

```bash
sudo apt-get install -y scratch
sudo apt-get install -y stellarium
```

*(Opsional edukasi tambahan, jalankan kalau tersedia):*
```bash
sudo apt-get install -y gcompris-qt || true
sudo apt-get install -y tuxpaint tuxmath || true
```

### 9.6 Browser (APT)

Coba install Chromium (kalau tersedia di repo kamu):

```bash
sudo apt-get install -y chromium || true
```

> Kalau `chromium` tidak tersedia, cukup pakai **Firefox bawaan** untuk akses web classroom (Google Classroom, Meet, Zoom web, LMS kampus).

---

## 10) GRUB background untuk BOOT ISO (di HOST, bukan di chroot)

Bagian ini dilakukan di **host OS kamu** (mis. Kali Linux) pada folder project Cubic (`~/izin`).

### 10.1 Copy wallpaper ke custom-disk

```bash
cd ~/izin
sudo cp -f custom-root/usr/share/digiroom/wallpapers/wallpaper.png custom-disk/boot/grub/wallpaper.png
ls -lh custom-disk/boot/grub/wallpaper.png
```

### 10.2 Edit `custom-disk/boot/grub/grub.cfg`

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

### 10.3 Edit `custom-disk/boot/grub/loopback.cfg`

Buka:

```bash
nano custom-disk/boot/grub/loopback.cfg
```

Tambahkan blok yang sama di **paling atas** sebelum `menuentry`:

```cfg
insmod all_video
insmod gfxterm
insmod gfxterm_background
insmod png
set gfxmode=auto
terminal_output gfxterm
background_image /boot/grub/wallpaper.png
```

### 10.4 Verifikasi cepat

```bash
grep -n "background_image" custom-disk/boot/grub/grub.cfg custom-disk/boot/grub/loopback.cfg
```

---

## 11) Build ISO & Testing

Di Cubic:
1. Pastikan semua perubahan di Terminal (chroot) sudah selesai
2. Klik **Generate** untuk build ISO
3. Test di VirtualBox / QEMU:
   - GRUB menu ISO muncul background wallpaper ✅
   - Live Desktop wallpaper berubah ✅
   - Login screen berubah ✅
   - `fastfetch --show-errors` aman ✅

---

## Troubleshooting cepat

- Wallpaper desktop masih default:
  - Pastikan file yang ditembak bener-bener ada:
    ```bash
    ls -lh /usr/share/xfce4/backdrops/xubuntu-questing.png
    ```
- GRUB background tidak tampil:
  - Pastikan file ada di `custom-disk/boot/grub/wallpaper.png`
  - Pastikan blok `background_image` ada **di grub.cfg dan loopback.cfg**
- Fastfetch error:
  - Cek config:
    ```bash
    cat /etc/skel/.config/fastfetch/config.jsonc
    ```
  - Cek file txt:
    ```bash
    ls -lh /usr/share/digiroom/branding/pnj.txt
    ```
