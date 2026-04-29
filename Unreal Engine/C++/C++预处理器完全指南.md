# C++ 预处理器完全指南

## 核心认知

> [!important] 带 `#` 的都是预处理指令吗？
> **是的，100% 正确。** 所有以 `#` 开头的行都是**预处理指令**（Preprocessor Directive），它们在编译之前被处理，不是 C++ 语法的一部分。
>
> 反过来也成立：**不带 `#` 的代码全部不是预处理指令。**

### 预处理器 vs 编译器

| | 预处理器 | 编译器 |
|--|---------|--------|
| 处理什么 | `#` 开头的指令 | C++ 代码 |
| 懂 C++ 语法吗？ | **不懂**，只做文本操作 | 懂 |
| 知道类型吗？ | 不知道 | 知道 |
| 输入 | `.cpp` + `.h` 文件 | 预处理后的文本 |
| 输出 | 展开后的纯文本（`.i` 文件） | 汇编代码 |

---

## 预处理指令大全

### 一、文件包含

#### `#include`

把另一个文件的**全部内容**复制粘贴到当前位置。

```cpp
#include <iostream>       // 尖括号：在系统/标准库目录搜索
#include "MyHeader.h"     // 引号：先在当前目录搜索，找不到再去系统目录
#include "Sub/Utils.h"    // 可以带相对路径
```

> [!note] `<>` vs `""`
> - `<file>` — 只在**系统 include 路径**里找（编译器/IDE 设置的路径）
> - `"file"` — 先在**当前文件所在目录**找，找不到再去系统路径
>
> UE 项目中一般用 `""`，标准库用 `<>`。

#### 展开后的效果

```cpp
// 假设 Player.h 有 5 行
// 在 main.cpp 里 #include "Player.h"
// 等价于把那 5 行原封不动粘贴到 #include 所在的位置
```

---

### 二、宏定义与取消

#### `#define` — 定义宏

```cpp
// 1. 对象式宏（最简单，就是文本替换）
#define PI 3.14159
#define MAX_PLAYERS 100
#define ENGINE_VERSION "5.4"

// 使用：
float circumference = 2 * PI * radius;  // 预处理后变成 2 * 3.14159 * radius

// 2. 函数式宏（带参数）
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define LOG(msg) std::cout << "[LOG] " << msg << std::endl

// 使用：
int y = SQUARE(5);        // 展开为 ((5) * (5))
int z = MAX(10, 20);      // 展开为 ((10) > (20) ? (10) : (20))

// 3. 空宏（只定义名字，没有值）
#define DEBUG
#define PLATFORM_WINDOWS

// 它们的值是"空"，但"已定义"这件事本身就有用（配合 #ifdef）

// 4. 多行宏（用 \ 续行）
#define DECLARE_CLASS(ClassName) \
    class ClassName { \
    public: \
        ClassName(); \
        ~ClassName(); \
    };

// 5. 带 ## 的宏（令牌粘接）
#define CONCAT(a, b) a##b
CONCAT(My, Class)   // → MyClass

// 6. 带 # 的宏（字符串化）
#define STRINGIFY(x) #x
STRINGIFY(hello)    // → "hello"

// 7. 可变参数宏（C++11）
#define LOG_FMT(fmt, ...) printf(fmt, __VA_ARGS__)
LOG_FMT("x=%d, y=%d\n", 10, 20);
```

> [!warning] 函数式宏的陷阱
> ```cpp
> #define SQUARE(x) x * x      // 错！
> SQUARE(1 + 2)                // 展开为 1 + 2 * 1 + 2 = 5（不是9）
>
> #define SQUARE(x) ((x) * (x))  // 对！加括号
> SQUARE(1 + 2)                  // 展开为 ((1 + 2) * (1 + 2)) = 9
> ```
> **规则：函数式宏的参数和整体都要加括号。**

#### `#undef` — 取消宏定义

```cpp
#define TEMP 100
// ... 用完了 ...
#undef TEMP          // 从这里开始 TEMP 不再被识别
// 之后再用 TEMP 会报"未定义"
```

用途：
- 防止宏名冲突
- 限制宏的作用范围
- UE 里有时候 `#undef` 第三方库的宏避免污染

---

### 三、条件编译

#### `#if` / `#elif` / `#else` / `#endif`

根据条件决定哪些代码**参与编译**，哪些**直接丢弃**（不是运行时判断！）。

```cpp
#if PLATFORM_WINDOWS
    #include <windows.h>
    // Windows 专属代码
#elif PLATFORM_LINUX
    #include <unistd.h>
    // Linux 专属代码
#elif PLATFORM_MAC
    #include <mach/mach.h>
#else
    #error "Unsupported platform!"
#endif
```

#### `#ifdef` / `#ifndef`（最常用）

