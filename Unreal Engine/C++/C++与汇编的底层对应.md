# C++ 与汇编的底层对应关系

> [!note] 前置知识
> 建议先阅读 [[C++编译原理]]，理解编译的四个阶段后再看本篇。

---

## 一、核心认知：C++ 的抽象在汇编中如何消失

C++ 提供了 class、模板、继承、多态等高级抽象，但到了汇编层面，这些全部被"打平"成最原始的操作：内存读写 + 跳转 + 函数调用。

| C++ 概念 | 汇编中变成 |
|---------|-----------|
| 局部变量 | 栈上的 `[rbp - 偏移]` |
| 全局变量 | `.data` / `.bss` 段的固定地址 |
| struct/class 定义 | **消失**（只是编译器记住布局，不生成代码） |
| 对象实例 | 一块连续内存（栈或堆） |
| 成员访问 `obj.x` | `[对象基地址 + 偏移量]` |
| 成员函数 | 普通函数 + 隐式 this 参数 |
| 虚函数调用 | 读 vptr → 读 vtable → 间接 call |
| if/else | `cmp` + 条件跳转 `jxx` |
| for/while | `cmp` + `jxx` + `jmp` 回跳 |
| new | `operator new`(malloc) + 构造函数 |
| delete | 析构函数 + `operator delete`(free) |
| 模板 | **消失**（编译期已展开成具体代码） |
| 引用 `&` | 和指针完全相同（就是地址） |
| 继承 | 子类内存前面包含父类部分 |

---

## 二、class/struct 定义 — 只是一张布局表

> [!important] 关键认知
> class 定义**不会生成任何机器码**，也**不占运行时内存**。它只是告诉编译器一张"布局表"：这个类型有多大，每个成员在第几个字节。

```cpp
struct Player {
    int hp;      // 偏移 0, 占 4 字节
    int mp;      // 偏移 4, 占 4 字节
    float speed; // 偏移 8, 占 4 字节
};
// 编译器记住：sizeof(Player) = 12, hp在+0, mp在+4, speed在+8
// 不生成任何汇编代码！
```

只有当你**创建实例**时，才会真正分配内存：

```cpp
Player obj;              // 栈上分配 12 字节
Player* p = new Player;  // 堆上分配 12 字节
```

### 对象内存布局

```
对象 obj 在内存中（起始地址假设为 0x1000）：

地址        内容         对应成员
0x1000      [4 bytes]    hp
0x1004      [4 bytes]    mp
0x1008      [4 bytes]    speed

一整块连续的 12 字节，仅此而已。
没有类名、没有成员名、没有函数——只有数据。
```

> [!tip] 函数存在哪？
> 成员函数的代码存在 `.text` 段（代码段），**不在对象里面**。所有 Player 对象共享同一份函数代码。对象里只存数据。

---

## 三、局部变量 — 栈上的偏移量

```cpp
void foo() {
    int a = 10;
    int b = 20;
    int c = a + b;
}
```

### 对应汇编（x64）

```asm
foo:
    push  rbp              ; 保存旧的栈帧基址
    mov   rbp, rsp         ; 建立新栈帧
    sub   rsp, 16          ; 在栈上留出空间（对齐到16字节）

    mov   dword ptr [rbp-4], 10    ; a = 10（a 住在 rbp-4）
    mov   dword ptr [rbp-8], 20    ; b = 20（b 住在 rbp-8）

    mov   eax, [rbp-4]            ; 把 a 读到寄存器 eax
    add   eax, [rbp-8]            ; eax = eax + b
    mov   dword ptr [rbp-12], eax  ; c = eax（c 住在 rbp-12）

    add   rsp, 16          ; 释放栈空间
    pop   rbp              ; 恢复旧栈帧
    ret                    ; 返回
```

> [!note] 变量名消失了
> 汇编里没有 `a`、`b`、`c` 的概念，全变成了 `[rbp-4]`、`[rbp-8]`、`[rbp-12]`。变量名只存在于调试信息（.pdb 文件）中。

