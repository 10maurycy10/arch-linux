# Install arch linux alongside other linux w/ boot device

## WHY?

This is for when you want to say "I use arch btw" but dont want to wipe you OS.

## Waning!

While this procedure wont touch you root partition, it is *strongly* recommended to have backups while messing around with installing OSs!

If you do not understand what a step will do or why it is needed, it is not recommended that you continue!

## preparations

you will need a unused partition of around 1GB or larger, this can be done by formatting swap space.

Remember to disable swapping before formatting swap.

```sudo swapoff -a```

editing fstab to not activate swap is recommended.

```sudo nano /etc/fstab```

## obtain arch linux

First click on a mirror on https://archlinux.org/download/.

And then download the bootstrap archive (.tar.gz);

## install it

You will need to be root for all of these steps.

```
sudo su
```

Now create a directory on the host system.

```
mkdir /arch
```

```
cd /arc
```

Unpack the archive. ( ``--numeric-owner`` is required to make sure root owned stuff ends up owned by UID=0)

```
tar xzf <path-to-bootstrap-archive>/archlinux-bootstrap-*-x86_64.tar.gz --numeric-owner
```

Select a mirror by editing ``root.x86_64/etc/pacman.d/mirrorlist``

Arches pacman wont work if root is not mounted, so we will have to make a dummy mount.

```
mount --bind root.x86_64 root.x86_64
```

You can use the arch-chroot script from the chroot.

```
./root.x86_64/bin/arch-chroot root.x86_64
```

Initialize pacman's keyring

```
pacman-key --init
pacman-key --populate archlinux
```

Install a text editor

```
pacman -Sy nano
```

Mount the partition you want to install arch to on /mnt of the chroot.

```
mount /dev/sdXY /mnt
```

Mount the EFI partiton in ``/mnt`` of the chroot.

```
mkdir /mnt/boot
```

```
mount /dev/sdXY /mnt/boot
```

Install required packages on /mnt of the chroot. (add neofetch if you want to show off)

```
pacstrap /mnt linux linux-firmware lvm2 nano grub util-linux networkmanager efibootmgr os-prober
```

Generate an fstab for the new system, you will need to edit it to remove incorrect lines.

```
genfstab > /mnt/etc/fstab
nano /mnt/etc/fstab
```

Chroot into the new system.


```
arch-chroot /mnt
```

Set the root password. (You should do this if you want to use your system!)

```
passwd
```

Now visit the arch wiki for information on how to install grub on your system.

NOTE: backup your $ESP/grub/grub.cfg somewhere on $ESP. this way you can you can run the  ``configfile`` command to load your old config, if you messed up. 

## Going further

### overwrite old system

mount old system.

```
mount /dev/sdaXY
```

While booted into your new system you can use rsync to clone files.

```
rsync -aAXHv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/home/*","/root/*"} / /mnt
```

Mount /boot on the old system.

```
mount /dev/sdXY /mnt/boot
```


Chroot, onto the old systems root.

```
chroot /mnt
```


Then you will need to adjust ``/mnt/etc/defaults/grub`` and ``/etc/fstab``.

And finally reinstall and configure grub.

### undoing the process.

mount old system.

```
mount /dev/sdaXY
```

Mount /boot on the old system.

```
mount /dev/sdXY /mnt/boot
```

Chroot, onto the old systems root.

```
chroot /mnt
```

Install grub in the chroot. (you may have to use ``/boot/efi`` instead of ``/boot``)

```grub-install /dev/sdaX --efi-directory=/boot/```
