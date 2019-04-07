# （重复劳动）安装Puppy到U盘--GRUB2

最近Design School解体， 从一堆尸体中翻出一个4G U盘， 很可耻地带了回来打算日常用。
以及最近教一个学部生Linux的知识时忽然重燃往日激情hhhhh，发现Puppy更新了，于是...

首先刷个grub:

 ### [安装grub2到U盘和移动硬盘的方法](https://www.nenew.net/install-grub2-u-disk-hard-disk.html)

```
grub2引导很强大，今天奶牛来说说如何安装grub2到U盘或者移动硬盘上。
首先进入终端
sudo su
fdisk -l
看看自己的u盘或者移动硬盘的设备名称。
然后在mnt下挂载我们的U盘或者移动硬盘设备。奶牛这里以sdb设备为例讲解。
sudo su
cd /mnt
mkdir u
mount /dev/sdb ./u （这里是把设备挂载到一个目录下，如果大家分过区就挂载第一个分区就可以了）
sudo grub-install –root-directory=/mnt/u /dev/sdb
运行到这里就安装完成了。大家可以重启看效果了，看看u盘启动是不是有grub2了~
```

立刻发现了问题， grub2的命令应该是

```
$ sudo grub-install –-boot-directory=/mnt/u /dev/sdb
```

运行了一下， 爆炸， 说 
1. File system fat doesn't support embedding.
2. Embedding is not posible, GRUB can only be installed in this setup by using blocklists, However , blocklists are unreliable and their use is doscouraged.
3. error: will not proceed with blocklists.

莫非是FAT32的锅？！

来格个盘
```
$ sudo umount /dev/sdb6
$ sudo mkfs.ntfs /dev/sdb6
```
其实我还是很不甘心放弃FAT32的。毕竟兼容性最强。
顺便提一下mkfs.fat和mkfs.vfat，似乎是fat的一个进化版，支持长文件名。其他没有区别。


然后TMD发现
# 依旧错误！！！！

尼玛，问题不是文件系统。
重新格回FAT32，并发现了一个和我需求一致的[网站](https://www.pendrivelinux.com/install-grub2-on-usb-from-ubuntu-linux/)

于是加了选项--force --removable， 这次通过了。
但是看到--removable选项注有Only available on EFI，有点心虚，于是放弃了这个选项。

尝试在实验室的淘汰机上启动了一下，**失敗**，提示插入设备。

忽然发觉，这个u盘对应/dev/sdb6和/dev/sdg1两个块设备！！

而Gpart不管怎样都搞不掉，看似没关联但是地址上确有一大块重合！！

在网上查了一下，可能是传说中的量产。

另外发现Gpart可以操作的boot flag,两个块设备都没有标上，只有/dev/sdg1有一个lba的flag。

/dev/sdb6这货跟我存有linux的硬盘挂在同一个/dev/sdb上，我不能把boot的flag给它。
所以我选择把boot flag给/dev/sdg1，并在上面重新做grub2。

再次实验，看到Missing Operating System,眼泪留下来💧
弄得太晚了，下次继续。

[另附一个参考](https://wiki.archlinux.org/index.php/Multiboot_USB_drive)
[再来一个参考](https://www.pendrivelinux.com/boot-multiple-iso-from-usb-via-grub2-using-linux/)
[鸟哥](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/168.html)
[Grub2 CN](https://wiki.gentoo.org/wiki/GRUB2/zh-cn)
[Grub2 EN](https://www.dedoimedo.com/computers/grub-2.html)
