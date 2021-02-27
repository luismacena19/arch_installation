
# My install instructions to archlinux in a luks container using btrfs and snapper !
First, if using ssh set password:
```
passwd
```
Start sshd and get the ip 
```
systemctl start ssh && ip a
```
Then is just login using the password set
If you not using ssh you need to set a layout to you keyboard
```
 loadkeys br-abnt2
 ```
 Now lets create the partitions, you can use whatever you like to create then.
 Please read https://wiki.archlinux.org/index.php/partitioning and https://wiki.archlinux.org/index.php/Dm-crypt for more information.
 I personally like to use fdisk and to create this layout:
## Disk layout  shoud be like:

| Partition   | Name 	  | Partition type 		| FS Type   	| Size   			|
|-------------|-----------|---------------------|---------------|-------------------|
| /boot/EFI   | /dev/sda1 | EFI System Partition| FAT32     	| 512 MiB			|
| /boot       | /dev/sda2 | Linux 		   		| ext4      	| 512 MiB			|
| [SWAP]      | /dev/sda3 | Linux swap 			| SWAP      	| 16 GiB 			|
| luks		  | /dev/sda4 | Linux 				| linux		    | rest of the disk	|


**It is worth noting that the / boot / EFI / and / boot / partition is essential for some commands of this installation.
If you prefer another layout, look for information on the wiki https://wiki.archlinux.org/index.php/Installation_guide and adapt the commands to suit you.**
After creating the system partitions we need to format and create the container.
```
mkfs.ext4 /dev/sda1
mkfs.fat -F32 /dev/sda2 
mkswap /dev/sda3 
cryptsetup -y -v luksFormat /dev/sda4 
``` 
Now we can open the container, and also, optionally (I recommend), we will write zeros on the SSD.
Note that containername is an arbitrary name that you can choose freely, BUT, that name will be used to mount the container in /dev/mapper/ . So in this tutorial it's gona be containername.
```
cryptsetup luksOpen /dev/sda4 containername 
dd if=/dev/zero of=/dev/mapper/container
mkfs.btrfs /dev/mapper/containername 
``` 
After that we need to mount de filesystem so we can create the subvolumes
```
mount -o noatime,compress=lzo,discard,ssd,defaults /dev/mapper/containername /mnt 
btrfs subvolume create /mnt/@ 
btrfs subvolume create /mnt/@home 
btrfs subvolume create /mnt/@var 
btrfs subvolume create /mnt/@snapshots 
umount /mnt 
```
It's necessary to unmount the container to be able to mount the system properly, It is necessary to dismantle the container in order to mount the system correctly, as well as create the directories.
```
mount -o noatime,compress=lzo,discard,ssd,defaults,space_cache,subvol=@ /dev/mapper/containername /mnt 
mkdir -p /mnt/{boot/EFI,home,var,.snapshots} 
mount -o noatime,compress=lzo,space_cache,subvol=@home /dev/mapper/containername  /mnt/home 
mount -o noatime,compress=lzo,space_cache,subvol=@var /dev/mapper/containername /mnt/var 
mount -o noatime,compress=lzo,space_cache,subvol=@snapshots /dev/mapper/containername /mnt/.snapshots 
mount /dev/sda2 /mnt/boot/ 
swapon /dev/sda3
sync 
```
Now lets install the system, generate fstab and chroot into the system

```
pacstrap /mnt base base-devel linux linux-headers linux-firmware neovim btrfs-progs
genfstab -p /mnt >> /mnt/etc/fstab (garantir que sejam usadas a localização das partições)
arch-chroot /mnt
```
In the system, we need to mount the EFI partition and define the environment variables as well as the system language, locale, time and user.
```
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
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
```
nvim /etc/hosts 
```
```
127.0.0.1	localhost
::1			localhost
127.0.1.1	hostname.localdomain	hostname
```
Now we can install some needed programs.
```
pacman -S dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant git xorg-server xorg-xinit wireless_tools dialog terminus-font grub linux-lts grub-btrfs efibootmgr snapper --noconfirm
```
Enable Network Manager and edit mkinitcpio.conf
```
systemctl enable NetworkManager 
nvim /etc/mkinitcpio.conf 
```
We gona add **encrypt** hook on the hooks line as well as **btrfs** on the modules line.

```
MODULES=(btrfs)

(HOOKS="base udev autodetect modconf block encrypt filesystems keyboard fsck btrfs")
```
Now we run mkinitcpio with kernel version of our choice
```
mkinitcpio -p linux-lts
```
After that we need to add the name and the path to the luks container on GRUB_CMDLINE_LINUX and uncomment GRUB_ENABLE_CRYPTODISK
```
nvim /etc/default/grub 
```
```
GRUB_ENABLE_CRYPTODISK=y
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda4:containername"
```
Now we can install grub. The option --bootloader-id can be used to choose whatever name you like. 
```
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```
And now the system is installed an everything shoud be fine, so exit chroot, umount the partitions and reboot to see your new system.
```
exit
umount -a
reboot
```
After uncrypt the system and login with root we need to configure snapper
```
umount /.snapshots
rm -r /.snapshots 
snapper -c root create-config /  && sudo snapper -c home create-config /home/ 
mount -o compression=lzo,discard,noatime,nodiratime,subvol=@snapshots /dev/mapper/containername /.snapshots 
systemctl start snapper-timeline.timer
```
You can add your user to sudoers now if you want
```
nvim /etc/sudoers
```
```
root  ALL=(ALL) ALL
username  ALL=(ALL) ALL
```
And we are done ! **Enjoy :)**