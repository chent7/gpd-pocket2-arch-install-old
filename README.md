# Arch Linux on the GPD Pocket 2 (WIP).                                                                                                       
                                                                                 (.                    
                                                                                (((.                   
                                                                               (((((                   
            .,,,,,  .,,,,,,     .,,,,,                                        ////((/                  
        .#%%####%#. (%%%###%#*  (%%%###%#/                                   // //////                 
       /%%%#*  .##. (%#    /##. (%(     #%#                                 *//////////                
      .#%(          (%(    ,##, (%/     #%%(         @@                    ****((((((((/               
       #%#          (%%#((#%%(  (%%#(/(#%%%/        %@%@                  *((((((((((((((              
       *#%%#*  .##. (%%####(.   (%%%%%%%%%#                              ((((((     ((((((             
         (#%%##%%#. (%#         (%%%%%%%#*                              (((((((      ((((((            
                                                                       (((((((*      ((((((            
                                                                      ((((((,          ((((((*         
                                                                     ((/                    ((/        
                                                                                               *       
The complete guide to installing Arch and configuring Arch on the GPD Pocket 2, based on all available online resources I can find on the GPD Pocket 2.

It is also relevant to other distros in the configuration parts.

I will do my best to explain everything ;)

## Installation
Download the live boot ISO from Arch Linux and write it to a USB stick.

Boot into the USB by going into the Bios, spam `Del` or hold `Fn + F12`.

~~Rememeber to boot into the `UEFI` partition.~~ (Not too important).

### Rotating the console
Let's start by rotating the console.
```
echo 1 > /sys/class/graphics/fbcon/rotate_all
```

### Internet
Connect via WiFi or plug in your an Ethernet Adaptor.
#### WiFi
Select your network and use dynamic IP
```
wifi-menu
systemctl restart dhcpcd.service
```
#### Checking connection
Wait a good 5-10 seconds, then check your connection by pinging a website.
```
ping archlinux.org
```
### Check your boot mode
You should be in `UEFI`, if not, you will need to boot into the `UEFI` partition on your USB stick.
```
ls /sys/firmware/efi/efivars
```
### Update your clock
```
timedatectl set-ntp true
```
### Partitioning
Check the storage devices you have and partitions on your Pocket 2 by:
```
lsblk
```
You should have a storage device `/dev/mmcblk0`, which is your main drive on the GPD Pocket 2.
#### GUID Partition Table (GPT)
__This is the part where you back up your files if you haven't already!__

We will be repartitioning the drive, __this will delete everything__.

Fire up `gdisk` and select the main drive `/dev/mmcblk0`:
```
gdisk /dev/mmcblk0
```
Use `m` to show available options.

__Keep in mind there will be no changes to the disk until you write them.__

To create a new empty GUID partition table, press `o` followed by `y`.
#### Boot Partition
To create a new partition, press `n` followed by `Enter`.

This will be the first partition, press `1` followed by `Enter`.

We want our partition to start at the default first sector, so just `Enter`.

I usually use 256MB for my boot partition, but you can choose 512MB just to be safe.

`+256M` follow by `Enter`.

This will be an EFI partition type, so `ef00` followed by `Enter`.
#### Swap Partition (Optional)
Swap partition is optional, but I would like the ability to hibernate.

To create a new partition, press `n` followed by `Enter`.

This will be the second partition, press `2` followed by `Enter`.

We want our partition to start at the default first sector, so just `Enter`.

I usually use my RAM size + 2GB for my swap partition, but you can also just have 8GB for the Pocket 2.

`+10G` follow by `Enter`.

This will be a linux swap partition, so `8200` followed by `Enter`.
#### Root Partition
I usually just have a root partition, if you want a home and a root partition see
[Arch Wiki](https://wiki.archlinux.org/index.php/installation_guide).

To create a new partition, press `n` followed by `Enter`.

This will be the third partition, press `3` followed by `Enter`.

We want our partition to start at the default first sector, so just `Enter`.

I will use the rest of the disk for Arch, so just `Enter`.

This will be a x86_64 root partition, so `8304` followed by `Enter`.
#### Write changes
Use `p` to see check your layout, start over if you mess something up.

Use `w` to write changes to your disk, and `y` to confirm.
### Formatting Partitions and Using Swap
Check your partition names:
```
lsblk
```
For the boot partition we will use fat32.
```
mkfs.fat -F32 /dev/mmcblk0p1
```
Now make the swap partition.
```
mkswap /dev/mmcblk0p2
```
For the root partition we will use ext4.
```
mkfs.ext4 /dev/mmcblk0p3
```
### Mounting the partitions
__I will not be encrypting the drives!__

Mount our root partition:
```
mount /dev/mmcblk0p3 /mnt
```
Create a folder for out boot partition:
```
mkdir /mnt/boot
```
Mount the boot partition:
```
mount /dev/mmcblk0p1 /mnt/boot
```
Activate the swap partition:
```
swapon /dev/mmcblk0p2
```
### Installing Arch Linux
To install Arch Linux:
```
pacstrap /mnt base base-devel dialog wpa_supplicant intel-ucode vim
```
#### Why do I need all these packages?
`base`: this is your system, kernel, etc.

`base-devel`: for other system essential stuff such as `sudo`, `pacman`, `systemd`.

`dialog`: __you will need this to connect to WiFi upon reboot into your system__ (I learned it the hard way FYI).

`wpa_supplicant`:__you will need this for WPA networks__.

`intel-ucode`: this is the microcode for intel based CPUs.

`vim`: optional, but makes your life easier later ~~(or harder)~~.
### Fstab
To generate the fstab file:
```
genfstab -U /mnt >> /mnt/etc/fstab
```
You need this so the system will know where to mount your drives.
### Into your new system
To change root into your Arch install:
```
arch-chroot /mnt
```
### Timezone
To select our timezone:
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```
Replace `Region` and `City` for your location.
### Locale
Open up `locale.gen` in vim:
```
vim /etc/locale.gen
```
Uncomment any locales you need, save and quit with `:wq`.

To generate them run:
```
locale-gen
```
### Language
To select your display language, grab one of the locales from earlier eg:`en_US.UTF-8` and:
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```
You generally want `utf8` as your encoding.
### Host Name
This will be the name of your Arch system:
```
echo yourhostname > /etc/hostname
```
Replace `yourhostname` with the host name you want.
### Root Password
To set a password for your root:
```
passwd
```
### Add a new user
It's a bad idea to log in as root, you will want to create a new user.
```
useradd -m -G wheel username
passwd username
```
Replace `username` according with your username.
#### What do these do?
`-m`: creates your home folder

`-G wheel`: optional but recommanded, allows root privillages
#### Creating user folders
It's nice to have a downloads or music folder in your home directory.

Some desktop environments or window managers don't create them, so we can create them manually here by installing `xdg-user-dirs`:
```
pacman -S xdg-user-dirs
xdg-user-dirs-update
```
### Root privillages
You will want to be able to run sudo commands:
```
visudo
```
Uncomment `%wheel ALL= (ALL) ALL` to run sudo commands with a password

Uncomment `%wheel ALL=(ALL) NOPASSWD: ALL` to run sudo without prompting password
### Console fonts
This is a larger font, easier on the eyes :)
```
echo FONT=latarcyrheb-sun32 > /etc/vconsole.conf
```
### Installing bootloader
__Important: Do not set bootloader timeout to 0, otherwise /etc/vconsole.conf settings will not load on the Pocket 2.__

I like to use `GRUB`, but it does not display correctly on the Pocket 2.

If you are dual-booting, I recommand using `systemd boot`.
#### GRUB
__Only install one bootloader, GRUB or Systemd Boot!__

Install the necessary packages:
```
pacman -S grub efibootmgr
```
Install grub:
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
`--bootloader-id` is set to the name that shows up in the Bios.
#### GRUB configuration
Open the configuration file:
```
vim /etc/default/grub
```
Find `GRUB_CMDLINE_LINUX` and add `fbcon=rotate:1`, this will keep the console landscape after reboot.

You can set the timeout by changing the value at `GRUB_TIMEOUT`, but remember not to set it to `0` if you want to keep our changes to `/etc/vconsole.conf`.

Lastly, generate config file with:
```
grub-mkconfig -o /boot/grub/grub.cfg
```
#### Systemd Boot
__Only install one bootloader, GRUB or Systemd Boot!__

Install systemd boot:
```
bootctl install
```
Bios entry will be `Linux Boot Manager`
#### Systemd Boot configuration
##### Create new entry
We will need to manually create a new entry for our system:
```
vim /boot/loader/entries/arch.conf
```
Now type the following into `arch.conf`:
```

```
Save and quit vim.
###### What do these do?

##### Configure bootloader
