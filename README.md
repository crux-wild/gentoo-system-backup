# 系统备份

> 本文所述的系统备份思路适用于类`unix`系统，相关示例以`gentoo`为平台运行。

在使用重装系统作为恢复系统的技术方案的过程中，存在操作过程费时繁琐和数据丢失的问
题。结合使用改进之后的全量备份和差量备份方案，为解决上述问题提供了一种可能。

## 备份动机

### 安装系统过程

-   磁盘或者其他块设备设置分区

    ```
    fdisk /dev/sda && ...
    ```

-   配置通用块层(可选)

    ```
    pvcreate /dev/sda[b]1 && ...
    ```

-   创建文件系统

    ```
    mkfs.ext2 /dev/sda2 && ...
    ```

-   复制系统文件快照

    ```
    tar xvjpf stage3-*.tar.bz2 --xattrs --numeric-owner && ...
    ```

-   获取最新更新

    ```
    emerge --sync && ...
    ```

-   自定义配置文件

    ```
    vim /etc/locale.gen && locale-gen && env-update && ...
    ```

-  安装必要软件

    ```
    emerge --ask sys-kernel/gentoo-sources && ...
    ```

-   自定义内核和`ramdisk`

    ```
    cd /usr/src/linux && make menuconfig && genkernel && ...
    ```

-   安装引导

    ```
    grub-install /dev/sda && grub-mkconfig -o /boot/grub/grub.cfg && ...
    ```

**过程分析:**

-   根据硬件环境和用户需求配置硬件分区方案(例:`BIOS`或者`GPT`)，配置通用块层(例:
    `lvm`)和文件系统(例: `ext4`或者`ntfs`)。

-   很多针对块设备的优化(例：`RAID`)都有特殊应有场景，过度使用高级特性，会增加系
    统操作难度和增加不必要性能消耗。是否使用需要根据所使用系统用途来确定。

-   不同硬件设备需要对应的设备驱动，内核通过编译链接或者模块的方式加载驱动。自定
    义匹配最小化的驱动，可以避免无效驱动的带来性能消耗，提高硬件驱动效率。

-   操作系统为了更加通用，很多选择以配置文件提供给用户选择(例：国际化和多时区)。

-   最小化的软件安装，可以避免过多内存占用和提高系统稳定性。(例：服务器系统都是
    通过`ssh`进行维护，通常不需要一个图形化，`Xorg`不仅会增加性能的消耗，也会降
    低系统稳定性)。

> 安装系统的过程虽然繁琐耗时，但是为了更好的贴合硬件和用户需求，上述核心步骤是难
> 以省略的。

#### 重新安装分析

-   **重新安装优点:**

    当运行中系统遭遇误操作等情况时，旧系统难以启动或者是错误难以排查的时候。重新
    安装系统可以避免陷在旧系统的复杂问题中。

-   **重新安装的缺点：**

    重装通常需要格式化块设备，以达到重置环境的作用。这个时候旧系统的数据将会丢失
    ,而恢复配置完备的环境和各类用户数据还需要较多时间。

#### 备份原理

-   一旦系统安装配置完备之后，此时的系统快照较之默认系统快照(`stage-3`)相比既贴
    合设备硬件，也符合用户的应用场景。同时也保留了当下各种系统文件和用户数据。

## 全量备份

### 备份思路

全量备份时间最好为系统功能确定，初次安装并配置完成系统之后。此时通过`tar`命令打
包压缩当前系统全量快照。

#### 备份环境

`/proc`和`/sys`及其`/dev`会在系统启动的时候开始填充数据，但这些数据并不需要备份
，可以通过`chroot`来避免。

```
mount /dev/<your-partition-or-driver>

cd /mnt

chroot . /bin/bash
```

> 通过`livecd`启动之后，CD中文件系统作为根目录被挂载。相对CD文件系统的`/proc`和`
> /sys`及其`/dev`三个目录，在`livecd`启动之后被内核填充。其后挂载系统分区到`/mnt
> `之后，`/mnt/proc`和`/mnt/sys`及其`/mnt/dev`其中的内容没有被填充。

#### 实施备份

```
tar --exclude-from=$exclude_file --xattrs -czpvf $backupfile /
```

