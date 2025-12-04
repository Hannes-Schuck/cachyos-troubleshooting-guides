# Fix broken bootloader

I am **dual booting** CachyOS and Windows (on two separate SSDs). After **updating my BIOS** (MSI MAG B850 Tomahawk MAX WIFI) **both boot partitions** showed the Windows bootloader and both instantly booted Windows skipping systemd-boot.

IMPORTANT: Please read the entire guide before changing something in your system! Maybe my solution will not work or you realize after the first half that this is not the problem you were searching for. At the end I will describe my hypothesis why my bootloader probably was destroyed.

## Fixing the problem

### 1. Install a live USB
The first step was creating a live USB and booting it. It usually takes a while to copy the image into RAM so I do not recommend to restart often.

### 2. Mount drive of the inaccessible system
Open the terminal and use `lsblk` to list all available partitions. In my case there are two NVME drives called `nvme1n1` (Windows drive with 4 partitions) and `nvme0n1` (CachyOS with 2 partitions). On my CachyOS partition I have the EFI partition (Size: ~2GB) and the partition I use for my system (Size: ~2TB, depends on your drive).

After this step you have to mount the two partitions. If you **encrypted** your system drive on system setup you first have to decrypt it in order to use it.

In my case the name of the partition is `nvme1n1p2` so I use following command to decrypt it:

`>sudo cryptsetup luksOpen /dev/nvme1n1p2 myssd`

This command will prompt you to enter your encryption password.

After typing your password the drive should be decrypted and you can now mount it with:

`>sudo mount -o subvol=@ /dev/mapper/myssd /mnt`

`subvol=@` is important if you use BTRFS as your filesystem. I do not know how other filesystems need to be mounted here.
(Also: for this guide you don't need to mount other BTRFS folders like @home but if you want to do more with your data, you should probably use this [Archwiki entry](https://wiki.archlinux.org/title/Chroot#Running_on_Btrfs))

(if your drive is not encrypted it should be enough to use `sudo mount -o subvol=@ /dev/nvme1n1p2 /mnt` which I did not try, therefore I cannot confirm that this works)

After mounting the system partition you need to mount your EFI partition. (use `mkdir /mnt/efi` if there is an error that the path does not exist)

`>sudo mount /dev/nvme1n1p1 /mnt/efi`

### 3. Use chroot to access a system shell

The following paragraph constists of commands that should not be sequentially run, because it is more like trying random things and I have to admit that I am not sure which exact commands fixed the problem.

Now it is the time to use `arch-chroot` to enter the system:

`>sudo arch-chroot /mnt`

Now you should be in a shell of the system. First of all I made a backup (I moved the files to cleanup the partition, I do not know if this is a very safe idea when the system reboots between the next steps!) of my EFI files:

`>>sudo mv /efi/* /efibackup` 

`>>bootctl --path=/mnt/boot install`

After this the bootloader worked again (but is named `UEFI OS` now) but does not show any entries.

I used my backup in `/efibackup` and copies the entries back to the `/boot` path with **one exception**: the Windows entry.

`>>sudo rm -rf /efibackup/EFI/Microsoft/` (where I added the Windows bootloader)

`>>sudo cp -r /efibackup/* /boot/`

Restart the system to see the change in the BIOS. After the reboot you can also see the change with `efibootmgr`. Without restarting the system `efibootmgr` will **not** show any difference!

### 4. My mistake

Creating the Windows systemd-boot entry.

I wanted to add Windows to my systemd-boot so I don't need to change the boot order in the BIOS each time I want to switch.


The guides I found online didn't work for me yet so I asked the last possible friendly helper ChatGPT (I know, I am going to hell for this) which just told me to copy the Windows boot loader into my Linux EFI partition: **which worked!** and probably was the only reason I ran into this problem at the end of the day... (I was aware that the Windows bootloader would not be updated that way, but I ran into this problem earlier)

Currently I am searching for a better way to deal with the Windows boot entry.
