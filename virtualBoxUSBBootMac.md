# 如何在Mac的VirtualBox上从USB启动（摔

反正我现在还没搞定。。。。摔

先看这位大哥的[文章]（https://www.linkedin.com/pulse/virtualbox-booting-from-usb-mac-cem-arslan）

Step 1 : Finding USB device number

```
$ diskutil list
```

Step 2 : Unmouting USB device

```
$ diskutil unmountdisk /dev/disk2
```

Step 3 : Preparing user

> The tricky part is that the VirtualBox process can only read/write files owned by the current user you are logged with. However Mac OS X, had put root as owner. With this default, you won’t be able to import the disk file that we are going to create. So the solution is too change the permission of the device.

```
$ sudo chown your-username-here /dev/disk2
```

Step 4 : Preparing VMDK

```
$ VBoxManage internalcommands createrawvmdk -filename /Users/your-username-here/Documents/usbdrive.vmdk -rawdisk /dev/disk2
RAW host disk access VMDK file /Users/leseb/Documents/usbdrive.vmdk created successfully
```

Step 5 : Unmount again

```
$ diskutil unmountdisk /dev/disk2
```

大哥提醒：
> **Do not unplug your USB because it will use that one!**

[问答社区]（https://apple.stackexchange.com/questions/192292/how-to-do-raw-device-access-with-virtualbox）

Start Virtual Box Manager with Root Privileges

```
sudo /Applications/VirtualBox.app/Contents/MacOS/VirtualBox
```

To confirm permissions of devices, you can run ls -l /dev/disk*:

1. Add New Machine, provide name: /dev/disk2, type: linux, version: arch linux (64-bit)
2. Give it some memory (whatever you can)
3. Select existing drive using VMDK created above. (IMPORTANT ensure the drive has not been remounted, otherwise you will get VERR_RESOURCE_BUSY, NS_ERROR_FAILURE)


下面有两行小字
> Notes
> tip: get yourself iterm 2
> This is super useful for Macbooks when installing Arch Linux, because the default ISO does not load the wlan card drivers.

不得不承认Arch真是麻烦，按教程搞了几个命令以后决定放弃。我都下载光盘了你告诉我安装还需要联网？？
还是德系好。
