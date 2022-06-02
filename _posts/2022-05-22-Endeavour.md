---
title: "EndeavourOS Windows 11 Dual Boot"
author_profile: true
toc: true
excerpt: "The trick is to get Windows 11 really drunk and sneak out of the house while it hangs rebooting."
header:
  overlay_image: /assets/img/home-header.jpg
  tags:
    - howto
    - endeavouros
    - pacman
    - linux
    - shrink ntfs partition
    - ansible
    - checksum
---
## Intro
My new laptop came with Windows 11 and I'm not saying that's why absolutely nothing is working, but nothing's working. I've enjoyed using [EndeavourOS](https://endeavouros.com) for my main virtual machine recently so I'm going to try it out as my primary os.
The plan is to:

* backup a list of explicitly installed packages and ssh keys
* shrink the windows partition
* download the latest EndeavourOS release (currently Apollo)
* prepare installation media
* install
* configure with Ansible

EndeavourOS allows packages to be added via a list using the Calamares installer but I went with Ansible because I'd eventually like to build off of the playbook to simplify configuration on non Arch based distros, e.g. Debian.
## Backup
I backed up my ssh keys to private S3 bucket and exported my packages with:
```bash
# backup path
pkgs=/home/${USER}/repos/dotfiles/bak
# get explicitly installed packages excluding
# "foreign" packages
pacman -Qqe | grep -Fvx "$(pacman -Qqm)" > \
${pkgs}/$(date --iso-8601='minutes').pkgs
# get "foreign" packages
pacman -Qqm > \
${pkgs}/$(date --iso-8601='minutes').pkgs.aur
```
I added this to my `.zshrc` so that I get a backup for every new shell. The `date` command substitution adds a timestamp to the filename. A `cronjob` would be a neater solution.
## Shrink Windows Partition

* Press **Win+r** to open the run dialog and type `diskmgmt.msc` to open the Disk Management mmc.
* Verify that the **File System** listed for your installation partition (typically `C:`) is listed as **NTFS**
* Right click the Volume and select **Shrink Volume**
* Shrink the partition leaving enough room for both OS's (preference)

## Download EndeavourOS
[Download](https://endeavouros.com/latest-release/) however you'd like. Verify the sha512 checksum (download first) on Linux with:
```bash
sha512sum <iso-path> -c <checksum-path>
```
or Windows command prompt with:
```
certutil -hashfile <iso-path> SHA512
```
comparing the output to the content of the checksum file.
## Prepare Installation Media
I prefer to do this on linux so I don't have to download a 3rd party tool. I have already written detailed instructions [here](https://michael-dez.github.io/InstallingProxmox/#preparing-the-installation-media) on doing this with linux. The `dd` I used for **reference**:
```bash
sudo dd if=/media/sf_Share/EndeavourOS_Apollo_22_1.iso of=/dev/sdb status=progress
```
## Install
Reboot, and make sure to press the key to change BIOS boot options before Windows starts. From there you will go through the Calamares Installer wizard.
### Partition
If dual booting with Windows, choose **Manual Partition**
**Don't alter your Windows partition if you intend to keep it**

* Select unused space and create a 600 MB partition with a mountpoint of `/boot/efi` and `FAT32` file system
* Select unused space and create a partition (as large as desired) with a mountpoint of `/` and `ext4` file system

## Configure with Ansible
When installation's complete install Ansible:
```bash
sudo pacman -S ansible
```
And here's the playbook I ran for some inspiration. Run with `ansible-playbook install.yml`.

```yaml
---
# ./install.yml
# currently only installs specified packages
# TODO: * install configuration files
# * install packages from var file.yaml
# * also support debian/arch

- hosts: localhost
  connection: local
  become: yes
  vars:
    pkg:
      - accountsservice
      - acpi
      - adobe-source-han-sans-cn-fonts
      - adobe-source-han-sans-jp-fonts
      - adobe-source-han-sans-kr-fonts
      - alsa-firmware
      - alsa-plugins
      - alsa-utils
      - ansible
      - arandr
      - arc-gtk-theme-eos
      - archlinux-xdg-menu
      - asciinema
      - autoconf
      - automake
      - awesome-terminal-fonts
      - aws-cli
      - b43-fwcutter
      - base
      - bash-completion
      - bind
      - binutils
      - bison
      - bluez
      - bluez-utils
      - btrfs-progs
      - cantarell-fonts
      - cryptsetup
      - device-mapper
      - dex
      - dhclient
      - dialog
      - diffutils
      - dmenu
      - dmidecode
      - dmraid
      - dnsmasq
      - docker
      - dosfstools
      - downgrade
      - duf
      - dunst
      - e2fsprogs
      - efibootmgr
      - efitools
      - endeavouros-keyring
      - endeavouros-mirrorlist
      - endeavouros-skel-i3wm
      - endeavouros-theming
      - endeavouros-xfce4-terminal-colors
      - eos-apps-info
      - eos-hooks
      - eos-lightdm-slick-theme
      - eos-log-tool
      - eos-packagelist
      - eos-qogir-icons
      - eos-quickstart
      - eos-rankmirrors
      - eos-update-notifier
      - ethtool
      - exfatprogs
      - f2fs-tools
      - fakeroot
      - ffmpegthumbnailer
      - file
      - file-roller
      - findutils
      - firefox
      - firewalld
      - flex
      - fsarchiver
      - galculator
      - gawk
      - gcc
      - gettext
      - git
      - glances
      - gnu-netcat
      - go
      - gopls
      - grep
      - groff
      - grub
      - grub-tools
      - grub2-theme-endeavouros
      - gst-libav
      - gst-plugin-pipewire
      - gst-plugins-bad
      - gst-plugins-ugly
      - gthumb
      - gtk-engine-murrine
      - gvfs
      - gvfs-afc
      - gvfs-gphoto2
      - gvfs-mtp
      - gvfs-nfs
      - gvfs-smb
      - gzip
      - haveged
      - hdparm
      - hwdetect
      - hwinfo
      - i3-gaps
      - i3blocks
      - i3lock
      - i3status
      - inetutils
      - intel-ucode
      - inxi
      - ipw2100-fw
      - ipw2200-fw
      - iwd
      - jfsutils
      - jq
      - jupyter-notebook
      - keepassxc
      - less
      - libdvdcss
      - libgsf
      - libopenraw
      - libtool
      - libwnck3
      - lightdm
      - lightdm-slick-greeter
      - linux
      - linux-firmware
      - linux-headers
      - logrotate
      - lsb-release
      - lsscsi
      - lvm2
      - lxappearance-gtk3
      - m4
      - make
      - man-db
      - man-pages
      - mdadm
      - meld
      - mesa-utils
      - mkinitcpio
      - mkinitcpio-busybox
      - mkinitcpio-nfs-utils
      - mkinitcpio-openswap
      - mlocate
      - modemmanager
      - mpv
      - mtools
      - nano
      - nano-syntax-highlighting
      - nbd
      - ndisc6
      - neofetch
      - neovim
      - netctl
      - network-manager-applet
      - networkmanager
      - networkmanager-openvpn
      - nfs-utils
      - nilfs-utils
      - nitrogen
      - nmap
      - noto-fonts
      - npm
      - nss-mdns
      - ntfs-3g
      - ntp
      - numlockx
      - openconnect
      - openvpn
      - os-prober
      - pacman
      - pacman-contrib
      - pandoc
      - patch
      - pavucontrol
      - perl
      - picom
      - pipewire-alsa
      - pipewire-jack
      - pipewire-media-session
      - pipewire-pulse
      - pkgconf
      - pkgfile
      - playerctl
      - polkit-gnome
      - poppler-glib
      - ppp
      - pptpclient
      - pv
      - pyright
      - python
      - python-capng
      - python-defusedxml
      - python-packaging
      - python-pip
      - python-pyqt5
      - qutebrowser
      - rebuild-detector
      - reflector
      - reflector-simple
      - reiserfsprogs
      - ripgrep
      - rofi
      - rp-pppoe
      - rsync
      - s-nail
      - scrot
      - sed
      - sg3_utils
      - smartmontools
      - sof-firmware
      - sshfs
      - sudo
      - sysfsutils
      - sysstat
      - systemd-sysvcompat
      - terraform
      - texinfo
      - texlive-core
      - thunar
      - thunar-archive-plugin
      - thunar-volman
      - tldr
      - tmux
      - tree
      - ttf-bitstream-vera
      - ttf-dejavu
      - ttf-liberation
      - ttf-opensans
      - tumbler
      - unrar
      - unzip
      - upower
      - usb_modeswitch
      - usbutils
      - vi
      - virtualbox-guest-utils
      - vpnc
      - welcome
      - wget
      - which
      - whois
      - wireless-regdb
      - wireless_tools
      - wpa_supplicant
      - xbindkeys
      - xclip
      - xdg-user-dirs
      - xdg-user-dirs-gtk
      - xdg-utils
      - xed
      - xf86-input-libinput
      - xf86-video-vmware
      - xfce4-terminal
      - xfsprogs
      - xl2tpd
      - xorg-server
      - xorg-xbacklight
      - xorg-xdpyinfo
      - xorg-xinit
      - xorg-xinput
      - xorg-xkill
      - xorg-xrandr
      - xterm
      - yad-eos
      - yay
      - yq
      - zsh
      - zsh-syntax-highlighting
      
  tasks:
    - name: "Upgrade all installed packages"
      pacman:
        update_cache: yes
        upgrade: yes

    - name: "Install from package list"
      package:
          name: "{{ pkg }}"
          state: latest
      when:
          - pkg is defined
          - pkg|length|int >=1
          - pkg[0]|length|int >= 1    
```
