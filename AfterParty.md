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
sudo pacman -S xfce4-terminal picom dunst gvfs pcmanfm zsh lxappearance ntfs-3g bc alsa-utils pulseaudio pavucontrol xorg-xbacklight  redshift feh imagemagick pulseaudio-alsa pulseaudio-equalizer pulseaudio-jack xorg-xsetroot mpv youtube-dl glances telegram-desktop xf86-video-intel jdk8-openjdk jdk-openjdk gnome-keyring
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