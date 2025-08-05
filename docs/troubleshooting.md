# 故障排除指南

本指南帮助您诊断和解决 pico-rv32ima 项目中的常见问题。

## 🔧 构建问题

### CMake 配置失败

**症状**: CMake 无法找到 Pico SDK
```
CMake Error: Could not find Pico SDK
```

**解决方案**:
1. 确认 Pico SDK 已正确安装
2. 设置环境变量:
   ```bash
   export PICO_SDK_PATH=/path/to/pico-sdk
   ```
3. 或在 CMake 命令中指定:
   ```bash
   cmake -DPICO_SDK_PATH=/path/to/pico-sdk ..
   ```

### 编译错误

**症状**: 找不到头文件或库
```
fatal error: 'pico/stdlib.h' file not found
```

**解决方案**:
1. 检查 Pico SDK 版本 (建议 1.5.0+)
2. 确认子模块已更新:
   ```bash
   git submodule update --init --recursive
   ```
3. 清理构建目录重新构建:
   ```bash
   rm -rf build && mkdir build && cd build && cmake .. && make
   ```

### 工具链问题

**症状**: 找不到 ARM GCC 编译器
```
arm-none-eabi-gcc: command not found
```

**解决方案**:
1. 安装 ARM GCC 工具链:
   ```bash
   # Ubuntu/Debian
   sudo apt install gcc-arm-none-eabi
   
   # macOS
   brew install arm-none-eabi-gcc
   ```
2. 检查 PATH 环境变量
3. 使用 VS Code Pico 扩展自动安装

## 🔌 硬件问题

### PSRAM 初始化失败

**症状**: 控制台显示 "Error initializing PSRAM!"

**可能原因和解决方案**:

1. **连接问题**
   - 检查 SPI 引脚连接 (CLK, MISO, MOSI, CS)
   - 确认焊接质量，避免虚焊
   - 使用万用表测试连通性

2. **电源问题**
   - 确认 PSRAM 芯片供电正常 (3.3V)
   - 检查电源纹波，使用去耦电容

3. **时序问题**
   - 降低 SPI 时钟频率测试:
     ```c
     spi_set_baudrate(PSRAM_SPI_INST, 1000 * 1000 * 10); // 10MHz
     ```
   - 检查信号完整性

4. **芯片兼容性**
   - 确认使用支持的 PSRAM 芯片 (LY68L6400, ESP-PSRAM64H)
   - 检查芯片 ID:
     ```c
     psram_read_id(PSRAM_SPI_PIN_S1, chipId);
     if (chipId[1] != PSRAM_KGD) return -1;
     ```

### SD 卡初始化失败

**症状**: 控制台显示 "Error initializing SD!"

**解决方案**:

1. **SD 卡格式**
   - 使用 FAT16 或 FAT32 格式
   - 避免使用 exFAT 或 NTFS
   - 重新格式化 SD 卡

2. **连接检查**
   - 验证 SPI 引脚连接
   - 检查 CS 引脚是否正确控制
   - 确认 SD 卡插入到位

3. **兼容性问题**
   - 尝试不同品牌的 SD 卡
   - 使用较小容量的卡 (≤32GB)
   - 避免高速卡可能的兼容性问题

### VGA 显示问题

**症状**: VGA 显示器无信号或显示异常

**解决方案**:

1. **同步信号检查**
   - 确认 HSYNC (GPIO17) 和 VSYNC (GPIO16) 连接
   - 使用示波器检查同步信号时序

2. **RGB 信号**
   - **必须使用 330Ω 电阻** 连接 RGB 信号
   - 检查 R (GPIO18), G (GPIO19), B (GPIO20) 连接
   - 确认接地连接良好

3. **显示器兼容性**
   - 尝试不同的 VGA 显示器
   - 检查显示器支持的分辨率 (640x480@60Hz)

### PS/2 键盘无响应

**症状**: 键盘输入无反应

**解决方案**:

1. **电平转换**
   - **必须使用 3.3V 电平转换器**
   - PS/2 键盘是 5V 设备，直接连接会损坏 Pico
   - 检查电平转换器工作状态

2. **连接检查**
   - DATA (GPIO26) 和 CLK (GPIO27) 连接
   - 确认键盘供电 (5V)
   - 检查接地连接

