---
slug: Docker-apparmor
title: 解决Linux kernel >=5.10.13无法使用Docker
author: Stitch-Zhang
author_title: A iddle programer
author_url: https://github.com/endiliey
author_image_url: https://avatars1.githubusercontent.com/u/61350804?s=460&v=4
tags: [linux, docker,apparmor]
---
## 原因

高版本Linux内核中Apparmor机制会使docker无法使用，具体表现为无法启动容器

其提示如下所示
``` 
···
caused: process_linux.go:495: container init caused: apply apparmor profile: apparmor failed to apply profile: write /proc/self/attr/exec: invalid argument: unknown.
···
``` 
<!--truncate-->

## 相关信息

> AppArmor （“Application Armor”，意为“应用盔甲”） 是一个Linux内核安全模块，允许系统管理员通过每个程序的配置文件限制程序的功能。如它的帮> 助页面所说，“AppArmor 是一个对内核的增强工具，将程序限制在一个有限的资源集合中。AppArmor 独特的安全模型将对访问属性的控制绑定到程序而非户。 -- 摘自维基百科


讲人话`AppArmor`就是现代`SELinux`


[Docker上有关Apparmor的issue](https://github.com/docker/for-linux/issues/1199)


[Apparmor详细介绍](https://zh.wikipedia.org/wiki/AppArmor)

# 解决方案
在内核启动参数中添加 
```
apparmor=1 lsm=lockdown,yama,apparmor,bpf
``` 

## 具体方法
### 临时解决 
**在`引导文件`中添加参数**


如果是 `GRUB`引导，编辑 `/boot/grub/grub.cfg` 文件
找到当前`Linux引导条目`

如下为我当前设备的启动项及参数，忽略部分信息
```
menuentry 'Manjaro Linux' --class manjaro --class gnu-linux --class gnu --class os $menuentry_id_option 
        ···
        linux   /boot/vmlinuz-linux-lily root=UUID=51fc2a33-f113-4b83-ac7d-ab117582e1a9 ro  quiet apparmor=1 security=apparmor resume=UUID=afaccb40-b>
        ···
}
```

修改为

```
menuentry 'Manjaro Linux' --class manjaro --class gnu-linux --class gnu --class os $menuentry_id_option 
        ···
        linux   /boot/vmlinuz-linux-lily root=UUID=51fc2a33-f113-4b83-ac7d-ab117582e1a9 ro  quiet apparmor=1 lsm=lockdown,yama,apparmor,bpf security=apparmor resume=UUID=afaccb40-b>
        ···
}
```

:::info
使用当前解决方案，在下次内核升级时，`GRUB` 配置文件`grub.cfg` 将会重新生成

导致之前配置的失效
:::
### 永久解决

在 `grub.cfg`文件生成器生成的`配置文件` `/etc/default/grub`中修改

原始如下

```
···
GRUB_CMDLINE_LINUX_DEFAULT="quiet apparmor=1 security=apparmor resume=UUID=afaccb40-b1f8-4df9-bf86-49dde48815b0 udev.log_priority=3"
···
```

修改为

```
···
GRUB_CMDLINE_LINUX_DEFAULT="quiet apparmor=1 lsm=lockdown,yama,apparmor,bpf security=apparmor resume=UUID=afaccb40-b1f8-4df9-bf86-49dde48815b0 udev.log_priority=3"
···
```

:::tip
使用当前方案时需要重新生成`GRUB`配置文件`grub.cfg`，修改后每次`grub-mkconfig`将自动添加内核引导参数 
`apparmor=1 lsm=lockdown,yama,apparmor,bpf`
:::

执行重新生成`grub`配置
```shell
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
