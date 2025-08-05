# API 参考文档

本文档提供 pico-rv32ima 项目核心 API 的详细参考信息。

## 📋 目录

- [RISC-V 模拟器 API](#risc-v-模拟器-api)
- [内存管理 API](#内存管理-api)
- [控制台 API](#控制台-api)
- [存储系统 API](#存储系统-api)
- [VGA 显示 API](#vga-显示-api)
- [PS/2 键盘 API](#ps2-键盘-api)
- [配置 API](#配置-api)

## 🧠 RISC-V 模拟器 API

### 核心结构体

```c
struct MiniRV32IMAState {
    uint32_t regs[32];      // 通用寄存器
    uint32_t pc;            // 程序计数器
    uint32_t mstatus;       // 机器状态寄存器
    uint32_t cyclel;        // 周期计数器低位
    uint32_t cycleh;        // 周期计数器高位
    uint32_t timerl;        // 定时器低位
    uint32_t timerh;        // 定时器高位
    uint32_t timermatchl;   // 定时器匹配低位
    uint32_t timermatchh;   // 定时器匹配高位
    uint32_t mscratch;      // 机器模式暂存寄存器
    uint32_t mtvec;         // 机器模式陷阱向量
    uint32_t mie;           // 机器模式中断使能
    uint32_t mip;           // 机器模式中断挂起
    uint32_t mepc;          // 机器模式异常程序计数器
    uint32_t mtval;         // 机器模式陷阱值
    uint32_t mcause;        // 机器模式陷阱原因
    uint32_t extraflags;    // 额外标志位
};
```

### 主要函数

#### MiniRV32IMAStep
```c
/**
 * @brief 执行 RISC-V 指令
 * @param state 处理器状态
 * @param ram 内存指针 (可为 NULL，使用 PSRAM)
 * @param ram_size 内存大小
 * @param elapsed_us 经过的微秒数
 * @param count 最大执行指令数
 * @return 执行状态码
 */
int MiniRV32IMAStep(struct MiniRV32IMAState *state, 
                    uint8_t *ram, uint32_t ram_size, 
                    uint64_t elapsed_us, int count);
```

**返回值**:
- `0`: 正常执行
- `1`: WFI (等待中断)
- `3`: 断点
- `0x7777`: 系统重启请求
- `0x5555`: 系统关机请求

#### riscv_emu
```c
/**
 * @brief 启动 RISC-V 模拟器主循环
 * @return 退出状态
 */
int riscv_emu(void);
```

**返回值**:
- `EMU_REBOOT`: 重启请求
- `EMU_POWEROFF`: 关机请求
- `EMU_UNKNOWN`: 未知错误

## 💾 内存管理 API

### PSRAM 接口

#### psram_init
```c
/**
 * @brief 初始化 PSRAM
 * @return 0 成功，负数表示错误
 */
int psram_init(void);
```

#### psram_access
```c
/**
 * @brief 访问 PSRAM 数据
 * @param addr 地址
 * @param size 数据大小
 * @param write true=写入，false=读取
 * @param bufP 数据缓冲区
 */
void psram_access(uint32_t addr, size_t size, bool write, void *bufP);
```

#### psram_load_data
```c
/**
 * @brief 批量加载数据到 PSRAM
 * @param buf 源数据缓冲区
 * @param addr 目标地址
 * @param size 数据大小
 */
void psram_load_data(void *buf, uint32_t addr, uint32_t size);
```

### 缓存接口

#### cache_init
```c
/**
 * @brief 初始化缓存系统
 */
void cache_init(void);
```

#### cache_read
```c
/**
 * @brief 从缓存读取数据
 * @param addr 地址
 * @param size 数据大小
 * @param data 输出缓冲区
 * @return true 缓存命中，false 缓存未命中
 */
bool cache_read(uint32_t addr, size_t size, void *data);
```

#### cache_write
```c
/**
 * @brief 向缓存写入数据
 * @param addr 地址
 * @param size 数据大小
 * @param data 输入数据
 */
void cache_write(uint32_t addr, size_t size, const void *data);
```

## 🖥️ 控制台 API

### 初始化和任务

#### console_init
```c
/**
 * @brief 初始化控制台系统
 */
void console_init(void);
```

#### console_task
```c
/**
 * @brief 控制台任务处理 (需要定期调用)
 */
void console_task(void);
```

### 输出函数

#### console_putc
```c
/**
 * @brief 输出单个字符
 * @param c 要输出的字符
 */
void console_putc(char c);
```

#### console_puts
```c
/**
 * @brief 输出字符串
 * @param s 要输出的字符串
 */
void console_puts(char s[]);
```

#### console_printf
```c
/**
 * @brief 格式化输出
 * @param format 格式字符串
 * @param ... 可变参数
 */
void console_printf(const char *format, ...);
```

#### console_panic
```c
/**
 * @brief 输出错误信息并停止系统
 * @param format 格式字符串
 * @param ... 可变参数
 */
void console_panic(const char *format, ...);
```

### 输入函数

#### IsKBHit
```c
/**
 * @brief 检查是否有键盘输入
 * @return true 有输入，false 无输入
 */
bool IsKBHit(void);
```

#### ReadKBByte
```c
/**
 * @brief 读取键盘输入字符
 * @return 输入的字符
 */
char ReadKBByte(void);
```

## 💿 存储系统 API

### PetitFatFS 接口

#### pf_mount
```c
/**
 * @brief 挂载文件系统
 * @param fs 文件系统对象
 * @return FR_OK 成功，其他值表示错误
 */
FRESULT pf_mount(FATFS *fs);
```

#### pf_open
```c
/**
 * @brief 打开文件
 * @param path 文件路径
 * @return FR_OK 成功，其他值表示错误
 */
FRESULT pf_open(const char *path);
```

#### pf_read
```c
/**
 * @brief 读取文件数据
 * @param buff 输出缓冲区
 * @param btr 要读取的字节数
 * @param br 实际读取的字节数
 * @return FR_OK 成功，其他值表示错误
 */
FRESULT pf_read(void *buff, UINT btr, UINT *br);
```

#### pf_lseek
```c
/**
 * @brief 移动文件指针
 * @param ofs 偏移量
 * @return FR_OK 成功，其他值表示错误
 */
FRESULT pf_lseek(DWORD ofs);
```

## 📺 VGA 显示 API

### 初始化和控制

#### VGA_init
```c
/**
 * @brief 初始化 VGA 显示
 */
void VGA_init(void);
```

#### VGA_clear
```c
/**
 * @brief 清空屏幕
 */
void VGA_clear(void);
```

### 文本输出

#### VGA_putc
```c
/**
 * @brief 输出单个字符到 VGA
 * @param c 要输出的字符
 */
void VGA_putc(char c);
```

#### VGA_puts
```c
/**
 * @brief 输出字符串到 VGA
 * @param s 要输出的字符串
 */
void VGA_puts(char s[]);
```

### 颜色控制

#### VGA_set_color
```c
/**
 * @brief 设置前景和背景颜色
 * @param fg 前景颜色 (0-15)
 * @param bg 背景颜色 (0-15)
 */
void VGA_set_color(uint8_t fg, uint8_t bg);
```

### 光标控制

#### VGA_set_cursor
```c
/**
 * @brief 设置光标位置
 * @param x 列位置 (0-79)
 * @param y 行位置 (0-29)
 */
void VGA_set_cursor(uint8_t x, uint8_t y);
```

## ⌨️ PS/2 键盘 API

### 初始化

#### ps2_init
```c
/**
 * @brief 初始化 PS/2 键盘
 */
void ps2_init(void);
```

### 输入处理

#### ps2_available
```c
/**
 * @brief 检查是否有键盘输入可用
 * @return true 有输入，false 无输入
 */
bool ps2_available(void);
```

#### ps2_read
```c
/**
 * @brief 读取键盘输入
 * @return 输入的字符，如果无输入返回 0
 */
char ps2_read(void);
```

## ⚙️ 配置 API

### 系统配置

#### 时钟配置
```c
// RP2040 配置
#define RP2040_CPU_FREQ 400000
#define RP2040_OVERVOLT VREG_VOLTAGE_MAX

// RP2350 配置
#define RP2350_CPU_FREQ 512000
#define RP2350_OVERVOLT VREG_VOLTAGE_1_60
```

#### 内存配置
```c
// RAM 大小 (MB)
#define EMULATOR_RAM_MB 8

// PSRAM 配置
#define PSRAM_TWO_CHIPS 0
#define PSRAM_CHIP_SIZE (8192 * 1024)
```

#### 控制台配置
```c
// 控制台类型启用/禁用
#define CONSOLE_UART 0
#define CONSOLE_CDC 1
#define CONSOLE_VGA 1
```

### 引脚配置

#### SD 卡引脚
```c
#define SD_SPI_PIN_CK 2
#define SD_SPI_PIN_TX 3
#define SD_SPI_PIN_RX 4
#define SD_SPI_PIN_CS 0
```

#### PSRAM 引脚
```c
#define PSRAM_SPI_PIN_CK 10
#define PSRAM_SPI_PIN_TX 11
#define PSRAM_SPI_PIN_RX 12
#define PSRAM_SPI_PIN_S1 13
#define PSRAM_SPI_PIN_S2 14
```

#### VGA 引脚
```c
#define VGA_VSYNC_PIN 16
#define VGA_HSYNC_PIN 17
#define VGA_R_PIN 18
```

#### PS/2 引脚
```c
#define PS2_PIN_DATA 26
#define PS2_PIN_CK 27
```

## 🔧 工具函数

### 时间函数

#### GetTimeMicroseconds
```c
/**
 * @brief 获取当前时间 (微秒)
 * @return 当前时间戳
 */
uint64_t GetTimeMicroseconds(void);
```

#### MiniSleep
```c
/**
 * @brief 短暂休眠
 */
void MiniSleep(void);
```

### 按钮处理

#### wait_button
```c
/**
 * @brief 等待 BOOTSEL 按钮按下
 */
void wait_button(void);
```

## 📊 常量定义

### 内存相关
```c
#define MINIRV32_RAM_IMAGE_OFFSET 0x80000000
#define MINI_RV32_RAM_SIZE (EMULATOR_RAM_MB * 1024 * 1024)
#define CACHE_SIZE (128 * 1024)
#define CACHE_LINE_SIZE 64
```

### 文件名
```c
#define KERNEL_FILENAME "IMAGE"
#define BLK_FILENAME "ROOTFS"
```

### 模拟器状态
```c
#define EMU_REBOOT 1
#define EMU_POWEROFF 2
#define EMU_UNKNOWN 3
```

### VGA 显示
```c
#define TERM_WIDTH 80
#define TERM_HEIGHT 30
#define VGA_COLORS 16
```

## 🐛 错误代码

### PSRAM 错误
- `-1`: 第一片 PSRAM 初始化失败
- `-2`: 第二片 PSRAM 初始化失败

### 文件系统错误
- `FR_OK (0)`: 成功
- `FR_DISK_ERR`: 磁盘错误
- `FR_NOT_READY`: 设备未就绪
- `FR_NO_FILE`: 文件不存在
- `FR_NOT_OPENED`: 文件未打开

这些 API 提供了完整的系统功能接口，开发者可以基于这些接口进行功能扩展和定制开发。
