# 系统备份

在使用重装系统作为恢复系统的过程中，存在操作过程繁琐费时和数据丢失的问题。结合改
进之后的全量备份和差量备份方案，为解决上述问题提供了一种可能。

> 本文所述的系统备份思路适用于类`unix`系统，相关示例以`gentoo`为平台运行。

## 备份动机

### 安装系统

[安装系统过程及其分析wiki↵](https://github.com/crux-wild/system-backup/wiki#%E5%AE%89%E8%A3%85%E7%B3%BB%E7%BB%9F%E8%BF%87%E7%A8%8B)

## 全量备份

### 备份思路

#### 备份时间

[备份时间介绍wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%A4%87%E4%BB%BD%E6%97%B6%E9%97%B4)

#### 备份环境

[备份环境(chroot)介绍和步骤wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%A4%87%E4%BB%BD%E7%8E%AF%E5%A2%83)

#### 实施备份

[全量备份操作及其分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%AE%9E%E6%96%BD%E5%A4%87%E4%BB%BD)

[查看全量备份脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-backup)

#### 排除文件

[排除文件介绍wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E6%8E%92%E9%99%A4%E6%96%87%E4%BB%B6)

[查看备份排除文件](https://raw.githubusercontent.com/crux-wild/system-backup/master/full-system-exclude)

### 还原思路

[还原操作及其分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E8%BF%98%E5%8E%9F%E6%80%9D%E8%B7%AF)

### 同步到其他的块设备

[同步操作分析wiki↵](https://github.com/crux-wild/system-backup/wiki/%E7%B3%BB%E7%BB%9F%E5%A4%87%E4%BB%BD#%E5%90%8C%E6%AD%A5%E5%88%B0%E5%85%B6%E4%BB%96%E7%9A%84%E5%9D%97%E8%AE%BE%E5%A4%87)

[查看同步其他的块设备脚本](https://raw.githubusercontent.com/crux-wild/system-backup/master/rsync-full-system-backup)

## 差量备份

### 备份时间

全量备份完成之后，在系统投入正式使用或者存放用户数据的时候，可以开始考虑进行差量
备份。

> 既然是差量备份，此时的数据应该相对稳定，没有大量的数据增加和减少，安装系统及其
> 全量备份期间有大量的数据写入，这个时候不是很适合。

#### 技术细节

差量备份使用的是通用块层的`lvm`来实现。

> `lvm`通过更有效的写时复制策略来完成。快照创建的时候所在分区目录是对应逻辑卷生
> 成对应数据节点的硬链接。如果数据没有改变，快照分区只会保留数据的一个硬链接。一
> 旦修改对应数据，`lvm`会自动复制旧数据，并将快照指向旧数据。修改之后的数据暂时
> 保留在原先目录中。

通过`lvcreate`创建分区，此时初始化已经完成，对应硬链接也填充完毕。

```
lvcreate --size 100M --snapshot --name snapshot01 /dev/mapper/vg-lv
```

通过`lvconvert`回滚数据到数据初始化时刻。

```
lvconvert --merge /dev/vg0/snap01
```

通过`lvremove`删除对应快照，提交新数据修改。

```
lvremove /dev/vg0/snap01
```

#### 实施备份

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
