# 系统备份

> 本文所述的系统备份思路适用于类`unix`系统，相关示例以`gentoo`为平台运行。

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

-   不同的块设备通过`VFS`对操作系统提高统一的接口，用户根据块设备的具体情况和应
    用常见，配置适合自己硬件分区方案(例:`BIOS`或者`GPT`)和通用块层(例:`lvm`)，以
    及文件系统(例: `ext4`或者`ntfs`)。很多针对块设备的优化都有特殊应有场景，过度
    使用高级特性，会增加系统操作难度和增加不必要性能消耗。

-   不同硬件设备需要不同的设备驱动，内核通过编译链接或者模块的方式加载驱动。自定
    义匹配最小化的驱动，可以避免无效驱动的带来性能消耗，提高硬件驱动效率。


## 全量备份

## 差量备份