```cpp
// #ifdef = "如果已定义"
#ifdef DEBUG
    printf("Debug mode is ON\n");
#endif

// #ifndef = "如果未定义"
#ifndef PLAYER_H        // 头文件保护的经典写法
#define PLAYER_H
// ... 头文件内容 ...
#endif

// 等价的简写
#if defined(DEBUG)        // 和 #ifdef DEBUG 一样
#if !defined(PLAYER_H)    // 和 #ifndef PLAYER_H 一样
```

#### `defined()` 运算符

在 `#if` 里可以用 `defined()` 来组合条件：

```cpp
// 多个条件组合
#if defined(DEBUG) && defined(PLATFORM_WINDOWS)
    // 只在 Windows Debug 时编译
#endif

#if defined(USE_OPENGL) || defined(USE_VULKAN)
    // 有图形 API 时编译
#endif

#if !defined(NDEBUG)
    // 不是 Release 时编译
#endif

// 数值比较
#if ENGINE_MAJOR_VERSION >= 5
    // UE5 以上
#endif

#if __cplusplus >= 201703L
    // C++17 以上
#endif
```

> [!tip] `#if 0` 技巧 — 注释大段代码
> ```cpp
> #if 0
> // 这里面的所有代码都不会被编译
> // 比 /* */ 好用，因为可以嵌套
> void old_function() { ... }
> #endif
> ```

---

### 四、头文件保护（防止重复包含）

#### 方式一：`#ifndef` / `#define` / `#endif`（传统）

```cpp
// Player.h
#ifndef PLAYER_H    // 如果 PLAYER_H 还没定义过
#define PLAYER_H    // 定义它（下次再 include 就跳过了）

class Player {
    int hp;
};

#endif              // 结束
```

#### 方式二：`#pragma once`（现代，UE 推荐）

```cpp
// Player.h
#pragma once        // 一行搞定，告诉预处理器：这个文件只包含一次

class Player {
    int hp;
};
```

| | `#ifndef` 方式 | `#pragma once` |
|--|---------------|----------------|
| 标准性 | C/C++ 标准 | 非标准但所有主流编译器支持 |
| 写法 | 三行 | 一行 |
| 风险 | 宏名写错会出问题 | 无 |
| UE 中 | 旧代码有 | 新代码推荐用这个 |

---

### 五、错误和警告

#### `#error` — 强制编译失败

```cpp
#if !defined(PLATFORM_WINDOWS) && !defined(PLATFORM_LINUX)
    #error "You must define a platform!"
#endif

// 如果走到 #error，编译直接终止并显示这条消息
```

#### `#warning` — 输出警告但继续编译

```cpp
#warning "This feature is deprecated, use NewFeature instead"
// GCC/Clang 支持，MSVC 不支持（MSVC 用 #pragma message）
```

---

### 六、`#pragma` — 编译器特定指令

`#pragma` 是给**编译器**的特殊命令（不同编译器支持不同的 pragma）：

#### 通用 pragma

```cpp
#pragma once                    // 头文件只包含一次

#pragma pack(push, 1)           // 设置结构体对齐为 1 字节（紧密排列）
struct NetworkPacket {
    char type;     // 1 byte
    int data;      // 4 bytes（紧贴 type 后面，不填充）
};
#pragma pack(pop)               // 恢复默认对齐
```

#### MSVC 专用 pragma

```cpp
#pragma warning(push)
#pragma warning(disable: 4996)   // 禁用特定警告
// ... 有警告的代码 ...
#pragma warning(pop)             // 恢复警告设置

#pragma comment(lib, "ws2_32.lib")  // 告诉链接器链接这个库

#pragma message("Compiling Player module...")  // 编译时输出消息
```

#### GCC/Clang 专用 pragma

```cpp
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wunused-variable"
// ... 有未使用变量的代码 ...
#pragma GCC diagnostic pop
```

#### UE 中常见的 pragma

```cpp
#pragma once                         // 几乎所有 UE 头文件

// 禁用第三方库的警告
THIRD_PARTY_INCLUDES_START
#include "ThirdParty/lib.h"
THIRD_PARTY_INCLUDES_END
// 上面两个宏展开后就是一堆 #pragma warning(disable: ...)
```

---

### 七、行控制

#### `#line` — 修改编译器报错时显示的行号和文件名

```cpp
#line 100 "generated_code.cpp"
// 从这里开始，编译器认为当前是 generated_code.cpp 的第 100 行
// 报错时显示的行号和文件名会变
```

用途：代码生成器生成的代码（比如 UE 的 .generated.h）用这个让报错信息指向原始文件。

---

### 八、预定义宏

编译器自动定义好的宏，你可以直接用：

#### 标准预定义宏

| 宏 | 含义 | 示例值 |
|----|------|--------|
| `__FILE__` | 当前文件名 | `"Player.cpp"` |
| `__LINE__` | 当前行号 | `42` |
| `__DATE__` | 编译日期 | `"Apr 29 2026"` |
| `__TIME__` | 编译时间 | `"23:45:01"` |
| `__cplusplus` | C++ 标准版本 | `201703L`(C++17), `202002L`(C++20) |
| `__func__` | 当前函数名 | `"TakeDamage"` |

