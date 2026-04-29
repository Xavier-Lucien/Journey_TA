# C++ 编译原理

## 总览：从源码到可执行文件

你写的 `.cpp` 文件，经过**四个阶段**变成最终的 `.exe`（或 `.dll`）：

```
源码 (.cpp/.h)
    ↓ 预处理
预处理后的源码 (.i)
    ↓ 编译
汇编代码 (.s)
    ↓ 汇编
目标文件 (.o / .obj)
    ↓ 链接
可执行文件 (.exe / .dll / .so)
```

> [!important] 核心认知
> "编译"这个词在日常中指的是整个过程（按 F5 到运行），但严格来说它只是四步中的第二步。

---

## 一、预处理（Preprocessing）

### 做了什么

预处理器是一个**纯文本替换工具**，它不懂 C++ 语法，只做机械的文本操作：

| 操作 | 指令 | 示例 |
|------|------|------|
| 文件包含 | `#include` | 把头文件的内容**复制粘贴**进来 |
| 宏替换 | `#define` | 把宏名替换为宏体 |
| 条件编译 | `#if / #ifdef / #ifndef` | 决定哪些代码参与编译 |
| 删除注释 | — | 所有 `//` 和 `/* */` 被移除 |

### 具体例子

```cpp
// main.cpp
#include "Player.h"
#define MAX_HP 100

int main() {
    int hp = MAX_HP;  // 这里 MAX_HP 会被替换成 100
}
```

预处理后变成（简化）：

```cpp
// Player.h 的全部内容被粘贴到这里
// ...几千行头文件内容...

int main() {
    int hp = 100;  // MAX_HP 已经不存在了
}
```

### 查看预处理结果

```bash
# GCC / Clang
g++ -E main.cpp -o main.i

# MSVC
cl /P main.cpp    # 生成 main.i
```

> [!tip] UE 中的应用
> UE 的 `UPROPERTY`、`UCLASS`、`GENERATED_BODY()` 等全是宏。预处理阶段它们会展开成大量的反射代码（由 UHT 生成）。这就是为什么 UE 项目的编译时间那么长。

---

## 二、编译（Compilation）

### 做了什么

编译器把 C++ 源码翻译成**汇编代码**。这是最复杂的阶段，包含多个子步骤：

```
预处理后的源码
    ↓ 词法分析 (Lexical Analysis)
Token 流
    ↓ 语法分析 (Parsing)
抽象语法树 AST
    ↓ 语义分析 (Semantic Analysis)
带类型信息的 AST
    ↓ 优化 (Optimization)
中间表示 IR
    ↓ 代码生成 (Code Generation)
汇编代码 (.s)
```

### 2.1 词法分析（Lexer / Tokenizer）

把字符流切成一个个**Token**（最小有意义单元）：

```cpp
int x = 42 + y;
```

切成：

| Token | 类型 |
|-------|------|
| `int` | 关键字 |
| `x` | 标识符 |
| `=` | 运算符 |
| `42` | 整数字面量 |
| `+` | 运算符 |
| `y` | 标识符 |
| `;` | 分号 |

> [!note] 宏的 `##`（Token Pasting）
> 上一篇笔记里的 `A##B` 就是在预处理阶段把两个 Token 粘成一个新 Token，然后再交给词法分析。

### 2.2 语法分析（Parser）

把 Token 流组织成**抽象语法树（AST）**：

```
        =
       / \
      x    +
          / \
        42    y
```

语法错误在这一步报告：
- 少了分号 → 报错
- 括号不匹配 → 报错
- 运算符用错 → 报错

### 2.3 语义分析（Semantic Analysis）

检查代码的**含义**是否合法：

- 类型检查：`int x = "hello";` → 类型不匹配
- 作用域检查：使用了未声明的变量
- 重载决议：多个同名函数选哪个
- **模板实例化**也在这一步发生！

> [!important] 模板与编译的关系
> 模板代码在语义分析阶段才被实例化。编译器拿到具体类型后，才生成真正的函数/类代码，然后对生成的代码做类型检查。这就是为什么模板错误信息往往很长很难读。

### 2.4 优化（Optimization）

编译器会对代码进行各种优化：

```cpp
// 你写的
int x = 2 + 3;
for (int i = 0; i < 10; i++) {
    result += x;
}

// 编译器优化后（等价）
int result_final = result + 50;  // 直接算出来了
```

常见优化：
- **常量折叠**：`2 + 3` → `5`
- **死代码消除**：永远不会执行的代码直接删掉
- **内联展开**：把小函数的代码直接嵌入调用处
- **循环展开**：把短循环展开成多条语句

优化等级：

| 标志 | 含义 | 场景 |
|------|------|------|
| `-O0` | 不优化 | 调试时用，断点精准 |
| `-O1` | 基本优化 | 平衡 |
| `-O2` | 标准优化 | Release 常用 |
| `-O3` | 激进优化 | 追求极致性能 |
| `-Os` | 优化体积 | 嵌入式 |

### 2.5 代码生成

把优化后的中间表示翻译成目标平台的汇编：