3. **键盘兼容性**
   - 尝试不同的 PS/2 键盘
   - 避免使用 USB 转 PS/2 适配器

## 💻 软件问题

### 系统无法启动

**症状**: 按下 BOOTSEL 后系统无响应

**诊断步骤**:

1. **检查控制台输出**
   - 连接 USB 查看 CDC 控制台
   - 或连接 UART 查看串口输出
   - 查看启动日志定位问题

2. **文件检查**
   - 确认 SD 卡包含 `IMAGE` 和 `ROOTFS` 文件
   - 检查文件大小是否正常
   - 重新下载或构建镜像文件

3. **配置检查**
   - 验证 `rv32_config.h` 配置
   - 确认内存大小设置正确
   - 检查引脚配置匹配硬件

### Linux 内核崩溃

**症状**: 内核启动过程中出现 panic 或异常

**常见错误和解决方案**:

1. **内存不足**
   ```
   Out of memory: Kill process
   ```
   - 增加 PSRAM 容量 (使用两片芯片)
   - 优化内核配置减少内存使用

2. **根文件系统错误**
   ```
   VFS: Cannot open root device
   ```
   - 检查 `ROOTFS` 文件完整性
   - 确认内核命令行参数正确

3. **设备树问题**
   - 检查设备树配置
   - 确认硬件描述匹配实际连接

### 性能问题

**症状**: 系统运行缓慢或响应迟钝

**优化方案**:

1. **超频设置**
   - 确认 CPU 频率设置正确
   - 检查电压设置是否足够
   - 监控温度避免过热

2. **缓存优化**
   - 检查缓存命中率
   - 优化内存访问模式
   - 调整缓存大小

3. **I/O 优化**
   - 提高 SPI 时钟频率
   - 使用 DMA 传输
   - 减少不必要的磁盘访问

## 🔍 调试技巧

### 使用控制台调试

1. **启用详细输出**
   ```c
   #define DEBUG_VERBOSE 1
   ```

2. **添加调试信息**
   ```c
   console_printf("Debug: addr=0x%08x, val=0x%08x\n", addr, val);
   ```

3. **监控系统状态**
   ```c
   console_printf("Cycles: %lld, PC: 0x%08x\n", core.cyclel, core.pc);
   ```

### 使用 SWD 调试器

1. **连接调试器**
   - SWDIO: GPIO2 (或专用调试引脚)
   - SWCLK: GPIO3 (或专用调试引脚)

2. **配置 VS Code**
   - 安装 Cortex-Debug 扩展
   - 配置 launch.json
   - 设置断点调试

### 逻辑分析仪

对于硬件问题，使用逻辑分析仪分析：
- SPI 时序
- 同步信号
- 数据完整性

## 📊 性能监控

### 内存使用监控
```bash
# 在 Linux 控制台中
free -m
cat /proc/meminfo
```

### CPU 使用监控
```bash
top
cat /proc/loadavg
```

### I/O 性能测试
```bash
# 磁盘读写测试
dd if=/dev/zero of=/tmp/test bs=1M count=10
dd if=/tmp/test of=/dev/null bs=1M
```

## 🆘 获取帮助

### 收集诊断信息

在报告问题时，请提供：

1. **硬件信息**
   - Pico 型号 (RP2040/RP2350)
   - PSRAM 芯片型号和数量
   - SD 卡规格
   - 连接方式

2. **软件信息**
   - 固件版本
   - 构建环境
   - 配置文件内容

3. **错误信息**
   - 完整的控制台输出
   - 错误发生的具体步骤
   - 环境条件 (温度、电源等)

### 社区支持

- **GitHub Issues**: 报告 bug 和功能请求
- **讨论区**: 技术交流和经验分享
- **文档**: 查阅最新的技术文档

### 常用诊断命令

```bash
# 检查系统状态
dmesg | tail
cat /proc/version
cat /proc/cpuinfo

# 检查硬件
lsmod
cat /proc/devices
cat /proc/interrupts

# 性能分析
coremark  # 运行基准测试
```

记住：大多数问题都与硬件连接或配置有关。仔细检查连接和配置通常能解决大部分问题。
