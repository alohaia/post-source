---
title: Linux 配置记录
comments: true
mathjax: false
date: 2021-04-06 21:49:19
tags:
    - Linux
categories:
    - Learning
        - Computer
---

记录 Linux 使用过程中遇到的坑、软件和开发环境的安装和配置等。

<!-- more -->

## 踩坑记录

### 用 Manjaro 安装镜像分区

Manjaro 安装程序分区有时候会有问题，建议先用 `fdisk` 分好区并格式化。

### 把 HOME 目录下的目录名改为英文

```bash
sudo pacman -S xdg-user-dirs-gtk && export LANG=en_US && xdg-user-dirs-gtk-update
export LANG=zh_CN.UTF-8
```

### Arch 系安装 deb 包

```bash
# 安装 debtap
yay -S debtap
sudo debtap -u

# 生成 arch 包
debtap <deb 包>

# 安装生成的包
sudo pacman -U <arch 包>
```

## Linux 安装及基本配置

### Linux 安装

**镜像及启动盘制作**（[清华镜像站](https://mirrors.tuna.tsinghua.edu.cn/)）
- Arch Linux：https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/
- Manjaro Linux
    - 官方版：https://mirrors.tuna.tsinghua.edu.cn/osdn/storage/g/m/ma/manjaro/
    - 社区版：https://mirrors.tuna.tsinghua.edu.cn/osdn/storage/g/m/ma/manjaro-community/
- 刻录软件：
    - Windows：rufus
    - Linux：`dd` 命令，示例：`dd if=~/Downloads/archlinux-2021.04.01-x86_64.iso of=/dev/sdc1`
      if 指定镜像文件，of 指定要用的启动盘，一般为 U 盘，可使用 `lsblk -f` 和 `sudo fdisk -l` 帮助确定。

启动盘制作完成后即可重启电脑，通过 BIOS 从启动盘启动计算机。

**后续步骤**

Arch
1. 参考 ArchWiki 的 [Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
2. 4-1-2021，官方发布[消息](https://archlinux.org/news/installation-medium-with-installer/)，可通过“a guided installer”安装 Arch Linux。镜像自带该安装器，只需在镜像系统中使用 `archinstall` 即可启动菜单式安装。~~爷的青春结束了~~

Manjaro：自带图形安装界面

**分区参考**

```xxx
NAME        FSTYPE FSVER LABEL  UUID                                 FSAVAIL FSUSE% MOUNTPOINT
nvme0n1
├─nvme0n1p1 vfat   FAT32        3833-B9DC                             510.5M     0% /boot/efi
├─nvme0n1p2 swap   1            8759088c-fdb3-4719-952d-a00cd91fa61b                [SWAP]
├─nvme0n1p3 xfs                 b78adcf1-81a8-4cb0-83dc-3813e68067b8   62.3G    38% /
├─nvme0n1p4 xfs                 7fd8a9e3-8e5e-4d7f-99c3-a6a1498bd192  146.7G    27% /home
└─nvme0n1p5 ntfs         Shared CC00E4A900E49C28                       17.8G    89% /home/aloha/Shared
```

我的是 Win10 + Linux 双系统，`nvme0n1p5` 为共享分区。

**文件系统格式推荐**

- btrfs：Fedora 的默认 文件系统，支持 copy-on-write（COW），被认为是 Linux 的下一代文件系统，参考
    - 知乎-如何看待 Fedora 33 将默认使用 btrfs 文件系统？-@醉卧沙场的回答
    - Fedora Wiki-Btrfs by Default
- xfs：RHEL（Red Hat Enterprise Linux）7 和 8 等著名的商用发行版默认采用的文件系统，我在使用的就是 xfs，主要以因为可以使用 `xfsdump` 方便备份

**使用 `xfsdump` 自动备份的脚本**

```bash
#!/bin/bash

day=$(date +%w)
# day=0

RET=''
#@1 str
#@2 color
Hi(){
    RET="\033[1;$2;40m$1\033[0m"
}

back_mnt_rel='_tmp/.sys-backup-mnt'
# mount point of backup device
back_mnt=`pwd -P`/${back_mnt_rel}
# dir to store backup files
back_dir=${back_mnt}
# dir to store old backup files
back_old_dir=${back_mnt}/.old
# device where to store dump files
back_dev="6b1bd96b-5abc-4625-aa44-7de685a79752"

echo "Backup Information:"
Hi ${back_dev} 33 && echo -e "\tUsing Device:\t\t${RET}"
Hi ${back_mnt} 33 && echo -e "\tMount Point:\t\t${RET}"
Hi ${back_dir} 33 && echo -e "\tBackup Directory:\t${RET}"
Hi ${day}      33 && echo -e "\tBackup Level:\t\t${RET}"
read -p "Continue? y(es)/[n(o)]: " confirm
case ${confirm} in
    [yY][eE][sS]|[yY])
        echo -e "\033[1;34;40mBackup Starting...\033[0m"
        ;;
    [nN][oO]|[nN])
        echo -e "\033[1;31;40mAborted.\033[0m"
        exit 1
        ;;
    *)
        echo -e "\033[1;31;40mInvalid input.\033[0m"
        exit 2
        ;;
esac


mkdir -p ${back_mnt}
mount --uuid ${back_dev} ${back_mnt}
mkdir -p ${back_dir}


if [[ $day -eq 1 || $day -eq 2 || $day -eq 3 || $day -eq 4 || $day -eq 5 || $day -eq 6 ]];then
    echo -e "\033[1;34;40m[Level ${day}]Dumping...\033[0m"
    # /boot/efi
    echo -e "\033[1;32;40mtar -upf ${back_dir}/efi-full-`date +%F`.tgz /boot/efi\033[0m"
    tar -upf ${back_dir}/efi-full-`date +%F`.tgz /boot/efi
    # /
    echo -e "\033[1;32;40mxfsdump -e -l ${day} -f ${back_dir}/root-inc${day}-`date +%F` -L root-inc${day}-`date +%F` -M nvme0n1p3 /\033[0m"
    xfsdump -e -l ${day} -f ${back_dir}/root-inc${day}-`date +%F` -L root-inc${day}-`date +%F` -M nvme0n1p3 /
    # /home
    echo -e "\033[1;32;40mxfsdump -e -l ${day} -f ${back_dir}/root-inc${day}-`date +%F` -L home-inc${day}-`date +%F` -M nvme0n1p4 /home\033[0m"
    xfsdump -e -l ${day} -f ${back_dir}/home-inc${day}-`date +%F` -L home-inc${day}-`date +%F` -M nvme0n1p4 /home
# 7
elif [ $day -eq 0 ];then
    echo -e "\033[1;34;40m[Level 0(full)]Dumping...\033[0m"
    echo -e "\033[1;34;40mCleaning up old Dumps..\033[0m"
    echo -e "\033[1;32;40mrm ${back_old_dir}/*\033[0m"
    rm ${back_old_dir}/*
    echo -e "\033[1;32;40mmv ${back_dir}/*inc* ${back_dir}/*full* ${back_old_dir}/\033[0m"
    mv ${back_dir}/*inc* ${back_dir}/*full* ${back_old_dir}/
    # /boot/efi
    echo -e "\033[1;32;40mtar -cpf ${back_dir}/efi-full-`date +%F`.tgz /boot/efi\033[0m"
    tar -cpf ${back_dir}/efi-full-`date +%F`.tgz /boot/efi
    # /
    echo -e "\033[1;32;40mxfsdump -e -f ${back_dir}/root-full-`date +%F` -L root-full-`date +%F` -M nvme0n1p3 /\033[0m"
    xfsdump -e -f ${back_dir}/root-full-`date +%F` -L root-full-`date +%F` -M nvme0n1p3 /
    # /home
    echo -e "\033[1;32;40mxfsdump -e -f ${back_dir}/home-full-`date +%F` -L home-full-`date +%F` -M nvme0n1p4 /home\033[0m"
    xfsdump -e -f ${back_dir}/home-full-`date +%F` -L home-full-`date +%F` -M nvme0n1p4 /home
else
    echo 'Actually, this is IMPOSSIBLE.'
fi

umount ${back_mnt}
rmdir -p ${back_mnt_rel}
```

## pacman 换源

1. 方法 1：sudo pacman-mirrors -i -c China -m rank
2. 方法 2：在 /etc/pacman.d/mirrorlist 开头加上：

```xxx /etc/pacman.d/mirrorlist
## Country : China
Server = https://mirrors.sjtug.sjtu.edu.cn/manjarostable/$repo/$arch
```
然后同步本地数据库：`sudo pacman -Syy`

启用 `archlinuxcn` 源（optional，==不建议 manjaro 用户启用==）：

在 `/etc/pacman.conf` 末尾加上：

```xxx /etc/pacman.conf
[archlinuxcn]
#SigLevel = Optional TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

然后执行命令：`sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring`

## 软件和开发环境

### WPS

```bash
paru -S wps-office-cn ttf-wps-fonts wps-office-mui-zh-cn
```

安装后出现提示

```xxx
# ATTENTION: When you shut down wps, the wpsoffice process may still exist.
# You can do 'sudo chmod -x /usr/lib/office6/wpsoffice' to fix it. But
# this might bring you problem with signing in.
```

### 字体

Nerd 字体：`nerd-fonts-jetbrains-mono` 或 `nerd-fonts-fira-code`

中文字体：`ttf-sarasa-gothic` 或 `adobe-source-han-serif-otc-fonts` 和 `adobe-source-han-sans-otc-fonts`

### fcitx 输入法

```bash
sudo pacman -S fcitx5-im fcitx5-material-color fcitx5-rime librime
```

设置环境变量：

```bash ~/.pam_environment
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=\@im=fcitx
SDL_IM_MODULE DEFAULT=fcitx
```

Rime 配置目录：`~/.local/share/fcitx5/rime/`

参见

- ArchWiki：
    - fcitx5：https://wiki.archlinux.org/index.php/Fcitx5_(简体中文)
    - rime：https://wiki.archlinux.org/index.php/Rime_(简体中文)
- Rime Github 主页：https://github.com/rime/home


### MariaDB 和 lua-mysql

- lua-mysql

```bash
sudo proxychains -q luarocks install luasql-mysql MYSQL_INCDIR=/usr/include/mysql
```

- MariaDB

```bash
sudo pacman -S mariadb mariadb-clients mariadb-libs
sudo mv /var/lib/mysql /tmp
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
sudo systemctl enable mariadb
sudo systemctl restart mariadb
mysql
```



