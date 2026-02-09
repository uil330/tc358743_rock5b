# tc358743_rock5b

本仓库提供在 Rock 5B 等基于 Radxa/内核的发行版上，为 TC358743 HDMI-to-CSI-2 芯片启用设备树覆盖与内核模块的参考流程。内容涵盖：生成并启用 dtbo、配置 extlinux、编译并安装 tc358743.ko。

## 环境准备
- 已安装系统编译需要的工具

## 快速开始
以下步骤按先生成设备树覆盖文件、启用覆盖，再构建并安装内核模块的顺序进行。命令均在目标设备上执行。

### 1. 生成 dtbo 并放置到 /boot
在项目根目录下执行：

```bash
dtc -@ -I dts -O dtb -o tc358743.dtbo tc358743.dts
sudo cp ./tc358743-cam0.dtbo /boot/dtbo/
```

说明：
- 第一行将 `tc358743.dts` 编译为覆盖文件 `tc358743.dtbo`
- 第二行将 `tc358743-cam0.dtbo` 放入 `/boot/dtbo/`（如仅生成了 `tc358743.dtbo`，请将该文件复制到 `/boot/dtbo/`，并在后续 `fdtoverlays` 使用对应文件名）

### 2. 配置 extlinux 以启用覆盖
编辑 `/boot/extlinux/extlinux.conf`，将设备树覆盖加入 `fdtoverlays`。例如：

```conf
label kernel-$(uname -r)
    ...
    fdtoverlays  ...  /boot/dtbo/tc358743.dtbo
    ...
```

保存后重启，使覆盖生效。

### 3. 下载内核并安装 TC358743 驱动模块
拉取内核源码并使用当前运行内核的配置：

```bash
git clone https://github.com/radxa/kernel.git
# 切换到kernel对应版本的分支
cd kernel
zcat /proc/config.gz > .config
make menuconfig
# 在菜单中将 tc358743 设置为 M（模块）
cp /usr/src/linux-headers-$(uname -r)/Module.symvers .
make modules_prepare
make -j4 M=drivers/media/i2c/ modules
sudo cp drivers/media/i2c/tc358743.ko /lib/modules/$(uname -r)/kernel/drivers/media/i2c/
sudo depmod -a
```

## 验证与排查
- 验证设备树覆盖是否生效：
  - `dmesg` 中应出现与 tc358743 相关的绑定与 I2C 初始化日志
  - `/sys/bus/i2c/devices/` 下可见对应设备节点
- 验证视频设备：
  - 安装 `v4l2-ctl` 后查看设备与格式

```bash
media-ctl -p -d /dev/media0 // 查看设备树是否正常
v4l2-ctl --list-devices // 查看视频设备是否正常出现
v4l2-ctl -d /dev/video0 --all
```

