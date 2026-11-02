# Debian + GNOME setup

Setup for my main workstation (`derek-workstation`). Plain Debian stable with stock GNOME.

- Configs live in [dots](https://github.com/Dechrissen/dots) (bare repo in `$HOME`)
- Files referenced here are in `debian/files/`

---

## 1. Before installing

- Back up `~` and `/etc` to be safe
- Push any unpushed work: `dots status`, and `git status` in each repo under `~/dev`
- Make the USB: download the **netinst amd64** ISO + `SHA512SUMS` from debian.org

```bash
sha512sum -c SHA512SUMS --ignore-missing
lsblk -o NAME,SIZE,TRAN,MODEL          # find the USB (TRAN = usb)
sudo umount /dev/sdX?*
sudo dd if=debian-13.*-amd64-netinst.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

## 2. Installer choices

| Screen | Choice |
|---|---|
| Hostname | `derek-workstation` |
| Root password | **leave blank** (disables root, gives `derek` sudo) |
| User | `derek` |
| Partitioning | Guided – use entire disk and set up **encrypted LVM** → all files in one partition |
| Random-data erase | optional (can click Cancel) |
| Software | Debian desktop environment, GNOME, SSH server, standard system utilities |

## 3. System basics

**Repos**: add `contrib non-free` to the components in `/etc/apt/sources.list.d/debian.sources` (or after `main` on the `deb` lines in `/etc/apt/sources.list`):

```
Components: main contrib non-free non-free-firmware
```

Then update (this is also the regular update command, run it ~weekly):

```bash
sudo apt update && sudo apt full-upgrade
```

Every ~month, clean up old kernels and leftover dependencies:

```bash
sudo apt autoremove
```

Check and make sure Settings → System → About → Graphics shows the real GPU (not `llvmpipe`).

**Flatpak**:

```bash
sudo apt install flatpak gnome-software-plugin-flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

**Firewall** (SSH only from the LAN):

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.8.0/24 to any port 22 proto tcp
sudo ufw enable
```

**Security updates** (no auto-reboot on a desktop):

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades   # answer Yes
```

Reboot.

**SSH keys**: restore this machine's keys from backup into `~/.ssh`, then:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
chmod 644 ~/.ssh/*.pub
ssh -T git@github.com
```

## 4. GNOME configuration (manual)

```bash
sudo apt install gnome-shell-extension-appindicator gnome-shell-extensions gnome-shell-extension-prefs gnome-tweaks
```

Log out and back in, then in the **Extensions** app enable:

- **AppIndicator** (listed as "Ubuntu AppIndicators") for tray icons
- **System Monitor** for CPU / RAM / network in the top bar

**Keyboard shortcuts** (Settings → Keyboard → Custom Shortcuts):

1. `Super+Enter`: terminal (command = whichever terminal you want)
2. `Super+S`: screenshot
3. `Super+Q`: close window

**Change the way workspaces work, similar to Hyprland (`Super+[1-9]`)**:
- Settings -> Multitasking -> Workspaces -> Fixed number
- Then run:
```bash
for i in {1..9}; do
  gsettings set org.gnome.shell.keybindings switch-to-application-$i "[]"
done

for i in {1..9}; do
  gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-$i "['<Super>$i']"
  gsettings set org.gnome.desktop.wm.keybindings move-to-workspace-$i "['<Super><Shift>$i']"
done
```

**Other things**:
- Night Light
- Screen timeout and auto-suspend
- Time and date format for top bar

## 5. Packages

**From Debian**:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install \
  keepassxc gimp \
  git curl wget unzip build-essential \
  vim htop fastfetch ripgrep fd-find rclone newsboat dict \
  python3-pip python3-venv \
  tesseract-ocr \
  lm-sensors cifs-utils \
  yubikey-manager pcscd \
  libfuse2t64 \
  fonts-font-awesome \
  steam-installer mesa-vulkan-drivers mesa-vulkan-drivers:i386 libgl1-mesa-dri:i386 libglx-mesa0:i386

sudo sensors-detect --auto
ykman info                     # with YubiKey plugged in
```

Python: one venv per project (`python3 -m venv .venv`) — system-wide `pip install` is blocked by design.

**Flatpak**:

```bash
flatpak install flathub org.godotengine.Godot
flatpak install flathub com.yubico.yubioath
flatpak install flathub com.discordapp.Discord
```

**VSCodium** (vscodium.com):

```bash
wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg \
    | gpg --dearmor \
    | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg
echo -e 'Types: deb\nURIs: https://download.vscodium.com/debs\nSuites: vscodium\nComponents: main\nArchitectures: amd64 arm64\nSigned-by: /usr/share/keyrings/vscodium-archive-keyring.gpg' \
| sudo tee /etc/apt/sources.list.d/vscodium.sources
sudo apt update && sudo apt install codium
```

**Signal** (signal.org/download):

```bash
curl https://updates.signal.org/desktop/apt/keys.asc | gpg --dearmor > signal-desktop-keyring.gpg
cat signal-desktop-keyring.gpg | sudo tee /usr/share/keyrings/signal-desktop-keyring.gpg > /dev/null
curl -o signal-desktop.sources https://updates.signal.org/static/desktop/apt/signal-desktop.sources
cat signal-desktop.sources | sudo tee /etc/apt/sources.list.d/signal-desktop.sources > /dev/null
sudo apt update && sudo apt install signal-desktop
rm signal-desktop-keyring.gpg signal-desktop.sources
```

**Dropbox**: download the **Ubuntu 22.10 or higher** `.deb` from https://dropbox.com/install-linux, then:

```bash
sudo apt install ~/Downloads/dropbox_*.deb
```

Launch Dropbox from the app search and sign in. (If `apt update` ever errors on the Dropbox repo, delete `/etc/apt/sources.list.d/dropbox.list` — the client updates itself.)

## 6. Dev tools

**nvm + Node**: copy the current install command from https://github.com/nvm-sh/nvm (the `curl` one-liner), reopen the terminal, then:

```bash
nvm install node
nvm alias default node
npm install -g nodemon
```

**Claude Code** (native installer, auto-updates):

```bash
curl -fsSL https://claude.ai/install.sh | bash
source ~/.profile          # ~/.local/bin isn't on PATH until next login
claude --version
```

**Make local `~/dev` directory**

```bash
mkdir -p ~/dev
```

Then clone personal project repos.

## 7. Hardware wallets and fonts

**Trezor Suite** (AppImage from https://trezor.io/trezor-suite):

```bash
mkdir -p ~/Applications
mv ~/Downloads/Trezor-Suite-*.AppImage ~/Applications/Trezor-Suite.AppImage
chmod +x ~/Applications/Trezor-Suite.AppImage
sudo curl -fsSL https://data.trezor.io/udev/51-trezor.rules -o /etc/udev/rules.d/51-trezor.rules
sudo udevadm control --reload-rules && sudo udevadm trigger
cp files/trezor-suite.desktop ~/.local/share/applications/
```

The fixed filename (no version) lets the in-app updater keep the launcher working.

**Nerd Fonts**:

```bash
mkdir -p ~/.local/share/fonts
cd ~/Downloads
for f in 0xProto JetBrainsMono RobotoMono; do
  curl -fLO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/$f.zip
  unzip -o $f.zip -d ~/.local/share/fonts/$f
  rm $f.zip
done
fc-cache -f
```

## 8. Dots

```bash
git clone --bare git@github.com:Dechrissen/dots.git $HOME/.dotfiles
alias dots='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
dots config status.showUntrackedFiles no
mv ~/.bashrc ~/.bashrc.debian-default
dots checkout
source ~/.bashrc
```

## 9. Storage server mount in `fstab`
