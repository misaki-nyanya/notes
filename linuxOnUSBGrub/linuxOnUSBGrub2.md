# （重复劳动）安装Puppy到U盘--GRUB2 （2）

事情大条了。

作为一个完美主义者，还是想[参照wiki](https://wiki.archlinux.org/index.php/Multiboot_USB_drive)做一个多重启动。

首先发现昨天的量产猜想是错的，那个sdb6似乎是之前电脑上的残留分区。。。。啊啊啊我的数据。不管了。

然后

```
$ gdisk /dev/sdX
```

这个命令之前用的不对，不能加分区号。应该直接用设备名。于是先用gdisk分区后再创建了多重启动表。
果然Arch社区这群爱折腾的人的文章可重复性很高。。。

然后激动人心的安装grub的时刻

发现我电脑上的grub没有EFI的支持😂

ubuntu有两个包grub-efi grub-pc分别支持EFI和MBR。然而不能共存Orz。

网上搜了一下如果换了一个装，怕是会改不回来。嘛，也就是说宿主机会再也启动不起来咯。
原本还想把grub源码编译了以后把文件烤过去。发现也是没用的。

于是最终极的武器。。。。在virtualbox上装个EFI的linux。。。

但是发现vitualbox识别不了u盘？？？

[参考 Virtualbox 启用USB 设备支持](https://blog.csdn.net/u012719256/article/details/73603938)
两个要件：
1. 装扩展包
2. 把vbox加入宿主机的用户组（这里的安全性风险还没确认，总之先用了。）

```
查看当前用户名：
sharl@sharl-laptop:~$ whoami
sharl

查看vbox 所在的组：
sharl@sharl-laptop:~$ cat /etc/group | grep vbox
vboxusers:x:125:sharl

将当前用户加入vbox组：
usermod -a -G vboxusers sharl

重启。
```

很棒，看到设备了。

接下来搞grub.
