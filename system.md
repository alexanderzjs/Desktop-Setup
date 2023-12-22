# System Setup
## Desktop Hardware
* Intel Xeon Silver 4109T, 64 GB.
* Nvidia Quadro P400.
* Hard drive, 2TB.
* Static IP address behind a proxy.

## Create a LiveUSB and prepare BIOS
1. Download Archlinux installation ISO file.
2. Download Rufus from https://rufus.ie or other equivalent.
3. Create a bootable LiveUSB with GPT partition and UEFI mode.
4. Disable secure boot in BIOS.
5. Insert LiveUSB to desktop and boot into Live Archlinux.

## Enable SSH access for Live OS
1. Login to desktop as root user.
2. Save below script as `first.sh`, give it execution privilege and execute it.
```
passwd
sed -i 's/#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/g' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
ip a
```
Note: Remember this IP address as "Live OS IP".

## Partition disk
1. Remote login as root user with "Live OS IP".
2. `cfdisk /dev/sda` to start partition disk (Don't forget to write your change).
    * 1GB, EFI System partition, `/boot`.
    * 1TB, Ext4 partition, `/`.
    * 884GB, Ext4 partition, `/home`.
3. Save below script as `second.sh`, give it execution privilege and execute it.
```
mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
mount --mkdir /dev/sda3 /mnt/home
```

## Install base system
1. Continue remote login as root and save below script as `third.sh`, give it execution privilege and execute it. Remember to replace the following in the script:
    * XXX: proxy address
    * YYY: fastest Archlinux repository mirror address for you
```
export http_proxy="XXX"
export https_proxy="XXX"
timedatectl set-ntp true
echo "Server = https://YYY/archlinux/\$repo/os/\$arch" > /etc/pacman.d/mirrorlist 
pacstrap -K /mnt base base-devel linux linux-firmware intel-ucode networkmanager vim grub openssh git efibootmgr sudo
genfstab -U /mnt > /mnt/etc/fstab
arch-chroot /mnt
```
3. The current user is root of the newly installed system. Save below script as `fourth.sh`, give it execution privilege and execute it. Remember to replace the following in the script:
    * XXX/YYY: region and city, e.g. Asia/Shanghai
    * ZZZ: normal username newly created

Note that, the following script will ask for two passwords to be typed in. First one is root user's password, second one is newly created user's password.
```
ln -sf /usr/share/zoneinfo/XXX/YYY /etc/localtime
hwclock --systohc
sed -i 's/#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
passwd
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=archlinux
grub-mkconfig -o /boot/grub/grub.cfg
sed -i 's/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/g' /etc/sudoers
useradd -G wheel -m ZZZ
passwd ZZZ
```
4. The current user is newly installed system root user, execute `exit` to return to Live OS root user.
5. Finish installation by `umount -R /mnt && reboot`. Remember to remove your Live USB, otherwise it might boot to your Live USB again.

## Enable network and SSH for newly installed system
1. Login as root user to newly installed system.
2. Save below script as `fifth.sh`, give it execution privilege and execute it. Remember to replace the following in the script:
    * XXX: proxy address
```
systemctl enable NetworkManager --now
sed -i 's/#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/g' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl enable sshd --now
echo 'export http_proxy="http://XXX/"' >> /etc/bash.bashrc
echo 'export https_proxy="http://XXX/"' >> /etc/bash.bashrc
ip a
```
Note: Remember this IP address as "System IP".

## Install AUR package manager paru
1. Remote login as normal user with "System IP".
2. Save below script as `sixth.sh`, give it execution privilege and execute it.
```
git clone https://aur.archlinux.org/paru-bin.git
cd paru-bin
makepkg -si
cd ../ && rm -rf paru-bin
```

## Install remote desktop components
There are four subsections:
1. Common setup for remote desktop session.
2. No audio and video support for remote session.
3. Audio support for remote session.
4. GPU acceleration for remote session.

Note: Common setup **MUST** be executed. After common setup is done, you can go with your selection. You can also combine 3 and 4 to get both audio support and GPU acceleration for remote session.
### Common setup for remote desktop
1. Continue remote login session as normal user.
2. Save below script as `seventh.sh`, give it execution privilege and execute it. 
```
sudo -E bash -c 'pacman -S xorg-server xorg-xinit'
cp /etc/X11/xinit/xinitrc ~/.xinitrc
sed -i 's/twm/#twm/g' ~/.xinitrc
sed -i 's/xclock/#xclock/g' ~/.xinitrc
sed -i 's/xterm -geometry/#xterm -geometry/g' ~/.xinitrc
sed -i 's/exec #xterm/#exec xterm/g' ~/.xinitrc
```
3. Switch to root user with `sudo su` and execute `echo "allowed_users = anybody" >> /etc/X11/Xwrapper.config` and `echo "needs_root_rights = no" >> /etc/X11/Xwrapper.config`

### Remote desktop without audio and video
1. Continue remote login session as normal user.
2. Save below script as `eighth.sh`, give it execution privilege and execute it. 
```
paru -S xrdp xorgxrdp --mflags="--skippgpcheck" --sudoflags="http_proxy=$http_proxy https_proxy=$https_proxy no_proxy=$no_proxy"
sudo systemctl enable xrdp --now
sudo systemctl enable xrdp-sesman --now
```
### Remote desktop with audio support
1. Continue remote login session as normal user.
2. Save one of below script as `eighth.sh`, give it execution privilege and execute it. Note that, you have two choices for audio server, i.e., `pulseaudio` or `pipewire`. Pipewire is new audio server implementation. I provide remote desktop with both server implementations. You can check `pavucontrol` panel, if it shows "xrdp-sink" instead of "Dummy output", you are able to hear sound from remote desktop client.

**RDP for pulseaudio**
```
sudo -E bash -c 'pacman -S pulseaudio pulseaudio-bluetooth pavucontrol'
paru -S pulseaudio-module-xrdp --mflags="--skippgpcheck" --sudoflags="http_proxy=$http_proxy https_proxy=$https_proxy no_proxy=$no_proxy"
sed -i 's@exec herbstluftwm@sh /usr/libexec/pulseaudio-module-xrdp/load_pa_modules.sh \&\nexec herbstluftwm@g' ~/.xinitrc
systemctl --user enable pulseaudio --now
```
**RDP for pipewire**
```
sudo -E bash -c 'pacman -S pipewire-pulse pavucontrol'
paru -S pipewire-module-xrdp-git --mflags="--skippgpcheck" --sudoflags="http_proxy=$http_proxy https_proxy=$https_proxy no_proxy=$no_proxy"
sed -i 's@exec herbstluftwm@sh /usr/lib/pipewire-module-xrdp/load_pw_modules.sh \&\nexec herbstluftwm@g' ~/.xinitrc
systemctl --user enable pipewire --now
systemctl --user enable pipewire-pulse --now
```
3. After installation, you'd better reboot the system.

### Remote desktop with GPU acceleration
To debug remote desktop connection has GPU acceleration, you can check `~/.xorgxrdp.*.log` for Xorg log, monitor `sudo journalctl -f` for xrdp and kernel log during remote login. To test if your RDP session is GPU accelerated, you can run `nvidia-smi` in an SSH session. If you see `xorgxrdp_helper` in GPU process, you are good to go. There are two variants: Intel/AMD GPU or NVidia GPU.
#### Intel GPU or AMD GPU
1. Remote login session as normal user.
2. The following commands would be sufficient, but I don't have testing device. Sorry about this.
`paru -S xorgxrdp-glamor --mflags="--skippgpcheck" --sudoflags="http_proxy=$http_proxy https_proxy=$https_proxy no_proxy=$no_proxy"`
#### Nvidia GPU
1. Remote login session as normal user.
2. Save below script as `nineth.sh`, give it execution privilege and execute it. Remember to replace the following in the script:
    * XXX: normal user name
    * -I/path/to/git/cloned/xrdp: 
```
sudo -E bash -c 'pacman -S nvidia'
sudo usermod XXX -a -G video
sudo usermod XXX -a -G tty
sudo -E bash -c 'pacman -S fuse2 nasm x264'
git clone https://github.com/Nexarian/xrdp.git
cd xrdp && git checkout mainline_merge_backup
./bootstrap
./configure --enable-fuse --enable-rfxcodec --enable-pixman --enable-mp3lame --enable-sound --enable-opus --enable-fdkaac --enable-x264 --enable-nvenc
make -j8 clean all
sudo make install
cd ..
sudo -E bash -c 'pacman -S xorg-server-devel'
git clone https://github.com/Nexarian/xorgxrdp.git
cd xorgxrdp && git checkout mainline_merge_backup
./bootstrap
XRDP_CFLAGS="-I/path/to/git/cloned/xrdp/common" ./configure --with-simd --enable-lrandr
make -j8 clean all
sudo make install
BUS_ID=$(nvidia-smi --query-gpu=pci.bus --format=csv | sed -n '2 p' | xargs -I{} printf "%d\n" {})
sudo sed -i -E 's/(BusID "PCI:)[[:digit:]]+(:0:0")/\1'$BUS_ID'\2/' /etc/X11/xrdp/xorg_nvidia.conf
sudo systemctl enable xrdp --now
sudo systemctl enable xrdp-sesman --now
```
3. After installation, you'd better reboot the system.

## Reference for system maintenance
If you are feeling good to remove unused pacman cache, you can execute `pacman -Sc`.

If you want to uninstall a pacman installed package, use `pacman -Rcns XXX`. It will remove package, its recursive dependencies, its configuration files.

If you want to cleanup orphan packages, you can execute `pacman -Qtdq | sudo pacman -Rns -`.

