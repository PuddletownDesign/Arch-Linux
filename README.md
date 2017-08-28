# Installing Arch Linux in Virtual Box

I'm installing Arch linux 4.12.8-2-Arch on Virtual Box running on a 2015 retina macbook pro. This should work on any os that virtual box supports.

This guide does not cover specialized set ups, keyboard layouts or languages. 

*Thank you Erik Dubios for the basis for this tutorial and the installation scripts*

## Explanation

This is going to be my test run system in a VM before I partition the mac hard drive and duel boot arch linux. I hope to transition over to a 100% linux set up within the year.

This the first guide I've been developing detailing getting a solid set up in arch in an expendable VM.

The next guide will entail installing arch natively on a mac partition.

I'm a software developer, web designer, and musician. I'm also a power user with short cuts. I'd like to have the system optimized for shortcuts, zsh, photoshop and ableton live running through wine.

## Virtual Box Prep

Download arch linux. Ideally by torrent and then leave the torrent seeding.

`archlinux-2017.08.01-x86_64.iso`

Copy the ISO to a new folder to a folder where you hold Operating system images. Leave the old one seeding.

Create a new machine with Virtual Box.

1.  Name Machine "Arch Linux" and confirm Type and version are set to arch linux.
2.  Allocate 1/4 you video memory
3.  Create hard disk
4.  Virtual Box Disk image (default)
5.  Dynamically allocated
6.  Set at least 12GB on a dynamically sized disk.

Finish Installation 

1. Right click Arch Linux in sidebar. Start > Normal Start
2. Select the `archlinux-2017.08.01-x86_64.iso` you copied
3. Start

## Installing Arch

1. Select Boot Arch Linux (x86_64)

You should see a bunch of text and end up at a `root@archiso` command prompt. This is the beginning.

### Partitioning the hard drive

Will will be partitiing the virtual hard drive of the machine we just created in virtual box. This will not effect the host machine. From here on out only the virtual drive.

```
cfdisk /dev/sda
```

1. select a `dos` label type when prompted.
2. select free space and give it a `10G` partition size
3. make it primary
4. make it bootable
5. select free space
6. give it a size of `2G`
7. make it primary
8. select type then `linux swap`
9. select write
10. type `yes` at the prompt and confirm
11. quit

*summary: `cfdisk` is a tool to partition disks. Using it we created a 10GB primary disk for the operating system and data and made it bootable. this is `sda1`. Then we created a 2GB swap disk for the system. Then we wrote the new partition scheme to the disk*

You should now be back at the command prompt after quitting `cfdisk`.

### Making the file system on the primary partition

Now that we have a partition to work with we will create a file system to put on it.

```
mkfs.ext4 /dev/sda1
```
File system is now created!

*we usd a tool predictably called `mkfs` (MaKe FileSystem) and give it a file system type of `ext4` and use it on the primary partition `/dev/sda1`.*

### Making the swap disk and activating it


Allocate the partition to be used for a swap drive. 

```
mkswap /dev/sda2 
```

Now turn the swap disk on for use.
```
swapon /dev/sda2
```

### Mounting the primary partition and installing

Mount the primary partition into the `mnt` (mount) directory.

```
mount /dev/sda1 /mnt
```

Now install to the mnt directory. 

```
pacstrap -i /mnt base base-devel
```

1. Select All
2. Select All
3. type `y` to confirm

Watch the text fly as the system installs!

*Summary: Pac is the arch package manager. More on this later. We're basically telling it to bootstrap the new partition and file system with the base install.*

### Log on to the newly created machine

First generate a file system tab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

Now log in
```
arch-chroot /mnt
```

You've logged into the new machine. You can confirm this because you will have a new, even more ugly command prompt that before. `[root@archiso /]#` that is no longer colored.

### Generate a new locale

```
locale-gen
```

Set up your timezone. Replace `Mexico/General` by tab autocompleting the correct files for your region.
```
rm /etc/localtime
ln -s /usr/share/zoneinfo/Mexico/General /etc/localtime
hwclock --systohc --utc
```

Set up machine `hostame`. You can replace `ArchLinux` with whatever you want.

```
echo ArchLinux > /etc/hostname
```

Edit the `hosts` file

```
nano /etc/hosts
```

replace `localhost` under the hostname column with your `hostname`.

save with `ctrl + x`, then `y` then return to quit nano. You are now back at the prompt (on the bottom of the screen).

#### Install the network manager