```asm
; int add(int a, int b) { return a + b; }
add:
    mov eax, edi      ; 第一个参数放到 eax
    add eax, esi      ; 加上第二个参数
    ret               ; 返回结果（在 eax 里）
```

### 查看编译结果

```bash
# 生成汇编代码
g++ -S main.cpp -o main.s

# 在线工具推荐：Compiler Explorer (godbolt.org)
```

---

## 三、汇编（Assembly）

### 做了什么

汇编器把汇编代码（`.s`）翻译成**机器码**，生成目标文件（`.o` / `.obj`）。

这一步相对简单——就是把汇编指令一一对应翻译成二进制机器指令。

### 目标文件的内容

```
┌─────────────────────────────────┐
│  .text 段 — 机器码（你的代码）    │
├─────────────────────────────────┤
│  .data 段 — 已初始化的全局变量    │
├─────────────────────────────────┤
│  .bss 段 — 未初始化的全局变量     │
├─────────────────────────────────┤
│  .rodata 段 — 只读数据（字符串等）│
├─────────────────────────────────┤
│  符号表 — 函数名/变量名的地址表   │
├─────────────────────────────────┤
│  重定位表 — 需要链接时填入的地址  │
└─────────────────────────────────┘
```

> [!note] 每个 .cpp 文件独立编译
> 每个 `.cpp` 文件都会生成一个独立的 `.obj` 文件。它们之间互相不知道对方的存在——这个问题留给链接器解决。

---

## 四、链接（Linking）

### 做了什么

链接器把多个目标文件（`.obj`）和库文件（`.lib` / `.a`）合并成最终的可执行文件。

### 链接器的核心工作

1. **符号解析**：把"我要调用 `foo()` 函数"和"这里是 `foo()` 的代码"对应起来
2. **重定位**：把所有地址从"相对偏移"改为"最终绝对地址"
3. **合并段**：把所有 `.obj` 的 `.text` 段合到一起，`.data` 段合到一起...

### 常见链接错误

```
// 错误 1：未定义的引用
undefined reference to `foo()`
// 原因：声明了 foo() 但没有实现，或者没有链接对应的 .cpp / .lib

// 错误 2：重复定义
multiple definition of `bar`
// 原因：同一个函数/变量在多个 .obj 里都有定义
```

> [!warning] 头文件与链接
> 这就是为什么：
> - 函数**声明**放 `.h`（告诉编译器"这个函数存在"）
> - 函数**定义**放 `.cpp`（只编译一次，不会重复定义）
> - 如果定义放头文件且被多个 `.cpp` include → 链接时报"重复定义"
>
> 例外：`inline` 函数和模板可以放头文件，因为编译器/链接器会自动去重。

### 静态链接 vs 动态链接

| | 静态链接 | 动态链接 |
|--|---------|---------|
| 文件 | `.lib` (Win) / `.a` (Linux) | `.dll` (Win) / `.so` (Linux) |
| 时机 | 编译时合并进 exe | 运行时加载 |
| exe 体积 | 大（包含库代码） | 小（库在外面） |
| 分发 | 只需 exe | 需要附带 dll |
| UE 中 | 引擎静态库部分 | 插件、第三方库 |

---

## 五、编译单元（Translation Unit）

### 概念

一个 `.cpp` 文件 + 它 include 的所有头文件 = 一个**编译单元**。

```
main.cpp ──┐
  #include "A.h" ──┐
  #include "B.h" ──┤── 合在一起 = 一个编译单元
  main.cpp 的代码 ──┘
```

### 关键规则

- 每个编译单元**独立编译**，互相不可见
- 编译器一次只看一个编译单元
- 这就是为什么要有**声明**（`.h`）和**定义**（`.cpp`）的分离

### ODR（One Definition Rule）— 一次定义规则

> [!important] C++ 最重要的规则之一
> - 每个函数/变量在**整个程序中只能定义一次**
> - 但可以**声明多次**（多个 .cpp 都能 include 同一个 .h）
> - 例外：`inline` 函数、模板、类定义（在每个编译单元中可以重复，但必须完全一致）

---

## 六、头文件的本质

头文件不是魔法，它就是**复制粘贴**：

```cpp
// Player.h
class Player {
    int hp;
public:
    void TakeDamage(int dmg);
};

// Player.cpp
#include "Player.h"   // 把 Player.h 的内容粘贴到这里
void Player::TakeDamage(int dmg) { hp -= dmg; }

// main.cpp
#include "Player.h"   // 又把 Player.h 粘贴了一份
int main() {
    Player p;
    p.TakeDamage(10);
}
```

### 头文件保护（Include Guard）

防止同一个头文件被粘贴多次：

```cpp
// 方式 1：传统宏保护
#ifndef PLAYER_H
#define PLAYER_H
// ... 内容 ...
#endif

// 方式 2：#pragma once（更简洁，UE 推荐）
#pragma once
// ... 内容 ...
```

---

## 七、声明 vs 定义