---

## 四、成员访问 — 基地址 + 偏移

```cpp
struct Player {
    int hp;      // 偏移 0
    int mp;      // 偏移 4
    float speed; // 偏移 8
};

void damage(Player* p) {
    p->hp -= 10;
}
```

### 对应汇编

```asm
damage:
    ; rcx = p（第一个参数，Windows x64 调用约定）
    mov   eax, [rcx]         ; eax = *(p + 0) = p->hp
    sub   eax, 10            ; eax -= 10
    mov   [rcx], eax         ; 写回 p->hp
    ret
```

如果是访问 `p->speed`：

```asm
    movss xmm0, [rcx + 8]   ; 读取偏移 8 处的 float（speed）
```

> [!important] 核心规则
> `p->成员` 在汇编里就是 `[p的地址 + 该成员的偏移量]`。编译器在编译期就算好了每个成员的偏移，硬编码到汇编指令里。

---

## 五、成员函数 — 就是普通函数 + this

```cpp
struct Foo {
    int x;
    void set(int v) { x = v; }
};

Foo obj;
obj.set(42);
```

### 编译器的翻译

编译器会把成员函数改写成等价的普通函数：

```cpp
// 编译器内部的等价形式
void Foo_set(Foo* this, int v) {
    this->x = v;
}

// obj.set(42) 等价于
Foo_set(&obj, 42);
```

### 对应汇编

```asm
; void Foo::set(int v)
; rcx = this 指针, edx = v
Foo_set:
    mov   [rcx], edx      ; this->x = v（x 偏移为 0）
    ret

; 调用处: obj.set(42)
    lea   rcx, [rbp-8]    ; rcx = &obj（this 指针）
    mov   edx, 42         ; edx = v
    call  Foo_set          ; 调用
```

> [!note] 成员函数不在对象里
> 对象只存数据。函数代码在 `.text` 段里只有一份，所有 Foo 对象共享。`this` 指针告诉函数"你在操作哪个对象"。

---

## 六、虚函数 — vtable 间接调用

有虚函数时，对象会多一个隐藏成员 `vptr`（虚函数表指针）：

```cpp
struct Animal {
    virtual void speak() { printf("..."); }
    int age;
};

struct Dog : Animal {
    void speak() override { printf("Woof!"); }
};

Animal* a = new Dog;
a->speak();  // 调用的是 Dog::speak，不是 Animal::speak
```

### 内存布局

```
Animal 对象（或 Dog 对象）的内存：

偏移 0:   vptr (8字节)    ← 指向该类的虚函数表
偏移 8:   age  (4字节)    ← 成员数据
偏移 12:  padding (4字节) ← 对齐填充
sizeof = 16

注意：vptr 是编译器自动插入的，你看不到但它确实在那里。
```

### vtable（虚函数表）

```
Animal 的 vtable（全局只读数据，存在 .rodata 段）：
┌────────────────────────────┐
│ [0] → Animal::speak 的地址 │
└────────────────────────────┘

Dog 的 vtable：
┌────────────────────────────┐
│ [0] → Dog::speak 的地址    │  ← override 后指向 Dog 的版本
└────────────────────────────┘
```

### 虚函数调用的汇编

```asm
; a->speak();  (a 是 Animal* 类型)
    mov   rax, [rcx]         ; rax = *a = vptr（读对象前8字节）
    call  [rax]              ; 调用 vtable[0] 指向的函数
    ; 如果实际是 Dog 对象 → vtable[0] = Dog::speak → 调用 Dog::speak
    ; 如果实际是 Animal 对象 → vtable[0] = Animal::speak
```

> [!important] 多态的代价
> - 每个有虚函数的对象多占 8 字节（一个 vptr）
> - 每次虚函数调用多一次间接寻址（读 vtable）
> - 虚函数调用无法被内联优化（编译期不知道调哪个版本）
>
> 这就是 UE 中 `FORCEINLINE` 和非虚函数存在的意义——性能敏感的热路径避免虚函数。

