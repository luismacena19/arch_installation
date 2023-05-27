# My install instructions to [archlinux](https://archlinux.org/) in a luks container in a EFI system.

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

## Disk layout shoud be like:

| Partition | Name | Partition type | FS Type | Size |
| --- | --- | --- | --- | --- |
| /boot | /dev/sda1 | linux | ext4 | 512 MiB |
| /boot/EFI | /dev/sda2 | EFI | FAT32 | 512 MiB |
| luks | /dev/sda3 | Linux | linux | rest of the disk |

**It is worth noting that the / boot / EFI / and / boot / partition is essential for some commands of this installation.

If you prefer another layout, look for information on the wiki https://wiki.archlinux.org/index.php/Installation_guide and adapt the commands to suit you.**

After creating the system partitions we need to format and create the container.

```bash
mkfs.ext4 /dev/sda1

mkfs.fat -F32 /dev/sda2

cryptsetup -y -v luksFormat /dev/sda3
```

Now we can open the container, and also, optionally (I recommend), we will write zeros on the SSD.

Note that chunsuke is an arbitrary name that you can choose freely, BUT, that name will be used to mount the container in /dev/mapper/ . So in this tutorial it's gona be chunsuke.

```bash
cryptsetup luksOpen /dev/sda3 chunsuke

pvcreate /dev/mapper/chunsuke

vgcreate VGMaruGroup /dev/mapper/chunsuke

lvcreate -L 60G VGMaruGroup -n root 

lvcreate -l 100%FREE VGMaruGroup -n home 

mkfs.ext4 /dev/VGMaruGroup/home 

mkfs.ext4 /dev/VGMaruGroup/root 

mount /dev/VGMaruGroup/root /mnt 

mkdir /mnt/home

mount /dev/VGMaruGroup/home /mnt/home

```

After that we need to mount de filesystem so we can create the directories that grub uses for boot.

```bash

mkdir -p /mnt/boot

mount /dev/sda1 /mnt/boot

sync
```

Now lets install the system, generate fstab and chroot into the system

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware neovim efibootmgr

genfstab -p /mnt >> /mnt/etc/fstab

arch-chroot /mnt
```

In the system, we need to mount the EFI partition and define the environment variables as well as the system language, locale, time and user. We can use KEYMAP=thinkpad60 insted of KEYMAP=br-abnt2 if we are in an old thinkpad like T420.

```bash
mkdir /boot/EFI

mount /dev/sda2 /boot/EFI

ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime

echo hostname > /etc/hostname

nvim /etc/locale.gen

locale-gen

cat <<EOT >> /etc/locale.conf
LANG=pt_BR.UTF-8
LC_ADDRESS=pt_BR.UTF-8
LC_IDENTIFICATION=pt_BR.UTF-8
LC_MEASUREMENT=pt_BR.UTF-8
LC_MONETARY=pt_BR.UTF-8
LC_NAME=pt_BR.UTF-8
LC_NUMERIC=pt_BR.UTF-8
LC_PAPER=pt_BR.UTF-8
LC_TELEPHONE=pt_BR.UTF-8
LC_TIME=pt_BR.UTF-8
EOT


cat <<EOT >> /etc/vconsole.conf
KEYMAP=br-abnt2
FONT=ter-v20n
FONT_MAP=8859-2
EOT
 

passwd

useradd -m -g users -G wheel $username

passwd username
```

We need to populate the hosts file. Change hostname to your hostname

```bash
nvim /etc/hosts
```

```bash
127.0.0.1 localhost

::1 localhost

127.0.1.1 hostname.localdomain hostname
```

Now we can install some needed programs.

```bash
pacman -S dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant git xorg-server xorg-xinit wireless_tools dialog terminus-font grub lvm2 dhcpcd --noconfirm
```

Enable Network and edit mkinitcpio.conf

```bash
systemctl enable dhcpcd

systemctl enable NetworkManager

nvim /etc/mkinitcpio.conf
```

We gona add **encrypt** hook on the hooks line.

```bash
(HOOKS="base udev autodetect modconf block encrypt lvm2 filesystems keyboard fsck")
```

Now we run mkinitcpio with kernel version of our choice

```bash
mkinitcpio -p linux
```

After that we need to add the name and the path to the luks container on GRUB_CMDLINE_LINUX and uncomment GRUB_ENABLE_CRYPTODISK

```bash
nvim /etc/default/grub

GRUB_ENABLE_CRYPTODISK=y

GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda3:chunsuke"
```

Now we can install grub. The option --bootloader-id can be used to choose whatever name you like.

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=ARCHLINUX

grub-mkconfig -o /boot/grub/grub.cfg
```

And now the system is installed an everything shoud be fine, so exit chroot, umount the partitions and reboot to see your new system.

```bash
exit

umount -a

reboot
```

And we are done ! **Enjoy :)**
