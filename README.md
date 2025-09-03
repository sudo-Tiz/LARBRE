# LARBRE - Arch Linux Auto Setup

Automated post-installation script for Arch Linux that installs and configures a complete Hyprland desktop environment.

## Quick Start

1. **Install Arch Linux** (follow [install guide](install_arch.md))
2. **Run the script as root:**
   ```bash
   curl -LO https://raw.githubusercontent.com/sudo-Tiz/LARBRE/main/larbre.sh
   sh larbre.sh
   ```
3. **Done!** Your system is ready to use.

## What it does

- Creates a new user account
- Installs 200+ packages (desktop environment, applications, development tools)
- Configures Hyprland wayland compositor
- Sets up my personal [dotfiles](https://github.com/sudo-Tiz/dotfilesV2)
- Configures battery monitoring for laptops
- Sets up secure DNS and network privacy

## Included Software

- **Desktop:** Hyprland, [Tsumiki](https://github.com/rubiin/tsumiki), Wofi
- **Terminal:** Foot, Zsh with Powerlevel10k
- **Editor:** Neovim
- **Browser:** Firefox, Brave, Qutebrowser
- **Media:** MPV, Mixxx, Reaper
- **Development:** Git, Docker, VSCodium, Ansible
- **Games:** Steam, Luanti

Full list in [progsV2.csv](progsV2.csv).

## Requirements

- Fresh Arch Linux installation
- Internet connection
- Run as root user

## Customization

Edit these variables at the top of `larbre.sh`:
- `dotfilesrepo`: Your dotfiles repository
- `progsfile`: Your package list (CSV format)

## CSV Format

The package list uses 3 columns:
```
TAG,PACKAGE,DESCRIPTION
,firefox,"is a web browser"
A,yay,"is an AUR helper" 
G,https://github.com/user/repo.git,"is a git program"
```

- Empty TAG = Official repository
- `A` = AUR package  
- `G` = Git repository (make install)

## Other Versions

- **Xorg branch:** For X11 users, check the [xorg branch](https://github.com/sudo-Tiz/LARBRE/tree/Xorg) with my previous configuration

## Credits

Based on [LARBS](https://github.com/LukeSmithxyz/LARBS) by Luke Smith. Special thanks to Luke for creating the original auto-rice bootstrapping concept that highly inspired this project. 