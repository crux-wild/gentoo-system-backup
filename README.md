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

- [备份环境(chroot)介绍和步骤wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%A4%87%E4%BB%BD%E7%8E%AF%E5%A2%83)

#### 实施备份

- [全量备份操作及其分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%AE%9E%E6%96%BD%E5%A4%87%E4%BB%BD)

- [查看全量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-backup)

#### 排除文件

- [排除文件介绍wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E6%8E%92%E9%99%A4%E6%96%87%E4%BB%B6)

- [查看备份排除文件](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-exclude)

### 还原思路

- [还原操作及其分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E8%BF%98%E5%8E%9F%E6%80%9D%E8%B7%AF)

### 同步到其他的块设备

- [同步操作分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%90%8C%E6%AD%A5%E5%88%B0%E5%85%B6%E4%BB%96%E7%9A%84%E5%9D%97%E8%AE%BE%E5%A4%87)

- [查看同步其他的块设备脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/rsync-full-system-backup)

## 差量备份

### 备份思路

#### 备份时间

- [备份时间分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%A4%87%E4%BB%BD%E6%97%B6%E9%97%B4-1)

#### 技术细节

- [lvm快照原理分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82)

#### 实施备份

- [实施备份步骤wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%AE%9E%E6%96%BD%E5%A4%87%E4%BB%BD-1)

- [查看差量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/difference-backup)

- [查看每周备份任务表](https://raw.githubusercontent.com/crux-wild/system-backup/master/difference-backup-crontab)

### 还原思路

- [还原思路wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E8%BF%98%E5%8E%9F%E6%80%9D%E8%B7%AF-1)

### 同步到其他块设备

- [同步到其他设备思路wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%90%8C%E6%AD%A5%E5%88%B0%E5%85%B6%E4%BB%96%E5%9D%97%E8%AE%BE%E5%A4%87)

## 相关链接

<https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/About><br/>
<https://en.wikipedia.org/wiki/Unix_filesystem><br/>
<https://wiki.archlinux.org/index.php/Full_system_backup_with_tar><br/>
<https://www.howtoforge.com/linux_lvm_snapshots><br/>
<https://wiki.gentoo.org/wiki/LVM><br/>
