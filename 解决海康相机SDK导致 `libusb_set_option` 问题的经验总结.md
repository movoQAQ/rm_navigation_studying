# 解决海康相机SDK导致 \`libusb\_set\_option\` 问题的经验总结

## 问题描述

当编译某些 ROS 代码时，出现类似错误：

```Plain Text
/usr/bin/ld: …/…/lib/libpcl_io.so.1.8.0: undefined reference to `libusb_set_option’
```

## 原因分析

安装海康相机 SDK 后，系统原本的 `libusb` 依赖会被重新链接到海康 SDK 指定的路径。

由于 `libusb` 是许多外设程序的重要依赖库，这种更改可能导致其他设备驱动或程序（例如使用系统默认 `libusb` 的程序）出现冲突，从而触发上述错误。

## 解决方法

为了解决 `libusb` 路径冲突问题，需要手动调整环境变量 `LD_LIBRARY_PATH`，让系统程序优先使用系统默认的 `libusb` 版本。

以下是详细步骤：

### 1\. 检查环境变量配置

打开终端，检查当前 `LD_LIBRARY_PATH` 配置：

```bash
echo $LD_LIBRARY_PATH
```

## 解决海康 SDK 与 libusb 路径冲突问题

在某些情况下，安装海康 SDK 后，系统中的 `libusb` 可能与海康 SDK 中的版本发生冲突，导致其他依赖 `libusb` 的程序出现问题。以下是解决此问题的步骤：

### 1\. 检查冲突源

如果输出中包含海康 SDK 的路径，例如 `/path/to/hikvision/sdk/lib`，这可能是冲突的来源。

### 2\. 修改 `~/.bashrc`

为确保优先使用系统默认的 `libusb`，可以通过调整 `~/.bashrc` 配置文件来重新设置 `LD_LIBRARY_PATH`：

在文件末尾添加以下一行：

```bash
export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
```

#### 说明

`/usr/lib/x86_64-linux-gnu` 是系统默认的 `libusb` 路径，根据你的系统架构可能有所不同。如果有多个默认路径，请确保所有可能的系统库路径都在 `LD_LIBRARY_PATH` 变量中优先级靠前。

### 3\. 应用更改

保存并关闭文件后，在终端执行以下命令使修改生效：

```bash
source ~/.bashrc
```

### 4\. 检查是否解决问题

重新运行导致错误的程序，确认问题是否解决。如果仍有问题，可以检查程序实际加载的 `libusb` 版本：

```bash
ldd /path/to/your/executable | grep libusb
```

### 确保输出路径正确

确保输出中显示的路径为 `/usr/lib/x86_64-linux-gnu/libusb.so` 或系统默认路径，而不是海康 SDK 的路径。

## 进一步优化

### 特定程序设置环境变量

如果你只希望某些程序使用默认的 `libusb`，可以在运行这些程序时临时设置 `LD_LIBRARY_PATH`：

```bash
LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH ./your_program
```

## 总结

安装海康 SDK 后，路径冲突问题可能影响到其他依赖 `libusb` 的程序。通过调整 `LD_LIBRARY_PATH` 优先使用系统默认库路径，可以有效解决问题。
