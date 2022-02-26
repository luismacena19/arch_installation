# My install instructions to [archlinux](https://archlinux.org/) in a luks container in a LEGACY system.
First, if using ssh set password:
```bash
passwd
```
Start sshd and get the ip 
```bash
systemctl start ssh && ip a
```
Then is just login using the password set
If you not using ssh you need to set a layout to you keyboard
```bash
 loadkeys br-abnt2
```
 Now lets create the partitions, you can use whatever you like to create then.
 Please read https://wiki.archlinux.org/index.php/partitioning and https://wiki.archlinux.org/index.php/Dm-crypt for more information.
 I personally like to use fdisk and to create this layout:
## Disk layout  shoud be like:

| Partition   | Name 	  | Partition type | FS Type   	| Size   			|
|-------------|-----------|----------------|------------|-------------------|
| /boot       | /dev/sda1 | linux          | FAT32     	| 512 MiB			|
| luks		  | /dev/sda2 | Linux 		   | linux      | rest of the disk	|


**If you prefer another layout, look for information on the wiki https://wiki.archlinux.org/index.php/Installation_guide and adapt the commands to suit you.**
After creating the system partitions we need to format and create the container.
```bash
mkfs.ext4 /dev/sda1
cryptsetup -y -v luksFormat /dev/sda2
```
Now we can open the container, and also, optionally (I recommend), we will write zeros on the SSD.
Note that containername is an arbitrary name that you can choose freely, BUT, that name will be used to mount the container in /dev/mapper/ . So in this tutorial it's gona be containername.
```bash
cryptsetup luksOpen /dev/sda2 containername 
dd if=/dev/zero of=/dev/mapper/container
mkfs.ext4 /dev/mapper/containername 
``` 
After that we need to mount de filesystem so we can create the directories that grub uses for boot.
```bash
mount /dev/mapper/containername /mnt 
mkdir -p /mnt/boot 
mount /dev/sda2 /mnt/boot 
sync 
```
Now lets install the system, generate fstab and chroot into the system

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware neovim
genfstab -p /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```
In the system, we need to define the environment variables as well as the system language, locale, time and user. We can use KEYMAP=thinkpad60 insted of KEYMAP=br-abnt2 if we are in an old thinkpad like T420.
```bash
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
echo hostname  > /etc/hostname 
nvim /etc/locale.gen
locale-gen 
echo LANG=pt_BR.utf8 >> /etc/locale.conf 
echo LANGUAGE=pt_BR >> /etc/locale.conf 
echo LC_ALL=C >> /etc/locale.conf 
echo KEYMAP=br-abnt2 > /etc/vconsole.conf  
passwd 
useradd -m -g users -G wheel username 
passwd username
```
We need to populate the hosts file. Change hostname to your hostname
```bash
nvim /etc/hosts 
```
```bash
127.0.0.1	localhost
::1			localhost
127.0.1.1	hostname.localdomain	hostname
```
Now we can install some needed programs.
```bash
pacman -S dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant git xorg-server xorg-xinit wireless_tools dialog terminus-font grub --noconfirm
```
Enable Network Manager and edit mkinitcpio.conf
```bash
systemctl enable NetworkManager 
nvim /etc/mkinitcpio.conf 
```
We gona add **encrypt** hook on the hooks line.

```bash
(HOOKS="base udev autodetect modconf block encrypt filesystems keyboard fsck btrfs")
```
Now we run mkinitcpio with kernel version of our choice
```bash
mkinitcpio -p linux
```
After that we need to add the name and the path to the luks container on GRUB_CMDLINE_LINUX and uncomment GRUB_ENABLE_CRYPTODISK
```bash
nvim /etc/default/grub 
GRUB_ENABLE_CRYPTODISK=y
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:containername"
```
Now we can install grub.
```bash
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
And now the system is installed an everything shoud be fine, so exit chroot, umount the partitions and reboot to see your new system.
``` bash
exit
umount -a
reboot
```
And we are done ! **Enjoy :)**