---

## 七、if/else — 条件跳转

```cpp
int abs_val(int x) {
    if (x < 0)
        return -x;
    return x;
}
```

### 对应汇编

```asm
abs_val:
    test  edi, edi        ; 检查 x（设置标志位）
    jns   .positive       ; Jump if Not Sign（非负则跳）
    neg   edi             ; x = -x
.positive:
    mov   eax, edi        ; 返回值放 eax
    ret
```

### 跳转指令速查

| 指令 | 含义 | 对应C++条件 |
|------|------|------------|
| `je` / `jz` | Jump if Equal / Zero | `==` |
| `jne` / `jnz` | Jump if Not Equal | `!=` |
| `jl` | Jump if Less | `<`（有符号） |
| `jg` | Jump if Greater | `>`（有符号） |
| `jle` | Jump if Less or Equal | `<=` |
| `jge` | Jump if Greater or Equal | `>=` |
| `ja` | Jump if Above | `>`（无符号） |
| `jb` | Jump if Below | `<`（无符号） |

---

## 八、for 循环 — 计数器 + 条件跳转 + 回跳

```cpp
int sum(int n) {
    int result = 0;
    for (int i = 0; i < n; i++) {
        result += i;
    }
    return result;
}
```

### 对应汇编

```asm
sum:
    xor   eax, eax        ; result = 0（xor 自身 = 清零，比 mov 快）
    xor   ecx, ecx        ; i = 0
.loop:
    cmp   ecx, edi        ; 比较 i 和 n
    jge   .done           ; 如果 i >= n，跳出循环
    add   eax, ecx        ; result += i
    inc   ecx             ; i++
    jmp   .loop           ; 无条件跳回循环头部
.done:
    ret                   ; 返回 eax（result）
```

### 循环结构的通用模式

```
    初始化
.loop_start:
    比较条件
    条件不满足 → 跳到 .loop_end
    循环体
    更新计数器
    jmp .loop_start
.loop_end:
```

---

## 九、new / delete — malloc + 构造 / 析构 + free

```cpp
struct Foo {
    int x;
    Foo(int v) : x(v) {}
    ~Foo() { x = 0; }
};

Foo* p = new Foo(42);
delete p;
```

### 对应汇编

```asm
; === new Foo(42) ===
    mov   ecx, 4              ; sizeof(Foo) = 4 字节
    call  operator_new        ; 分配堆内存（底层调用 malloc）
    ; rax = 分配到的地址
    mov   rcx, rax            ; this = rax
    mov   edx, 42             ; 构造参数
    call  Foo::Foo            ; 调用构造函数
    mov   [rbp-8], rax        ; p = rax

; === delete p ===
    mov   rcx, [rbp-8]        ; rcx = p
    call  Foo::~Foo           ; 先调用析构函数
    mov   rcx, [rbp-8]        ; rcx = p
    call  operator_delete     ; 释放内存（底层调用 free）
```

> [!note] new 做了两件事
> 1. 分配内存（`operator new` → `malloc`）
> 2. 在分配好的内存上调用构造函数
>
> delete 也是两步，但顺序相反：先析构，再释放内存。

---

## 十、继承 — 子类包含父类的内存

```cpp
struct Base {
    int a;        // 偏移 0
    int b;        // 偏移 4
};

struct Derived : Base {
    int c;        // 偏移 8（紧接在 Base 后面）
    float d;      // 偏移 12
};
```

### 内存布局

```
Derived 对象的内存：
偏移 0:   a (4B)   ← 来自 Base
偏移 4:   b (4B)   ← 来自 Base
偏移 8:   c (4B)   ← Derived 自己的
偏移 12:  d (4B)   ← Derived 自己的
sizeof(Derived) = 16

Base* ptr = &derived_obj;
// ptr 指向偏移 0，正好是 Base 部分的开头
// 所以 ptr->a 和 ptr->b 都能正确访问
```

