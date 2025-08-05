# 项目架构

本文档详细介绍 pico-rv32ima 项目的技术架构、核心组件和实现原理。

## 🏗️ 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Raspberry Pi Pico                        │
├─────────────────────────────────────────────────────────────┤
│  Core 0              │  Core 1                              │
│  ┌─────────────────┐ │  ┌─────────────────────────────────┐ │
│  │ Console Task    │ │  │ RISC-V Emulator                 │ │
│  │ - USB CDC       │ │  │ - mini-rv32ima core             │ │
│  │ - UART          │ │  │ - Linux kernel execution        │ │
│  │ - VGA Terminal  │ │  │ - System calls handling         │ │
│  │ - PS/2 Keyboard │ │  │ - Memory management             │ │
│  └─────────────────┘ │  └─────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                    Hardware Abstraction                     │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │ PSRAM       │ │ SD Card     │ │ VGA/PS2     │           │
│  │ (SPI)       │ │ (SPI)       │ │ (GPIO/PIO)  │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

## 🧠 核心组件

### 1. RISC-V 模拟器核心

基于 CNLohr 的 mini-rv32ima 实现，支持：

- **指令集**: RV32IMA (32位基础指令 + 乘法/除法 + 原子操作)
- **特权级**: Machine Mode 和 User Mode
- **CSR**: 控制和状态寄存器支持
- **中断**: 定时器中断和外部中断
- **系统调用**: Linux 系统调用模拟

#### 关键特性
```c
// 核心配置
#define MINI_RV32_RAM_SIZE (EMULATOR_RAM_MB * 1024 * 1024)
#define MINIRV32_RAM_IMAGE_OFFSET 0x80000000

// 执行循环
int MiniRV32IMAStep(struct MiniRV32IMAState *state, 
                    uint8_t *ram, uint32_t ram_size, 
                    uint64_t elapsed_us, int count);
```

### 2. 内存管理系统

#### PSRAM 接口
- **容量**: 8MB 或 16MB (1-2 片 PSRAM)
- **接口**: SPI 高速访问
- **缓存**: 128KB 二路组相联缓存

```c
// PSRAM 访问接口
void psram_access(uint32_t addr, size_t size, bool write, void *bufP);

// 缓存系统
typedef struct {
    uint32_t tag;
    uint8_t data[CACHE_LINE_SIZE];
    bool valid;
    bool dirty;
} cache_line_t;
```

#### 内存映射
```
0x80000000 - 0x80000000 + RAM_SIZE : 主内存 (PSRAM)
0x10000000 - 0x10000FFF             : UART MMIO
0x02000000 - 0x02000FFF             : CLINT (定时器)
0x0C000000 - 0x0CFFFFFF             : PLIC (中断控制器)
```

### 3. 存储系统

#### SD 卡接口
- **协议**: SPI 模式
- **文件系统**: FAT16/FAT32 (PetitFatFS)
- **文件**: IMAGE (内核), ROOTFS (根文件系统)

```c
// SD 卡操作
FRESULT pf_mount(FATFS *fs);
FRESULT pf_open(const char *path);
FRESULT pf_read(void *buff, UINT btr, UINT *br);
```

### 4. 显示和输入系统

#### VGA 文本显示
- **分辨率**: 640x480 @ 60Hz
- **文本模式**: 80x30 字符
- **颜色**: 16 色前景/背景
- **实现**: PIO + DMA

```c
// VGA 核心结构
char termBuf[TERM_HEIGHT][TERM_WIDTH];
uint8_t fgColBuf[TERM_HEIGHT][TERM_WIDTH];
uint8_t bgColBuf[TERM_HEIGHT][TERM_WIDTH];
```

#### PS/2 键盘
- **协议**: PS/2 标准协议
- **实现**: GPIO 中断 + 状态机
- **缓冲**: 环形缓冲区

### 5. 控制台系统

支持多种控制台同时工作：

```c
// 控制台类型
#define CONSOLE_CDC  1  // USB CDC
#define CONSOLE_VGA  1  // VGA 文本
#define CONSOLE_UART 0  // UART 串口

// 统一接口
void console_putc(char c);
void console_printf(const char *format, ...);
```

## ⚙️ 系统启动流程

