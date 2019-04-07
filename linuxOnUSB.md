# （重复劳动）安装Puppy到U盘

最近Design School解体， 从一堆尸体中翻出一个4G U盘， 很可耻地带了回来打算日常用。
以及最近教一个学部生Linux的知识时忽然重燃往日激情hhhhh，发现Puppy更新了，于是...

首先刷个grub:
> ### [安装grub2到U盘和移动硬盘的方法](https://www.nenew.net/install-grub2-u-disk-hard-disk.html)
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

立刻发现了问题， 命令应该是

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
其实我还是很不甘心放弃FAT32的。毕竟兼容性最强

顺便提一下mkfs.fat和mkfs.vfat，似乎是fat的一个进化版，支持长文件名。其他没有区别。
