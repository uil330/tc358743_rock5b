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
- 第二行将 `tc358743.dtbo` 放入 `/boot/dtbo/`

### 2. 配置 extlinux 以启用覆盖
编辑 `/boot/extlinux/extlinux.conf`，将设备树覆盖加入 `fdtoverlays`。例如：

```conf
label kernel-$(uname -r)
    ...
    fdtoverlays  ...  /boot/dtbo/tc358743.dtbo
    ...
```

保存后重启，使覆盖生效。

### 3.  验证与排查
- 验证设备树覆盖是否生效：
  - `dmesg` 中应出现与 tc358743 相关的绑定与 I2C 初始化日志
  - `media-ctl -p -d /dev/media0` 下可见对应设备节点
- 验证视频设备：
  - 执行 `gst-launch-1.0 v4l2src device=/dev/video0   do-timestamp=true  ! 'video/x-raw,format=NV12,width=1920,height=1080,framerate=30/1' !    queue   ! mpph264enc bps=500000 rc-mode=cbr ! h264parse config-interval=-1  ! mpegtsmux ! udpsink host=x.x.x.x port=xxxx` 

### 4. 注意事项

- Rock5B理论上能使用4Line,但是实际上只能用2Line.不知道是硬件限制还是个体问题,需要后续排查
- 理论上支持UYVY,但是实际上只能用NV12,需要后续查看是Rockchip的架构特性还是带宽问题