### 1. 硬件初始化
```c
int main() {
    // 超频配置
    vreg_set_voltage(RP2040_OVERVOLT);
    set_sys_clock_khz(RP2040_CPU_FREQ, true);
    
    // 控制台初始化
    console_init();
    
    // 双核启动
    multicore_launch_core1(core1_entry);
}
```

### 2. 系统组件启动
```c
void core1_entry() {
    // 等待用户按键
    wait_button();
    
    // PSRAM 初始化
    psram_init();
    
    // SD 卡挂载
    pf_mount(&fatfs);
    
    // 启动 RISC-V 模拟器
    riscv_emu();
}
```

### 3. Linux 内核加载
```c
int riscv_emu() {
    // 打开内核文件
    pf_open(KERNEL_FILENAME);
    
    // 加载到 PSRAM
    while (pf_read(blk_buf, sizeof(blk_buf), &br)) {
        psram_access(addr, br, true, blk_buf);
        addr += br;
    }
    
    // 设置启动参数
    core.regs[10] = 0x00;  // hart ID
    core.regs[11] = dtb_ptr; // device tree
    core.pc = MINIRV32_RAM_IMAGE_OFFSET;
    
    // 开始执行
    MiniRV32IMAStep(&core, NULL, 0, elapsedUs, instrs_per_flip);
}
```

## 🔄 执行模型

### 双核分工
- **Core 0**: 处理 I/O 和控制台任务
- **Core 1**: 运行 RISC-V 模拟器

### 时间管理
```c
// 时间同步
uint64_t GetTimeMicroseconds();
uint64_t elapsedUs = (rt - lastTime) * time_divisor;

// 指令执行控制
int instrs_per_flip = 1024;
MiniRV32IMAStep(&core, NULL, 0, elapsedUs, instrs_per_flip);
```

### 中断处理
```c
// 定时器中断
if ((CSR(mip) & (1<<7)) && (CSR(mie) & (1<<7)) && (CSR(mstatus) & 0x8)) {
    trap = 0x80000007;  // 定时器中断
}

// 系统调用处理
case 0x7777: return EMU_REBOOT;   // 重启
case 0x5555: return EMU_POWEROFF; // 关机
```

## 🚀 性能优化

### 1. 缓存系统
- **类型**: 二路组相联
- **大小**: 128KB
- **策略**: LRU 替换

### 2. SPI 优化
- **PSRAM**: 55MHz 时钟频率
- **DMA**: 用于大块数据传输
- **批处理**: 减少 SPI 事务开销

### 3. 指令优化
- **批量执行**: 每次执行多条指令
- **分支预测**: 简单的分支优化
- **内存预取**: 缓存友好的访问模式

## 🔧 扩展接口

### MMIO 处理
```c
// 控制寄存器访问
static uint32_t HandleControlStore(uint32_t addy, uint32_t val);
static uint32_t HandleControlLoad(uint32_t addy);

// UART 8250 模拟
if (addy == 0x10000000) console_putc(val);      // 数据寄存器
if (addy == 0x10000005) return 0x60 | IsKBHit(); // 状态寄存器
```

### 设备树支持
```c
// 设备树指针传递
core.regs[11] = dtb_ptr ? (dtb_ptr + MINIRV32_RAM_IMAGE_OFFSET) : 0;
```

## 📊 系统限制

### 硬件限制
- **内存**: 最大 16MB PSRAM
- **存储**: SD 卡容量限制
- **CPU**: RP2040/RP2350 性能限制

### 软件限制
- **MMU**: 无内存管理单元
- **浮点**: 软件浮点运算
- **多核**: Linux 运行在单核模拟器上

## 🔍 调试和监控

### 性能监控
```c
// 周期计数
uint64_t cycleh, cyclel;
console_printf("Cycles: 0x%08x%08x\n", core.cycleh, core.cyclel);

// 指令计数
long long instct;
```

### 错误处理
```c
// 异常类型
case 1: // 指令访问异常
case 2: // 非法指令
case 3: // 断点
case 4: // 加载地址不对齐
case 5: // 加载访问异常
```

这个架构设计实现了在资源受限的微控制器上运行完整 Linux 系统的目标，通过精心的组件设计和优化策略，达到了良好的性能和稳定性。
