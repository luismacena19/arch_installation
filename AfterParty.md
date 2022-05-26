# This section is dedicated to install some things and customizing the system
Let's install some things starting with yay to be able to access AUR !
``` 
git clone https://aur.archlinux.org/yay.git 
cd yay &&\
makepkg -si
```
And enable colors by uncommenting Color option and pacman animation adding ILoveCandy in **Misc options**
```
sudo nvim /etc/pacman.conf
```
Should look like this:
```
#UseSyslog
Color
#TotalDownload
CheckSpace
#VerbosePkgLists
ILoveCandy
```
Installing usefull programs
```
sudo pacman -S xfce4-terminal picom dunst gvfs pcmanfm zsh lxappearance ntfs-3g bc alsa-utils pulseaudio pavucontrol xorg-xbacklight  redshift feh imagemagick pulseaudio-alsa pulseaudio-equalizer pulseaudio-jack xorg-xsetroot mpv youtube-dl glances telegram-desktop xf86-video-intel jdk8-openjdk jdk-openjdk gnome-keyring cronie lxsession polkit
```
And configuring video options, to Intel Graphics
```
sudo nvim /etc/X11/xorg.conf.d/20-intel.conf
```
```
Section "Device"
	Identifier "Intel Graphics"
	Driver "Intel"
	Option "AccelMethod" "sna"
	Option "TearFree" "true"
EndSection 
```
And configuring video options, to Radeon
```
sudo nvim /etc/X11/xorg.conf.d/20-radeon.conf
```
```
Section "Device"
	Identifier "Radeon"
	Driver "radeon"
	Option "TearFree" "true"
	Option "ColorTiling" "on"
EndSec
Fixing java blank apps by
```
sudo echo "export _JAVA_AWT_WM_NONREPARENTING=1" >> /etc/profile.d/jre.sh
```
Installing docker, and adding your user to the group
``` 
lsmod | grep loop
tee /etc/modules-load.d/loop.conf <<< "loop"
sudo tee /etc/modules-load.d/loop.conf <<< "loop"
modprobe loop
sudo modprobe loop
sudo pacman -S docker
systemctl start docker.service
sudo gpasswd -a $USERS docker
```
Installing VirtualBox
```
sudo pacman -S virtualbox virtualbox-guest-iso
sudo modprobe vboxdrv
sudo gpasswd -a $USERS vboxusers
sudo systemctl enable vboxweb.service
sudo systemctl start vboxweb.service && sudo systemctl status vboxweb.service
```
Install FiraCode Nerd Font and Terminess Nerd Font. You need logout and login to be able to select the fonts on your terminal emulator
```
mkdir ~/.fonts && cd ~/.fonts
curl -fLo Terminess\ Nerd\ Font\ Complete  https://github.com/ryanoasis/nerd-fonts/raw/master/patched-fonts/Terminus/terminus-ttf-4.40.1/Regular/complete/Terminess%20\(TTF\)%20Nerd%20Font%20Complete.ttf
curl -fLo Fira\ Code\ Regular\ Nerd\ Font\ Complete.ttf https://github.com/ryanoasis/nerd-fonts/raw/master/patched-fonts/FiraCode/Regular/complete/Fira%20Code%20Regular%20Nerd%20Font%20Complete.ttf
sudo fc-cache -f -v
```
Install Oh my zsh and enabling zsh-autosuggestions, zsh-syntax-highlighting and powerlevel10k plugins
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
Downloading the plugins
 
```
 git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting && git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions && git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

```
Enabling the plugins and choosing powerlevel10k theme 
### Remember, oh my zsh is more attractive with NerdFonts installed and enabled
```
nvim ~/.zshrc 
```
And set ZSH_THEME="powerlevel10k/powerlevel10k"
and add the plugins to the plugins list.
```
ZSH_THEME="powerlevel10k/powerlevel10k"

plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```