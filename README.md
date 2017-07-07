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

## 全量备份

## 差量备份

