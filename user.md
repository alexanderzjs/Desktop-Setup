# User Workspace Setup
## Software configuration files
1. Xorg: `~/.xinitrc`, it is originally copied from `/etc/X11/xinit/xinitrc`
2. Rofi: `~/.config/rofi/config.rasi`, you can try this one: https://gist.github.com/EgZvor/ea6343988d0a1d66481daa79b7de6fbd. Please manually edit this.
3. Polybar: `~/.config/polybar/launch.sh`
4. Herbstluftwm: `~/.config/herbstluftwm/autostart`, it is originally copied from `/etc/xdg/herbstluftwm/autostart`
5. Starship: `~/.config/starship.toml`, you can try this one: https://github.com/theRubberDuckiee/dev-environment-files/blob/main/starship.toml.
## Install software
1. Remote login as normal user and install software by `sudo -E bash -c 'pacman -S zsh herbstluftwm rofi polybar starship'` and `paru -S wezterm microsoft-edge-stable-bin --mflags="--skippgpcheck" --sudoflags="http_proxy=$http_proxy https_proxy=$https_proxy no_proxy=$no_proxy"`.
2. Save below script as `a.sh`, give it execution privilege and execute it. Remember to replace `XXX` with proxy address.
```
cp /etc/X11/xinit/xinitrc ~/.xinitrc
sed -i 's/twm/#twm/g' ~/.xinitrc
sed -i 's/xclock/#xclock/g' ~/.xinitrc
sed -i 's/xterm/#xterm/g' ~/.xinitrc
sed -i 's/exec/#exec/g' ~/.xinitrc
echo "exec herbstluftwm" >> ~/.xinitrc
mkdir -p ~/.config/rofi/
touch ~/.config/rofi/config.rasi
mkdir -p ~/.config/polybar/
echo "#!/bin/bash" > ~/.config/polybar/launch.sh
echo "killall -q polybar" >> ~/.config/polybar/launch.sh
echo "polybar &" >> ~/.config/polybar/launch.sh
chmod +x ~/.config/polybar/launch.sh
mkdir -p ~/.config/herbstluftwm
cp /etc/xdg/herbstluftwm/autostart ~/.config/herbstluftwm/autostart
sed -i 's@#!/usr/bin/env bash@#!/usr/bin/env bash\n~/.config/polybar/launch.sh \&\nfcitx5 \&@g' ~/.config/herbstluftwm/autostart
sed -i 's@hc keybind $Mod-Return spawn "${TERMINAL:-xterm}"@hc keybind $Mod-Return spawn wezterm@g' ~/.config/herbstluftwm/autostart
sed -i 's@# basic movement@hc keybind $Mod-d spawn rofi -show drun\n\n# basic movement@g' ~/.config/herbstluftwm/autostart
touch ~/.config/starship.toml
echo 'export http_proxy="http://XXX/"' >> ~/.zshrc
echo 'export https_proxy="http://XXX/"' >> ~/.zshrc
echo 'eval "$(starship init zsh)"' >> ~/.zshrc
chsh -s /usr/bin/zsh
```
Don't forget to update rofi's configuration `config.rasi` and Starship's configuration `starship.toml`.

**Note: Since you select a window manager, please update startup script for xrdp by `sudo sed -i 's@xterm@sh ~/.xinitrc@g' /etc/xrdp/startwm.sh`**

## Install fonts
1. Remote login session as normal user.
2. Save below script as `font.sh`, give it execution privilege and execute it.
```
sudo -E bash -c 'pacman -S wqy-zenhei fcitx5 fcitx5-configtool fcitx5-chinese-addons'
```
