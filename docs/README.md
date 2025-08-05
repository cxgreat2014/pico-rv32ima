# pico-rv32ima 项目文档

欢迎来到 pico-rv32ima 项目文档！本项目在 Raspberry Pi Pico (RP2040) 和 Pico 2 (RP2350) 上运行 Linux 系统。

## 📚 文档目录

### 快速开始
- [硬件设置指南](hardware-setup.md) - 硬件连接和配置
- [软件设置指南](software-setup.md) - 软件构建和配置

### 深入了解
- [项目架构](architecture.md) - 技术架构和实现细节
- [开发指南](development.md) - 代码开发和贡献指南
- [API参考](api-reference.md) - 核心API和接口文档

### 支持
- [故障排除](troubleshooting.md) - 常见问题和解决方案

## 🚀 项目概述

pico-rv32ima 是一个在 Raspberry Pi Pico 微控制器上运行 Linux 的项目。它使用 RISC-V 模拟器核心，通过 SPI PSRAM 芯片作为系统内存，从 SD 卡加载 Linux 内核和根文件系统。

### 主要特性

- **RISC-V 架构**: 32位 RISC-V 处理器模拟，支持 IMA 扩展
- **无 MMU Linux**: 运行 no-MMU 版本的 Linux 内核
- **大容量内存**: 支持 8MB 或 16MB PSRAM 作为系统内存
- **存储支持**: SD 卡作为块设备存储
- **多种显示**: VGA 文本显示和 PS/2 键盘支持
- **高性能缓存**: 128KB 二路组相联缓存
- **多种控制台**: USB CDC、UART 和 VGA 控制台

### 硬件要求

- Raspberry Pi Pico 或 Pico 2 (或其他 RP2040/RP2350 开发板)
- SD 卡 (FAT16/FAT32 格式)
- 1-2 个 8MB (64Mbit) SPI PSRAM 芯片 (LY68L6400 或 ESP-PSRAM64H)
- (可选) VGA 显示器和 PS/2 键盘的相关电路

### 软件组件

- **RISC-V 模拟器**: 基于 CNLohr 的 mini-rv32ima 核心
- **Linux 内核**: 定制的 no-MMU RISC-V Linux
- **根文件系统**: 包含各种 Linux 工具和解释器
- **应用程序**: C4 编译器、Duktape JS 引擎、Lua 解释器、CoreMark 基准测试

## 🎯 快速开始

1. **准备硬件**: 按照 [硬件设置指南](hardware-setup.md) 连接所有组件
2. **构建软件**: 按照 [软件设置指南](software-setup.md) 编译项目
3. **准备 SD 卡**: 下载预构建的镜像或自行构建 Linux 镜像
4. **运行系统**: 按下 BOOTSEL 按钮启动 Linux

## 📖 更多信息

- **GitHub 仓库**: [pico-rv32ima](https://github.com/cxgreat2014/pico-rv32ima)
- **预构建镜像**: [pico-linux-images](https://github.com/tvlad1234/pico-linux-images)
- **原始项目**: [mini-rv32ima](https://github.com/cnlohr/mini-rv32ima)

## ⚠️ 重要提示

本项目会对微控制器进行超频和超压操作！使用时请自行承担风险。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！请参阅 [开发指南](development.md) 了解如何参与项目开发。

## 📄 许可证

本项目遵循相应的开源许可证，详见 [LICENSE](../LICENSE) 文件。