> [!tip] 继承的本质
> 子类对象 = 父类内存布局 + 子类新增成员，连续排列。
> 向上转型（子类指针 → 父类指针）不需要任何运行时操作，因为父类部分就在对象的开头。

---

## 十一、引用 vs 指针 — 汇编完全相同

```cpp
void by_pointer(int* p) { *p = 10; }
void by_reference(int& r) { r = 10; }
```

### 汇编对比

```asm
by_pointer:
    mov   dword ptr [rcx], 10    ; *p = 10
    ret

by_reference:
    mov   dword ptr [rcx], 10    ; r = 10（一模一样！）
    ret
```

> [!important] 引用的真相
> 引用在底层就是指针。区别只存在于 C++ 的语法和类型系统层面（引用不能为 null、不能重新绑定），到了机器码层面**完全没有区别**。

---

## 十二、如何在 Visual Studio 中查看反汇编

### 操作步骤

1. 在代码上**设一个断点**（F9）
2. **调试运行**（F5）
3. 程序停在断点后，菜单：**调试 → 窗口 → 反汇编**（或按 `Ctrl+Alt+D`）
4. 你会看到 C++ 源码和汇编指令交替显示

### 推荐设置

- 右键反汇编窗口 → 勾选 **显示源代码**
- 勾选 **显示符号名**（函数名代替裸地址）
- 勾选 **显示代码字节**（可以看到机器码的十六进制）

### 编译配置对结果的影响

| 配置 | 汇编特点 |
|------|---------|
| Debug (`/Od`) | 没有优化，变量全在栈上，一一对应好理解 |
| Release (`/O2`) | 大量优化，变量可能在寄存器里，循环可能被展开/消除 |

> [!tip] 初学建议
> 先用 **Debug 模式** 看反汇编（对应关系清晰），等熟悉后再切 Release 看编译器做了哪些优化。

### 在线工具推荐

**[Compiler Explorer (godbolt.org)](https://godbolt.org/)**
- 左边写 C++，右边实时看汇编
- 支持 GCC / Clang / MSVC
- 可以切换优化等级
- 鼠标悬停高亮对应关系
- 支持多文件

---

## 十三、x64 调用约定速查（Windows）

在 Windows x64 下（MSVC 编译器）：

| 参数位置 | 整数/指针 | 浮点 |
|---------|----------|------|
| 第 1 个参数 | `rcx` | `xmm0` |
| 第 2 个参数 | `rdx` | `xmm1` |
| 第 3 个参数 | `r8` | `xmm2` |
| 第 4 个参数 | `r9` | `xmm3` |
| 第 5+ 个参数 | 栈上 | 栈上 |
| 返回值 | `rax` | `xmm0` |

> [!note] this 指针
> 成员函数的 `this` 指针通过 `rcx` 传递（占用第一个参数位置），其余参数依次后移。

---

## 十四、常用寄存器速查

| 寄存器 | 用途 |
|--------|------|
| `rax` | 返回值、临时计算 |
| `rcx` | 第 1 参数 / this 指针 |
| `rdx` | 第 2 参数 |
| `r8, r9` | 第 3、4 参数 |
| `rsp` | 栈顶指针（Stack Pointer） |
| `rbp` | 栈帧基址（Base Pointer） |
| `rip` | 指令指针（下一条要执行的指令地址） |
| `xmm0-xmm3` | 浮点参数/返回值 |

### 寄存器大小

```
rax  (64位, 8字节)   ← 完整寄存器
eax  (低32位, 4字节) ← int 通常用这个
ax   (低16位, 2字节)
al   (低8位, 1字节)
```

---

## 相关链接

- [[C++编译原理]] — 编译的四个阶段
- [[模板编程]] — 模板如何在编译期实例化
- [[C++和计算机基础总览]]