| | 声明（Declaration） | 定义（Definition） |
|--|-------|-------|
| 作用 | 告诉编译器"这个东西存在" | 真正创建这个东西 |
| 可以多次？ | 可以 | 只能一次（ODR） |
| 放哪里 | `.h` 文件 | `.cpp` 文件 |
| 例子 | `void foo();` | `void foo() { ... }` |
| 变量 | `extern int x;` | `int x = 42;` |
| 类 | `class Foo;`（前向声明） | `class Foo { ... };` |

> [!tip] 前向声明的用途
> 如果你只需要用指针/引用，不需要知道类的具体内容，用前向声明可以**减少头文件依赖、加快编译**：
> ```cpp
> class Player;  // 前向声明，不需要 include "Player.h"
> void Heal(Player* p);  // 只用了指针，够了
> ```

---

## 八、模板与编译的特殊关系

> [!warning] 这是模板的一个大坑

### 问题：模板为什么要放在头文件？

```cpp
// Box.h — 只有声明
template <typename T>
struct Box {
    T value;
    void set(T v);
};

// Box.cpp — 定义
template <typename T>
void Box<T>::set(T v) { value = v; }

// main.cpp
#include "Box.h"
Box<int> b;
b.set(42);  // 链接错误！undefined reference to Box<int>::set(int)
```

**原因**：
1. 编译 `main.cpp` 时，编译器看到 `Box<int>::set(42)`，需要 `Box<int>::set` 的代码
2. 但模板的定义在 `Box.cpp` 里，`main.cpp` 的编译单元**看不到**
3. 编译 `Box.cpp` 时，编译器不知道有人要用 `Box<int>`，所以**没有实例化**
4. 结果：没有人生成 `Box<int>::set` 的代码 → 链接时找不到

### 解决方案

```cpp
// 方案 1（最常见）：模板的声明和定义都放 .h 文件
// Box.h
template <typename T>
struct Box {
    T value;
    void set(T v) { value = v; }  // 直接在头文件定义
};

// 方案 2：显式实例化（大型项目用）
// Box.cpp
template <typename T>
void Box<T>::set(T v) { value = v; }

template struct Box<int>;     // 手动告诉编译器：生成 Box<int> 的代码
template struct Box<float>;   // 生成 Box<float> 的代码
// 缺点：你必须预先知道会用哪些类型
```

---

## 九、UE 的编译流程

UE 的编译比普通 C++ 多了额外步骤：

```
.h 文件（含 UCLASS 等宏）
    ↓ UHT (Unreal Header Tool)
生成 .generated.h 文件（反射代码）
    ↓ 预处理
展开宏 + 生成代码
    ↓ 编译
    ↓ 汇编
    ↓ 链接
.dll（模块）/ .exe（编辑器）
```

### UHT 做了什么

- 解析 `UCLASS()`, `UPROPERTY()`, `UFUNCTION()` 等标记
- 生成反射信息（让蓝图能认识 C++ 类）
- 生成序列化代码
- 生成网络复制代码

### UBT (Unreal Build Tool) 做了什么

- 管理模块依赖关系
- 决定编译顺序
- 管理 PCH（预编译头文件）
- 增量编译（只重编改过的文件）

---

## 十、实用知识：常见编译错误对照表

| 错误信息 | 阶段 | 原因 |
|---------|------|------|
| `unexpected token` | 语法分析 | 写错了语法（少分号、括号等） |
| `undeclared identifier` | 语义分析 | 用了没声明的变量/函数 |
| `cannot convert` | 语义分析 | 类型不匹配 |
| `undefined reference` | 链接 | 声明了但没定义/没链接库 |
| `multiple definition` | 链接 | 同一个东西定义了多次 |
| `unresolved external symbol` | 链接 (MSVC) | 同上，MSVC 的叫法 |
| `#include file not found` | 预处理 | 头文件路径不对 |

> [!tip] 判断错误属于哪个阶段
> - 报错提到**行号** → 编译阶段（预处理/编译）
> - 报错提到**符号名但无行号** → 链接阶段
> - UE 里报 `Unresolved external` → 检查 `.Build.cs` 有没有添加模块依赖

---

## 速查：完整流程图

```
你写的代码 (.cpp + .h)
        │
        ▼
┌─────────────────────────┐
│  ① 预处理 (Preprocessor)│  #include 展开、宏替换、条件编译
└─────────────────────────┘
        │ 生成 .i 文件
        ▼
┌─────────────────────────┐
│  ② 编译 (Compiler)      │  词法→语法→语义→优化→代码生成
│     - 模板在此实例化     │
│     - 类型检查在此完成   │
└─────────────────────────┘
        │ 生成 .s 文件（汇编）
        ▼
┌─────────────────────────┐
│  ③ 汇编 (Assembler)     │  汇编 → 机器码
└─────────────────────────┘
        │ 生成 .obj 文件
        ▼
┌─────────────────────────┐
│  ④ 链接 (Linker)        │  合并所有 .obj + .lib → 生成 exe/dll
│     - 符号解析           │
│     - undefined reference│ 在此报错
└─────────────────────────┘
        │
        ▼
   最终可执行文件 (.exe / .dll)
```

---

## 相关链接

- [[模板编程]] — 模板在编译阶段的实例化细节
- [[C++和计算机基础总览]]
