# 开发指南

本指南面向希望参与 pico-rv32ima 项目开发或基于此项目进行二次开发的开发者。

## 🚀 开发环境设置

### 必需工具
- **Git** (版本控制)
- **CMake** 3.13+ (构建系统)
- **ARM GCC** (交叉编译器)
- **Python 3** (脚本和工具)
- **VS Code** (推荐 IDE)

### 推荐扩展
- Raspberry Pi Pico (官方扩展)
- C/C++ (Microsoft)
- CMake Tools
- Cortex-Debug (用于调试)

### 开发环境配置
```bash
# 克隆项目
git clone https://github.com/cxgreat2014/pico-rv32ima.git
cd pico-rv32ima

# 初始化子模块
git submodule update --init --recursive

# 设置 Pico SDK
export PICO_SDK_PATH=/path/to/pico-sdk

# 创建开发分支
git checkout -b feature/your-feature-name
```

## 📁 项目结构

```
pico-rv32ima/
├── pico-rv32ima/           # 主要源代码
│   ├── main.c              # 程序入口
│   ├── emulator.c          # RISC-V 模拟器
│   ├── mini-rv32ima.h      # 模拟器核心
│   ├── psram.c             # PSRAM 驱动
│   ├── cache.c             # 缓存系统
│   ├── console.c           # 控制台系统
│   ├── terminal.c          # 终端处理
│   ├── rv32_config.h       # 配置文件
│   ├── vga/                # VGA 显示
│   ├── ps2/                # PS/2 键盘
│   └── petitfatfs/         # FAT 文件系统
├── linux/                  # Linux 构建
│   ├── Makefile            # 构建脚本
│   ├── linux_configs/      # 配置文件
│   └── buildroot/          # Buildroot (克隆)
├── docs/                   # 项目文档
├── pictures/               # 截图和图片
├── CMakeLists.txt          # 主构建文件
└── README.md               # 项目说明
```

## 🔧 核心组件开发

### 1. RISC-V 模拟器扩展

#### 添加新指令
```c
// 在 mini-rv32ima.h 中添加指令处理
case 0x??:  // 新指令操作码
{
    // 指令解码
    uint32_t rd = (ir >> 7) & 0x1f;
    uint32_t rs1 = (ir >> 15) & 0x1f;
    uint32_t rs2 = (ir >> 20) & 0x1f;
    
    // 指令执行
    rval = /* 计算结果 */;
    break;
}
```

#### 添加 CSR 寄存器
```c
// 在 CSR 处理部分添加
case 0x???:  // 新 CSR 地址
    rval = CSR(your_new_csr);
    break;
```

### 2. 内存系统扩展

#### 扩展 PSRAM 支持
```c
// 在 psram.c 中添加新芯片支持
#define NEW_PSRAM_KGD 0x??

int psram_init() {
    // 检测新芯片类型
    psram_read_id(chip, chipId);
    if (chipId[1] == NEW_PSRAM_KGD) {
        // 新芯片初始化
    }
}
```

#### 缓存算法优化
```c
// 在 cache.c 中实现新的替换策略
typedef enum {
    CACHE_LRU,
    CACHE_LFU,
    CACHE_RANDOM
} cache_policy_t;

void cache_replace_line(uint32_t set, cache_policy_t policy);
```

### 3. 外设驱动开发

#### 添加新的 SPI 设备
```c
// 创建新的驱动文件 new_device.c
typedef struct {
    spi_inst_t *spi;
    uint cs_pin;
    uint clock_pin;
} new_device_t;

int new_device_init(new_device_t *dev);
int new_device_read(new_device_t *dev, uint8_t *data, size_t len);
int new_device_write(new_device_t *dev, const uint8_t *data, size_t len);
```

#### 扩展控制台支持
```c
// 在 console.c 中添加新的控制台类型
#define CONSOLE_NEW_TYPE 1

#if CONSOLE_NEW_TYPE
void new_console_init(void);
void new_console_putc(char c);
char new_console_getc(void);
#endif
```

## 🧪 测试和调试

### 单元测试
```c
// 创建测试文件 test_component.c
#include "unity.h"  // 测试框架
#include "component.h"

void test_component_init(void) {
    int result = component_init();
    TEST_ASSERT_EQUAL(0, result);
}

void test_component_operation(void) {
    uint32_t input = 0x12345678;
    uint32_t expected = 0x87654321;
    uint32_t actual = component_process(input);
    TEST_ASSERT_EQUAL_HEX32(expected, actual);
}
```