We want to make sure that we will have internet once we reboot into the new machine, so we will need to install `networkmanager`.

```
pacman -S networkmanager
```

type `y` and then enter to proceed

Now enable the network manager

```
systemctl enable NetworkManager
```

Generate a new image of the system

```
mkinitcpio -p linux
```

#### Create a user password and install grub

```
passwd
```

and enter your root password twice.

Install The grub bootloader and os prober

```
pacman -S grub os-prober
```

`y` and confirm installation

Now install the grub on `sda` (not 1 or 2 just sda) make sure that target is set correctly.

```
grub-install --target=i386-pc --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

Now unmount `/dev/sda1`

```
umount /dev/sda1
```

Now exit back to original `root@archlinuxiso` red root. and then reboot

```
exit
reboot
```

Now back at the original arch install screen select power off.

In virtual box unmount the disk on the partition.

Now start the machine again.

You should now see a comand prompt for `ArchLinux login`!!!

Login with root and the password that you set!

## Logging into the new machine

### Creating a new non root user account

Watch spaces on the following line when assigning groups. user name must be lowercase, no spaces.
```
useradd -m -g users -G wheel,storage,power -s /bin/bash <USER NAME>

passwd <USER NAME>
```

edit the sudo file for your user

```
EDITOR=nano visudo
```

uncomment the line for allowing memebers of the group `wheel` to execute any command.

save and quit nano with `ctrl + x`, `y`, enter.

### install zsh as well as bash completion

```
pacman -S zsh bash-completion
```

### Log out of the system 

```
exit
```

and now log back in with the new user that you just created!

### Install Git

```
sudo pacman -S git
```
This is the test right here! 

1. Does the machine have internet?
2. can you sudo?

The base install is done and you should be up and running now!

Make sure to take a snap shot at this phase with virtual box. Just in case anything goes wrong in the next phases.

## Installing Gnome Desktop

```
git clone https://github.com/erikdubois/archgnome

cd archgnome/installation/
```

you can see a list of software we will be installing by running:
```
ls -aux
```

we will install each number package (selecting the right one) in order by number.

First we will start off with a set of mirrors to install from

```
./020-install-fastest-arch-mirrors-v1.sh

sudo pacman -Syu
```

Install VB guest additions

```
./030-install-xorg-virtualbox.sh
```

select `2` for `virtual-box-guest-modules-arch`

```
./040-install-packer-for-aur-make-build-v1.sh

./050-install-gnome-core-v1.sh

./100-install-core-software-v1.sh
```

you can skip 
```
./110-install-printers-v1.sh
```

then install sound and network
```
./120-install-sound-v1.sh

./130-install-network-v1.sh

```

**You can skip** `200-install-extra-software.sh`. This just installs spotify and sublime and some other shit we can add later if you really want it.

```
./200-install-extra-software.sh
```

Now install themes and icons. Then install other software specific to arch distro. 

Then install Samba for live shared network folders.

```
./300-install-themes-icons-cursors-conky-v1.sh

./400-install-software-distro-specific-v1.sh

```

This is optional to import Erik's personal settings. 

```
./500-install-samba-v1.sh
./600-install-personal-settings-v3.sh
./610-install-personal-settings-keyboard-shortcuts-v3.sh
```

Now reboot the machine!

```
sudo reboot
```

Gnome should now be loading!!!!

### Customizing the appearance of Gnome

With all of the fancy packages that took forever to download, we can now start tweaking the theme to look like we want it. 

1. Exit the reminders thing and open `tweaktool` configure your theme with the 3 options listed under `appearance`.


## Installing ZSH and Oh My ZSH!!


Run zsh once and create a default `.zshrc`

```
zsh
```

find the path of the shell

```
chsh -l
```

then change the shell for your user

```
chsh -s <full/path/to/zsh>
```

### Install Oh My ZSH

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### Install custom puddletown theme


`cd` into the folder that will hold your user configurations. Mine is located in `~/Config/`

```
take ~/Config

git clone https://github.com/PuddletownDesign/ZSH

rm ~/.zshrc

cp ~/Config/ZSH/.zshrc ~/.zshrc

cp ~/Config/ZSH/puddletown.zsh-theme ~/.oh-my-zsh/themes/puddletown.zsh-theme
```

## Fixing resolution for Retina Displays

## Fixing the screen flicker (if it's problem on your system)

## Installing and configuring Armorak

## Installing Atom text editor

## Installing Wine

### Installing Photoshop in Wine

### Installing Ableton Live in Wine