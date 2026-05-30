---
title: "C++ Day 1"
date: 2026-05-30
draft: false
---

# C++ Day 1

## std::cout 缓冲机制（Buffering）

`std::cout` 默认是**缓冲输出（buffered output）**。

输出内容不会立即显示到控制台，而是先写入一块内存区域（**缓冲区 Buffer**）：

```text
std::cout → Buffer → flush() → Console
```

缓冲的目的：

* 减少系统调用次数
* 批量输出，提高性能
* 因为向终端/设备输出通常比写内存慢得多

### 缓冲区何时刷新（flush）

常见刷新时机：

1. **缓冲区满**
2. **程序正常结束**
3. 使用 `std::endl`
4. 显式调用 `std::flush` / `std::cout.flush()`
5. 某些输入操作（如 `std::cin` 前）

### `'\n'` 与 `std::endl` 区别

```cpp
std::cout << "Hello\n";      // 换行，不保证立即刷新
std::cout << "Hello" << std::endl; // 换行 + 立即刷新
```

* `'\n'`：仅换行，性能更好
* `std::endl`：换行并强制刷新缓冲区

实际开发通常优先使用 `'\n'`，仅在需要**立即输出**时使用 `std::endl`。

### 注意：程序崩溃可能导致输出丢失

若程序在缓冲区刷新前崩溃：

```cpp
std::cout << "Before crash";
```

内容可能仍停留在 Buffer 中，因此不会显示。

解决方式：

```cpp
std::cout << "Before crash" << std::endl;
// 或
std::cout << "Before crash" << std::flush;
```

下面是适合直接贴博客的简要整理版：

## std::cin 缓冲机制（Buffered Input）

`std::cin` 与 `std::cout` 类似，也是**缓冲输入（buffered input）**。

输入过程分为两步：

```text
用户输入 → Input Buffer → std::cin >> 提取 → 变量
```

### 输入缓冲区（Input Buffer）

用户输入的字符会先进入 **输入缓冲区**。

例如输入：

```text
4 5
```

实际进入缓冲区的是：

```text
4 5\n
```

注意：**每次回车都会产生一个 `'\n'`（换行符）并存入缓冲区。**

输入缓冲区遵循 **FIFO（先进先出）** 原则。

---

### 基本提取流程（`>>`）

提取运算符 `>>` 的行为：

1. 检查 `std::cin` 状态是否正常。
2. 丢弃前导空白字符（空格、Tab、换行）。
3. 若缓冲区为空，则等待用户输入。
4. 提取连续有效字符，直到：

   * 遇到空白字符 / 换行
   * 遇到对目标类型无效的字符

---

### 示例：一次输入，多次提取

```cpp
int x{}, y{};

std::cin >> x;
std::cin >> y;
```

输入：

```text
4 5
```

缓冲区：

```text
4 5\n
```

执行过程：

```text
x 提取 4
缓冲区剩余：5\n

y 提取 5
```

因此第二次提取**无需再次等待输入**。

这就是 `std::cin` 缓冲的意义：**输入与提取分离，可以一次输入，多次读取。**

---

### 输入失败示例

#### 情况1：部分合法输入

输入：

```text
5a
```

缓冲区：

```text
5a\n
```

执行：

```cpp
int x{};
std::cin >> x;
```

结果：

```text
x = 5
缓冲区剩余：a\n
```

因为整数提取在遇到非法字符 `a` 时停止。

---

#### 情况2：非法输入

输入：

```text
b
```

结果：

```text
提取失败
x = 0（C++11 起）
std::cin 进入失败状态
```

之后所有输入都会立即失败，直到调用清除函数恢复输入流状态。

---

### 核心总结

```text
std::cin 本质：

用户输入
    ↓
Input Buffer
    ↓
operator>>
    ↓
变量
```

关键点：

* `std::cin` 是缓冲输入
* 输入数据按 FIFO 顺序处理
* 回车会产生 `'\n'`
* `>>` 默认跳过前导空白字符
* 一次输入可支持多次提取
* 提取失败会使输入流进入错误状态

## 头文件卫士、声明与定义

### 头文件卫士（Header Guard）

头文件卫士用于防止**同一个翻译单元（单个 `.cpp` 编译结果）**重复包含同一个头文件（本质是防止因为嵌套include引起的重复定义）。

常见写法：

a.h
```cpp
#ifndef A_H
#define A_H

class A{};

#endif
```

b.h
```cpp
#include "a.h"
```

main.cpp
```cpp
#include "a.h"
#include "b.h"
```
没有 guard 时：

main.cpp
 ├─ a.h
 └─ b.h
     └─ a.h

a.h 被展开两次。

注意：**Header Guard 不能阻止多个 `.cpp` 文件同时包含同一个头文件。**

因为：

```text
main.cpp  → 独立预处理/编译
other.cpp → 独立预处理/编译
```

每个 `.cpp` 都拥有自己的预处理环境，宏定义状态不会共享。

---

### 声明（Declaration）与定义（Definition）

#### 声明：告诉编译器“某东西存在”

例如：

```cpp
int add(int,int);
```

这是**函数声明**。

声明可以重复出现：

* 单个 `.cpp` 中允许重复声明（前提一致）
* 多个 `.cpp` 中允许重复声明

工程中通常：

```text
add.h
 └── 放声明

main.cpp
other.cpp
 └── include add.h
```

这种写法完全正常。

---

#### 定义：真正创建实体/实现代码

例如：

```cpp
int add(int a,int b)
{
    return a+b;
}
```

这是**函数定义**。

普通定义遵循 **ODR（One Definition Rule）**：

> 一个工程中通常只能有一个定义。

错误示例：

```cpp
// main.cpp
int x=5;

// other.cpp
int x=10;
```

编译可能通过，但链接时报错：

```text
multiple definition
```

---

### 工程中的标准组织方式

通常采用：

```text
xxx.h   → 声明

xxx.cpp → 定义/实现
```

示例：

```cpp
// add.h
int add(int,int);
```

```cpp
// add.cpp
#include "add.h"

int add(int a,int b)
{
    return a+b;
}
```

```cpp
// main.cpp
#include "add.h"
```

这样：

* 所有文件都能看到声明
* 实现只有一份
* 满足 ODR 规则
* 方便大型工程组织代码