#### 编译器特定预定义宏

| 宏 | 含义 |
|----|------|
| `_MSC_VER` | MSVC 版本（1930 = VS2022） |
| `__GNUC__` | GCC 主版本号 |
| `__clang__` | 是否是 Clang 编译器 |
| `_WIN32` | Windows 平台（32和64位都定义） |
| `_WIN64` | Windows 64 位 |
| `__linux__` | Linux 平台 |
| `__APPLE__` | macOS / iOS |
| `NDEBUG` | Release 模式（标准库 assert 用它） |

#### UE 特定预定义宏

| 宏 | 含义 |
|----|------|
| `WITH_EDITOR` | 编辑器模式编译 |
| `UE_BUILD_DEBUG` | Debug 配置 |
| `UE_BUILD_DEVELOPMENT` | Development 配置 |
| `UE_BUILD_SHIPPING` | Shipping（发布）配置 |
| `PLATFORM_WINDOWS` | Windows 平台 |
| `ENGINE_MAJOR_VERSION` | 引擎主版本号（5） |
| `ENGINE_MINOR_VERSION` | 引擎次版本号 |

---

### 九、常见实战用法

#### 1. 跨平台代码

```cpp
#if PLATFORM_WINDOWS
    #include <windows.h>
    void Sleep(int ms) { ::Sleep(ms); }
#elif PLATFORM_LINUX
    #include <unistd.h>
    void Sleep(int ms) { usleep(ms * 1000); }
#endif
```

#### 2. Debug 专用代码

```cpp
#ifdef UE_BUILD_DEBUG
    #define DEBUG_LOG(msg) UE_LOG(LogTemp, Warning, TEXT(msg))
#else
    #define DEBUG_LOG(msg)  // Release 时展开为空，零开销
#endif
```

#### 3. 功能开关

```cpp
#define ENABLE_MULTIPLAYER 1
#define ENABLE_VOICE_CHAT 0

#if ENABLE_MULTIPLAYER
    #include "NetworkManager.h"
    // 多人联网相关代码
#endif

#if ENABLE_VOICE_CHAT
    #include "VoiceChat.h"
#endif
```

#### 4. 编译器兼容

```cpp
// 不同编译器的强制内联写法不同
#if defined(_MSC_VER)
    #define FORCE_INLINE __forceinline
#elif defined(__GNUC__) || defined(__clang__)
    #define FORCE_INLINE __attribute__((always_inline)) inline
#else
    #define FORCE_INLINE inline
#endif
```

#### 5. 避免宏污染（Windows.h 常见问题）

```cpp
// Windows.h 定义了 min 和 max 宏，会和 std::min/max 冲突
#define NOMINMAX            // 在 include 前定义，阻止 Windows.h 定义 min/max
#include <windows.h>

// 或者事后取消
#include <windows.h>
#undef min
#undef max
```

---

### 十、预处理器的运算符总结

| 符号 | 名称 | 用法 | 示例 |
|------|------|------|------|
| `##` | 令牌粘接 | 拼接两个 Token | `A##B` → `AB` |
| `#` | 字符串化 | 参数变成字符串 | `#x` → `"x"` |
| `defined()` | 是否已定义 | 在 `#if` 中判断 | `#if defined(X)` |
| `\` | 续行符 | 宏跨行 | 放在行末 |

---

### 十一、查看预处理结果的方法

| 工具 | 命令 |
|------|------|
| GCC/Clang | `g++ -E main.cpp -o main.i` |
| MSVC 命令行 | `cl /P main.cpp`（生成 `.i` 文件） |
| VS IDE | 右键文件 → 属性 → C/C++ → 预处理器 → 预处理到文件 = 是 |

> [!tip] 遇到宏看不懂怎么办
> 展开它！用 `-E` 或 `/P` 看预处理后的结果，所有宏都会被替换成最终文本，一目了然。

---

## 速查：所有预处理指令一览

| 指令 | 作用 |
|------|------|
| `#include <file>` | 包含系统头文件 |
| `#include "file"` | 包含项目头文件 |
| `#define NAME VALUE` | 定义对象式宏 |
| `#define NAME(x) expr` | 定义函数式宏 |
| `#undef NAME` | 取消宏定义 |
| `#if expr` | 条件编译（表达式为真时） |
| `#elif expr` | 否则如果 |
| `#else` | 否则 |
| `#endif` | 结束条件编译 |
| `#ifdef NAME` | 如果 NAME 已定义 |
| `#ifndef NAME` | 如果 NAME 未定义 |
| `#pragma ...` | 编译器特定指令 |
| `#error "msg"` | 强制编译失败 |
| `#warning "msg"` | 输出编译警告 |
| `#line N "file"` | 修改行号/文件名 |

---

## 相关链接

- [[C++编译原理]] — 预处理是编译四步中的第一步
- [[模板编程]] — 宏的 `##` 在 UE 模板技巧中大量使用
- [[C++和计算机基础总览]]
