# Setup Windows bootloader entry (with systemd-boot)

I am dual booting CachyOS and Windows from two different NVME SSDs so the systemd boot manager does not automatically recognize my Windows 11 installation when I install Windows first and then Linux.

The first time I created a Windows bootloader entry, I just copied the entire `Microsoft` folder of the Windows boot partition into my Linux boot partition `/boot/EFI/` (like in this [YouTube tutorial](https://www.youtube.com/watch?v=SqA7loOXPZw)). This worked until I updated my BIOS (which resulted in [this](fix-broken-bootloader.md) problem). Also, when something in the Windows bootloader changes the boot might not work anymore because I copied the folder instead of referencing the boot partiation that Windows uses and knows by default.

I chose this method because the direct reference like in this [forum post](https://forum.endeavouros.com/t/tutorial-add-a-systemd-boot-loader-menu-entry-for-a-windows-installation-using-a-separate-esp-partition/37431/24) did not work for me.

After fixing my broken boot loader, I tried to use the reference method, by following this [YouTube tutorial](https://www.youtube.com/watch?v=ySFV5igQv44). Here, instead like in the [forum post i referenced ealier](https://forum.endeavouros.com/t/tutorial-add-a-systemd-boot-loader-menu-entry-for-a-windows-installation-using-a-separate-esp-partition/37431/24) there is no `windows.nsh` file, because the drive is directly referenced in the `/boot/loader/entries/windows.conf` file, which I created. The rest of the file was created by following the forum post and the YouTube tutorial (at least the `HD1b:` should be adjusted).


```  
title   Windows 11
efi     /EFI/shellx64.efi
options -nointerrupt -noconsolein -noconsoleout HD1b:EFI\Microsoft\Boot\Bootmgfw.efi
```

This is not a guide that describes how to create a Windows entry but rather tries to show the problems that can occur. When using the `windows.nsh` method I probably just used the wrong path where I saved the file or the wrong file path in the `.conf` file.