> 安装操作系统实质是通过计算把数据写入到块设备完成持久化。而`unix`系统过`VFS`规
> 范把不同的块设备抽象为文件，通过格式化和挂在成为目录使用。通过相同的系统调用(
> 例:`read`)进行操作。`tar`的备份就是基于文件级别的备份。`unix`系统中的七种类型:
> `d`(目录文件)，`l`(符号链接)，`s`(套接字)，`b`(设备文件)，`p`(管道文件)，普通
> 文件遵循`POSIX`规范。而文件中字符和二进制数据也是遵循相应的编码规范(例:`utf-8`
> )。基于文件的备份可以有更好的移植性。

[查看全量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-backup)

#### 排除文件

文件系统中一些目录是内存映像，而一些是系统的运行时产生的缓存文件。这些目录都是不
应纳入到备份快照中来的。这些目录`tar`命令可以通过`--exclude-from`指定排除文件。
具体内容如下:

-   `/dev`包含`I/O`设备(`/dev/sda0`)或者是虚拟设备(`/dev/null`)的设备文件。通常
    是设备驱动程序在文件系统的一个表征，用户程序可以通过`VFS`标准的系统读写设备
    。加载之后的驱动程序是存在于内存中的动态数据，不需要备份。

-   `/proc`查看进程信息的特殊文件系统，保存的是内核中动态的数据。

-   `/tmp`是用来保存临时的目录，里面的数据每次重启之后都会被清空。

-   `/run`一个临时文件系统用来存储运行产生的动态数据。

-   `/mnt`一个空目录，是做临时挂载点。

-   `/media`便携设备(例：`usb`)的默认挂载点。

-   `/lost+found`用来存储`fsck`命令发现没有引用的数据片段。

[查看备份排除文件](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-exclude)

### 还原思路

全量备份由于单次备份和中断时间较长，备份系统的时间节点通常比较靠前，与之对应还原
之后的系统损失的数据也比较多。所以，全量备份还原不是首选。但是，如果情况用差量备
份难以还原(例如磁盘损坏)，这个时候只能通过全量备份还原来恢复。

```
tar --xattrs -xpf $backupfile
```

### 同步到其他的块设备

根据上述分析可以看出，通常还原全量备份的时候，都是块设备发生较大损坏的时候。如果
使用块设备其中的分区作为存储目录，如果块设备损坏较为严重，数据所处的分区本身也很
难确保其完整。这个使用必要使用其他的块设备(例：`便携设备`)做多点备份。

[查看同步其他的块设备](https://raw.githubusercontent.com/crux-wild/system-backup/master/rsync-full-system-backup)

> `rsync`相对于`cp`能够在复制的过程中保证文件属性(例:`owner`)不丢失。

## 差量备份

### 备份思路

#### 技术细节

#### 实施备份

全量备份完成之后，在系统投入正式使用或者存放用户数据的时候，可以开始考虑进行差量
备份。

> 既然是差量备份，此时的数据应该相对稳定，没有大量的数据增加和减少，安装系统及其
> 全量备份期间有大量的数据写入，这个时候不是很适合。

[查看差量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/difference-backup)

差量备份相对全量备份开销较小，可以多次少量进行备份，以达到其能够灵活选择还原的时
节点。为了能够自动化完成定期备份，我们可以使用`crontab`进行备份。

通过下述命令进行编辑：

```
sudo crontab -e
```

[查看每周备份任务表](https://raw.githubusercontent.com/crux-wild/system-backup/master/difference-backup-crontab)

> `run-parts`命令可以目录下的一组可执行文件，具体可以参看`man run-parts`。

### 还原思路

### 同步到其他块设备

差量备份虽然是块设备数据损坏不大的时候使用。但是，通过全量备份时间节点比较靠前的
问题，可以通过多个差量文件尽可能的进行弥补。可以复用全量备份同步脚本，只是对目录
结构进行修改，其中`full-system`和`difference`分别存放全量和差量备份文件。使用
`rsync`进行增量传输。

```
|__ backup/
      |
      |__ full-system/
          |
          |__ full-system<date>.tar.xz
          |
          |__ ...
      |
      |__ difference/
          |
          |__ difference<date>.tar.xz
          |
          |__ ...
```

## 相关链接

<https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/About><br/>
<https://en.wikipedia.org/wiki/Unix_filesystem><br/>
<https://wiki.archlinux.org/index.php/Full_system_backup_with_tar><br/>
<https://www.howtoforge.com/linux_lvm_snapshots><br/>
<https://wiki.gentoo.org/wiki/LVM><br/>
