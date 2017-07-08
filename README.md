# 系统备份

在使用重装系统作为恢复系统的过程中，存在操作过程繁琐费时和数据丢失的问题。结合改
进之后的全量备份和差量备份方案，为解决上述问题提供了一种可能。

> 本文所述的系统备份思路适用于类`unix`系统，相关示例以`gentoo`为平台运行。

## 备份动机

### 安装系统过程

- [安装系统过程及其分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%AE%89%E8%A3%85%E7%B3%BB%E7%BB%9F%E8%BF%87%E7%A8%8B)

### 备份原理

-   一旦系统安装配置完备之后，此时的系统快照较之默认系统快照(`stage-3`)相比既贴
    合设备硬件，也符合用户的应用场景。同时也保留了当下各种系统文件和用户数据。

## 全量备份

### 备份思路

#### 备份时间

全量备份时间最好为系统功能确定，初次安装并配置完成系统之后。此时通过tar命令打 包压缩当前系统全量快照。

#### 备份环境

`/proc`和`/sys`及其`/dev`会在系统启动的时候开始填充数据，但这些数据并不需要备份
，可以通过`chroot`来避免。

- [备份环境(chroot)介绍和步骤wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%A4%87%E4%BB%BD%E7%8E%AF%E5%A2%83)

#### 实施备份

```
tar --exclude-from=$exclude_file --xattrs -czpvf $backupfile /
```

- [全量备份操作分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%AE%9E%E6%96%BD%E5%A4%87%E4%BB%BD)

- [查看全量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-backup)

#### 排除文件

文件系统中一些目录是内存映像，而一些是系统的运行时产生的缓存文件。这些目录都是不
应纳入到备份快照中来的。这些目录tar命令可以通过--exclude-from指定排除文件。

- [排除文件介绍wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E6%8E%92%E9%99%A4%E6%96%87%E4%BB%B6)

- [查看备份排除文件](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-exclude)

### 还原思路

```
tar --xattrs -xpf $backupfile
```

- [全量还原应用场景wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E8%BF%98%E5%8E%9F%E6%80%9D%E8%B7%AF)

### 同步到其他的块设备

- [同步操作分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%90%8C%E6%AD%A5%E5%88%B0%E5%85%B6%E4%BB%96%E7%9A%84%E5%9D%97%E8%AE%BE%E5%A4%87)

- [查看同步其他的块设备脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/rsync-full-system-backup)

## 差量备份

### 备份思路

#### 备份时间

全量备份完成之后，在系统投入正式使用或者存放用户数据的时候，可以开始考虑进行差量
备份。

> 既然是差量备份，此时的数据应该相对稳定，没有大量的数据增加和减少，安装系统及其
> 全量备份期间有大量的数据写入，这个时候不是很适合。

#### 技术细节

差量备份使用的是通用块层的lvm来实现。

- [lvm快照原理分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82)

#### 实施备份

差量备份相对全量备份开销较小，可以多次少量进行备份，以达到其能够灵活选择还原的时
节点。为了能够自动化完成定期备份，我们可以使用crontab进行备份。

- [实施备份步骤wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%AE%9E%E6%96%BD%E5%A4%87%E4%BB%BD-1)

- [查看差量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/difference-backup)

- [查看每周备份任务表](https://raw.githubusercontent.com/crux-wild/system-backup/master/difference-backup-crontab)

### 还原思路

当系统损害，但是`lvconvert`仍然可以使用的情况下，应该优先考虑是差量还原。其实就是
特殊时间节点中快照的旧数据，回滚到当前目录。

- [还原思路wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E8%BF%98%E5%8E%9F%E6%80%9D%E8%B7%AF-1)

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