### 集成测试
```c
// 创建集成测试
void test_psram_cache_integration(void) {
    // 测试 PSRAM 和缓存的协同工作
    uint32_t test_data = 0xDEADBEEF;
    uint32_t addr = 0x1000;
    
    // 写入数据
    psram_access(addr, 4, true, &test_data);
    
    // 通过缓存读取
    uint32_t read_data;
    cache_read(addr, 4, &read_data);
    
    TEST_ASSERT_EQUAL_HEX32(test_data, read_data);
}
```

### 性能测试
```c
// 性能基准测试
void benchmark_emulator_performance(void) {
    uint64_t start_time = GetTimeMicroseconds();
    
    // 执行固定数量的指令
    for (int i = 0; i < 1000000; i++) {
        MiniRV32IMAStep(&core, NULL, 0, 1, 1);
    }
    
    uint64_t end_time = GetTimeMicroseconds();
    uint64_t elapsed = end_time - start_time;
    
    console_printf("Performance: %llu MIPS\n", 1000000 * 1000000 / elapsed);
}
```

## 📝 代码规范

### 命名约定
```c
// 函数名：小写 + 下划线
int psram_init(void);
void console_printf(const char *format, ...);

// 变量名：小写 + 下划线
uint32_t memory_size;
bool is_initialized;

// 常量：大写 + 下划线
#define MAX_MEMORY_SIZE (16 * 1024 * 1024)
#define DEFAULT_CLOCK_FREQ 400000

// 结构体：小写 + 下划线 + _t 后缀
typedef struct {
    uint32_t address;
    size_t size;
} memory_region_t;
```

### 代码格式
```c
// 函数定义
int function_name(int param1, const char *param2) 
{
    // 4 空格缩进
    if (condition) {
        // 语句
    } else {
        // 语句
    }
    
    return 0;
}

// 条件语句
if (condition) {
    // 单行也要使用大括号
}

// 循环
for (int i = 0; i < count; i++) {
    // 循环体
}
```

### 注释规范
```c
/**
 * @brief 函数简要描述
 * @param param1 参数1描述
 * @param param2 参数2描述
 * @return 返回值描述
 */
int example_function(int param1, const char *param2);

// 单行注释用于解释复杂逻辑
uint32_t result = (value << 2) | 0x3;  // 左移2位并设置低2位
```

## 🔄 开发流程

### 1. 功能开发
```bash
# 创建功能分支
git checkout -b feature/new-feature

# 开发和测试
# ... 编写代码 ...

# 提交更改
git add .
git commit -m "feat: add new feature description"
```

### 2. 代码审查
- 确保代码符合项目规范
- 添加必要的测试
- 更新相关文档
- 检查性能影响

### 3. 测试验证
```bash
# 构建测试
mkdir build && cd build
cmake .. && make

# 运行单元测试
make test

# 硬件测试
# 在实际硬件上验证功能
```

### 4. 文档更新
- 更新 API 文档
- 添加使用示例
- 更新配置说明
- 记录已知问题

## 🐛 调试技巧

### 使用 printf 调试
```c
#ifdef DEBUG
#define DBG_PRINTF(fmt, ...) console_printf("[DBG] " fmt, ##__VA_ARGS__)
#else
#define DBG_PRINTF(fmt, ...)
#endif

// 使用示例
DBG_PRINTF("Function called with param=%d\n", param);
```

### 使用断言
```c
#include <assert.h>

void critical_function(void *ptr) {
    assert(ptr != NULL);  // 调试版本会检查
    // 函数实现
}
```

### 内存调试
```c
// 检查内存访问
void debug_memory_access(uint32_t addr, size_t size, bool write) {
    if (addr + size > MAX_MEMORY_SIZE) {
        console_printf("ERROR: Memory access out of bounds!\n");
        console_printf("  Address: 0x%08x, Size: %zu\n", addr, size);
    }
}
```

## 📚 贡献指南

### 提交 Pull Request
1. Fork 项目仓库
2. 创建功能分支
3. 实现功能并添加测试
4. 确保所有测试通过
5. 提交 Pull Request

### 提交信息格式
```
type(scope): description

[optional body]

[optional footer]
```

类型：
- `feat`: 新功能
- `fix`: 错误修复
- `docs`: 文档更新
- `style`: 代码格式
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具相关

### 代码审查清单
- [ ] 代码符合项目规范
- [ ] 添加了适当的测试
- [ ] 文档已更新
- [ ] 性能影响已评估
- [ ] 向后兼容性已考虑

## 🔮 未来发展方向

### 性能优化
- 指令级并行优化
- 更高效的缓存算法
- DMA 优化

### 功能扩展
- 支持更多外设
- 网络功能
- 图形界面支持

### 平台支持
- 支持更多 RP2040/RP2350 开发板
- 适配其他微控制器平台

欢迎开发者参与项目贡献，共同完善这个有趣的项目！
