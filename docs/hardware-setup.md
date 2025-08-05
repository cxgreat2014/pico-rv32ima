# 硬件设置指南

本指南将帮助您正确连接和配置运行 pico-rv32ima 项目所需的硬件组件。

## 📋 硬件清单

### 必需组件
- **Raspberry Pi Pico** 或 **Pico 2** (或其他 RP2040/RP2350 开发板)
- **SD 卡** (建议 4GB 以上，支持 FAT16/FAT32)
- **SPI PSRAM 芯片** (1-2 个)
  - LY68L6400 (8MB/64Mbit)
  - ESP-PSRAM64H (8MB/64Mbit)
  - 其他兼容的 8MB SPI PSRAM

### 可选组件 (VGA 显示)
- **VGA 显示器**
- **PS/2 键盘**
- **3.3V 电平转换器** (用于 PS/2 键盘)
- **电阻** (3 个 330Ω 电阻用于 VGA RGB 信号)

## 🔌 引脚连接

### SD 卡 SPI 接口 (默认配置)

| 功能 | GPIO 引脚 | SD 卡引脚 |
|------|-----------|-----------|
| CLK  | GPIO2     | CLK       |
| MISO | GPIO4     | DAT0      |
| MOSI | GPIO3     | CMD       |
| CS   | GPIO0     | DAT3/CS   |

### PSRAM SPI 接口 (默认配置)

| 功能 | GPIO 引脚 | PSRAM 引脚 |
|------|-----------|------------|
| CLK  | GPIO10    | CLK        |
| MISO | GPIO12    | SO         |
| MOSI | GPIO11    | SI         |
| CS1  | GPIO13    | CS (第一片) |
| CS2  | GPIO14    | CS (第二片，可选) |

### VGA 显示接口 (可选)

| 功能   | GPIO 引脚 | VGA 引脚 | 备注 |
|--------|-----------|----------|------|
| VSYNC  | GPIO16    | 14       | 垂直同步 |
| HSYNC  | GPIO17    | 13       | 水平同步 |
| R      | GPIO18    | 1        | 红色信号 (需要 330Ω 电阻) |
| G      | GPIO19    | 2        | 绿色信号 (需要 330Ω 电阻) |
| B      | GPIO20    | 3        | 蓝色信号 (需要 330Ω 电阻) |
| GND    | GND       | 5,6,7,8,10 | 接地 |

### PS/2 键盘接口 (可选)

| 功能 | GPIO 引脚 | PS/2 引脚 | 备注 |
|------|-----------|-----------|------|
| DATA | GPIO26    | 1         | 需要 3.3V 电平转换 |
| CLK  | GPIO27    | 5         | 需要 3.3V 电平转换 |
| VCC  | 5V        | 4         | 5V 电源 |
| GND  | GND       | 3         | 接地 |

## ⚡ 电源和时钟配置

### 超频设置
项目会自动配置以下超频参数：

**RP2040 (Pico):**
- CPU 频率: 400MHz
- 电压: VREG_VOLTAGE_MAX

**RP2350 (Pico 2):**
- CPU 频率: 512MHz  
- 电压: VREG_VOLTAGE_1_60

### 电源要求
- 建议使用稳定的 5V 电源
- 确保电源能够提供足够的电流 (建议 1A 以上)

## 🔧 硬件配置

### PSRAM 配置

#### 单片 PSRAM (8MB)
```c
#define PSRAM_TWO_CHIPS 0
#define EMULATOR_RAM_MB 8
```

#### 双片 PSRAM (16MB)
```c
#define PSRAM_TWO_CHIPS 1
#define EMULATOR_RAM_MB 16
```

### 控制台配置

可以在 `rv32_config.h` 中启用/禁用不同的控制台：

```c
// 启用 USB CDC 控制台
#define CONSOLE_CDC 1

// 启用 VGA 控制台
#define CONSOLE_VGA 1

// 启用 UART 控制台
#define CONSOLE_UART 0
```

## 🛠️ 组装步骤

### 1. 准备开发板
- 确保 Raspberry Pi Pico 或 Pico 2 正常工作
- 检查所有引脚是否完好

### 2. 连接 PSRAM
- 按照引脚表连接 PSRAM 芯片
- 确保连接稳固，避免接触不良
- 如果使用两片 PSRAM，注意 CS 引脚的区别

### 3. 连接 SD 卡
- 使用 SD 卡适配器或直接焊接
- 确保 SPI 连接正确
- SD 卡需要格式化为 FAT16 或 FAT32

### 4. 连接 VGA 显示 (可选)
- **重要**: RGB 信号线必须通过 330Ω 电阻连接，避免损坏显示器
- 确保同步信号连接正确
- 接地线连接可靠

### 5. 连接 PS/2 键盘 (可选)
- **重要**: 必须使用 3.3V 电平转换器
- PS/2 键盘使用 5V 电源，数据信号需要转换为 3.3V
- 确保电平转换器正确连接

## 🧪 硬件测试

### 基本功能测试
1. 连接 USB 线到电脑
2. 按住 BOOTSEL 按钮，插入 USB
3. 应该看到 RPI-RP2 磁盘出现

### PSRAM 测试
- 启动项目后，控制台会显示 "PSRAM init OK!"
- 如果初始化失败，检查 SPI 连接

### SD 卡测试
- 启动项目后，控制台会显示 "SD init OK!"
- 如果初始化失败，检查 SD 卡格式和连接

### VGA 测试
- 连接 VGA 显示器
- 启动后应该看到文本输出
- 如果没有显示，检查同步信号和 RGB 连接

### PS/2 键盘测试
- 连接键盘后，应该能够在控制台输入
- 如果无响应，检查电平转换器和连接

## ⚠️ 安全注意事项

1. **超压风险**: 项目会对微控制器超压，可能影响器件寿命
2. **VGA 保护**: 必须使用 330Ω 电阻保护显示器
3. **电平转换**: PS/2 键盘必须使用电平转换器
4. **电源稳定**: 确保电源稳定，避免系统不稳定
5. **散热**: 超频运行时注意散热

## 🔍 故障排除

常见硬件问题请参考 [故障排除指南](troubleshooting.md)。

## 📝 自定义配置

如需修改引脚配置，请编辑 `pico-rv32ima/rv32_config.h` 文件中的相应定义。
