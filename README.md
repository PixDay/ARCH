# ARCH

## Summary:

1. Set keyboard layout
2. Manage partitions
3. Connect to WI-FI
4. Mounts disks and bootloader

---

### 1. Set keyboard layout:

Change your keyboard layout using `loadkeys [lang]` for french layout: `loadkeys fr`

---

### 2. Manage partitions

To manage partitions, we will use `fdisk` for more info `man fdisk`

First of all, display the list of existing disk in your machine using: `fdisk -l`

Open the fdisk utils giving a disk name (for the next steps we are going to use `nvme0n1` as disk name): `fdisk nvme0n1`

#### Inside the fdisk utils:

> Delete all existing partitions: `d` repeat for all partitions

> Create new partition for the grub and bootload: `n` press enter 3 times then give half a Gib `+512M` [Y] (by default this will be partition 1 alias p1)

> Press `t` to see list of the partition type

> Set to 1 for EFI

> Create another partition: `n` press enter 4 times (to take all the space available) [Y] (by default this will be partition 2 alias p2)

> Write the changes on disk: `w` then exit the utils `exit`

Format the new partitions previously created:

- p1: `mkfs.fat -F32 nvme0n1p1`
- p2: `mkfs.ext4 nvme0n1p2`

### 3. Connect to WI-FI

To connect to the WI-FI we will use `iwctl`

> Run the `help` command to get the documentation

> To connect to your WI-FI just run `station wlan0 connect[wi-fi name]` it will then ask for the password and you should be connected

> Exit iwctl with `exit`

### 4. Mounts disks and bootloader

Mount your second partition on `/mnt` with the following command: `mount nvme0n1p2 /mnt`

Run: `pacstrap /mnt base linux linux-firmware vim` and `genfstab -U /mnt /mnt/etc/fstab`

We are now gonna be root on /mnt: `arch-chroot /mnt` we have to change the password using `passwd`

Let's change the lang by editing `/etc/locale.gen` and uncomment the `[lang].UTF-8 UTF-8` for me it should always be english so uncomment `en_US.UTF-8 UTF-8`

Generate the lang with `locale.gen` export it in `/etc/locale.conf` with the echo command: `echo LANG=en_US.UTF-8 > /etc/locale.conf` then `export LANG=en_US.UTF-8`

Let's setup the timezone: `timedatectl list-timezones` (!! most of the time I have an error can't be done here !!)

Let's setup localhost networking: `echo workbench > /etc/hostname` and add in `/etc/hostname` the following lines: `127.0.0.1 localhost` `::1 localhost` and `127.0.1.1 workbench`

Now let's handle the bootload, type all the following commands: 

`mkdir /boot/efi`

`mount /dev/nvme0n1p1 /boot/efi`

`pacman -S grub efibootmgr`

`grub-install --target=x86_64-efi --bootloader-id=ARCH --efi-directory=/boot/efi`

`grub-mkconfig -o /boot/grub/grub.cfg`

`cd boot`

`cd efi`

`cd EFI`

`mkdir boot`

`cd ARCH`

copy the only file to the previous boot folder and go back HOME `cp file ../boot && cd`

Time for GUI, first install xorg: `pacman -S xorg` once done you can isntall the GUI you prefer with pacman (I like gnome or kde): `pacman -S gnome` and enable it: `systemctl enable gdm`

Enable network manager: `systemctl enable NetworkManager`

Restart your machine you are done :+1:
