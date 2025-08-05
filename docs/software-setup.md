# 软件设置指南

本指南将帮助您构建和配置 pico-rv32ima 项目的软件环境。

## 🛠️ 开发环境要求

### 必需工具
- **CMake** (3.13 或更高版本)
- **ARM GCC 工具链** (arm-none-eabi-gcc)
- **Git** (用于克隆仓库和子模块)
- **Make** (用于构建 Linux 镜像)
- **Python 3** (Pico SDK 依赖)

### 推荐开发环境
- **Visual Studio Code** 配合 Pico 扩展
- **Linux** 或 **WSL** (用于构建 Linux 镜像)

## 📦 安装依赖

### Ubuntu/Debian
```bash
sudo apt update
sudo apt install cmake gcc-arm-none-eabi build-essential git python3 python3-pip
```

### macOS (使用 Homebrew)
```bash
brew install cmake arm-none-eabi-gcc git python3
```

### Windows
1. 安装 [Visual Studio Code](https://code.visualstudio.com/)
2. 安装 [Raspberry Pi Pico 扩展](https://marketplace.visualstudio.com/items?itemName=raspberry-pi.raspberry-pi-pico)
3. 扩展会自动安装所需的工具链

## 🚀 项目构建

### 1. 克隆项目
```bash
git clone https://github.com/cxgreat2014/pico-rv32ima.git
cd pico-rv32ima
git submodule update --init --recursive
```

### 2. 设置 Pico SDK
```bash
# 如果没有安装 Pico SDK，需要先安装
git clone https://github.com/raspberrypi/pico-sdk.git
export PICO_SDK_PATH=/path/to/pico-sdk
```

### 3. 构建项目
```bash
mkdir build
cd build
cmake ..
make -j4
```

### 4. 生成 UF2 文件
构建完成后，会在 `build/pico-rv32ima/` 目录下生成 `pico-rv32ima.uf2` 文件。

## 🐧 Linux 镜像构建

### 使用预构建镜像 (推荐)
1. 下载预构建镜像：
   ```bash
   git clone https://github.com/tvlad1234/pico-linux-images.git
   ```

2. 将 `IMAGE` 和 `ROOTFS` 文件复制到 SD 卡根目录

### 自行构建镜像
```bash
cd linux
make
```

构建过程包括：
1. 克隆 buildroot 源码
2. 应用自定义配置
3. 构建交叉编译工具链
4. 编译 Linux 内核
5. 构建根文件系统
6. 编译附加应用程序

**注意**: 完整构建可能需要几小时时间和大量磁盘空间。

## ⚙️ 配置选项

### 主要配置文件
所有配置都在 `pico-rv32ima/rv32_config.h` 文件中：

#### CPU 频率配置
```c
// RP2040 (Pico) 配置
#define RP2040_CPU_FREQ 400000  // 400MHz
#define RP2040_OVERVOLT VREG_VOLTAGE_MAX

// RP2350 (Pico 2) 配置  
#define RP2350_CPU_FREQ 512000  // 512MHz
#define RP2350_OVERVOLT VREG_VOLTAGE_1_60
```

#### 内存配置
```c
// RAM 大小 (MB)
#define EMULATOR_RAM_MB 8

// 是否使用两片 PSRAM
#define PSRAM_TWO_CHIPS 0
```

#### 控制台配置
```c
// 启用 USB CDC 控制台
#define CONSOLE_CDC 1

// 启用 VGA 控制台  
#define CONSOLE_VGA 1

// 启用 UART 控制台
#define CONSOLE_UART 0
```

#### 文件名配置
```c
// Linux 内核文件名
#define KERNEL_FILENAME "IMAGE"

// 根文件系统文件名
#define BLK_FILENAME "ROOTFS"
```

### 引脚配置
可以修改各种外设的引脚分配：

```c
// SD 卡 SPI 引脚
#define SD_SPI_PIN_CK 2
#define SD_SPI_PIN_TX 3
#define SD_SPI_PIN_RX 4
#define SD_SPI_PIN_CS 0

// PSRAM SPI 引脚
#define PSRAM_SPI_PIN_CK 10
#define PSRAM_SPI_PIN_TX 11
#define PSRAM_SPI_PIN_RX 12
#define PSRAM_SPI_PIN_S1 13
#define PSRAM_SPI_PIN_S2 14

// VGA 引脚
#define VGA_VSYNC_PIN 16
#define VGA_HSYNC_PIN 17
#define VGA_R_PIN 18

// PS/2 键盘引脚
#define PS2_PIN_DATA 26
#define PS2_PIN_CK 27
```

## 📱 烧录和运行

### 1. 烧录固件
1. 按住 Pico 的 BOOTSEL 按钮
2. 连接 USB 线到电脑
3. 将 `pico-rv32ima.uf2` 文件复制到出现的 RPI-RP2 磁盘
4. Pico 会自动重启并运行固件

### 2. 准备 SD 卡
1. 格式化 SD 卡为 FAT16 或 FAT32
2. 将 Linux 镜像文件复制到 SD 卡根目录：
   - `IMAGE` (Linux 内核)
   - `ROOTFS` (根文件系统)

### 3. 启动系统
1. 插入 SD 卡到读卡器
2. 连接所有硬件组件
3. 按下 BOOTSEL 按钮启动 Linux
4. 等待约 30 秒完成启动

## 🔧 开发工具配置

### Visual Studio Code
1. 安装 Raspberry Pi Pico 扩展
2. 打开项目文件夹
3. 使用 Ctrl+Shift+P 打开命令面板
4. 选择 "Raspberry Pi Pico: Configure Project"
5. 选择 SDK 路径和构建配置

### CMake 配置
项目使用标准的 CMake 配置：

```cmake
# 主 CMakeLists.txt
cmake_minimum_required(VERSION 3.13)
include(pico_sdk_import.cmake)
project(pico-rv32ima)
pico_sdk_init()
add_subdirectory(pico-rv32ima)
```

### 调试配置
可以使用 SWD 调试器进行调试：
1. 连接 SWD 调试器到 Pico 的调试引脚
2. 在 VS Code 中配置调试器
3. 设置断点进行调试

## 🧪 测试和验证

### 基本功能测试
1. 检查控制台输出是否正常
2. 验证 PSRAM 初始化成功
3. 确认 SD 卡读取正常
4. 测试 Linux 启动过程

### 性能测试
启动后可以运行内置的基准测试：
```bash
# 在 Linux 控制台中运行
coremark
```

### 应用程序测试
测试内置的解释器和编译器：
```bash
# C4 编译器
c4 hello.c

# JavaScript 引擎
duktape fizzbuzz.js

# Lua 解释器
lua
```

## 🐛 常见问题

### 构建失败
- 检查工具链是否正确安装
- 确认 Pico SDK 路径设置正确
- 检查 CMake 版本是否满足要求

### 烧录失败
- 确认按住 BOOTSEL 按钮
- 检查 USB 连接
- 尝试不同的 USB 端口

### 启动失败
- 检查 SD 卡格式和文件
- 验证硬件连接
- 查看控制台错误信息

更多问题解决方案请参考 [故障排除指南](troubleshooting.md)。

## 📚 下一步

- 阅读 [项目架构](architecture.md) 了解技术细节
- 查看 [开发指南](development.md) 学习如何贡献代码
- 参考 [API 文档](api-reference.md) 了解接口详情
