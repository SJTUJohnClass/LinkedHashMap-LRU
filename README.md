# Project 1: LinkedHashMap-LRU

> SJTU CS1966-01 2026Spring 第一次大作业
> 
> 

# 1. 内容概述

本次作业要求实现 HashMap、LinkedHashMap 两种数据结构，并基于这两种结构完成 LRU（Least Recently Used，最近最少使用）算法。三种组件的核心功能的详细要求，将在「任务说明」部分具体展开。

**作业限制**：禁止使用以下头文件及具备类似功能的头文件（需自行实现相关功能）：

```cpp
#include <unordered_map>
#include <map>
```

# 2. 文件说明

## 2.1 编码说明

所有文件均采用 UTF-8 编码。若添加中文注释后，使用 Dev-C++ 打开出现乱码，可按以下步骤重新编码：

1. 右击目标文件 → 选择「打开方式」→ 用「记事本」打开；

2. 点击记事本菜单栏「文件」→「另存为」；

3. 编码选择「GB18030」，保存后关闭；

4. 重新用 Dev-C++ 打开，中文注释即可正常显示。

## 2.2 各文件功能介绍

注：标注「无需修改」的文件，仅用于提供基础功能支持，请勿改动其代码，避免影响程序运行。

### 2.2.1 class-integer.hpp（无需修改）

实现了整数类 `Integer`，包含静态变量 `counter`，用于记录该类对象的构造与析构次数。**程序正常结束时，counter 的值应该为 0**（检测内存是否泄漏）。

使用示例：

```cpp
Integer a = Integer(1);
Integer *p = new Integer(2);
std::cout << a.val << " " << p->val << std::endl;
```

### 2.2.2 class-matrix.hpp（无需修改）

实现了矩阵类 `Matrix`，支持指定行数、列数和初始元素值构造矩阵。

使用示例：

```cpp
Matrix<int> a = Matrix<int>(1, 2, 3);
Matrix<int> *p = new Matrix<int>(1, 2, 3);
std::cout << a << " " << *p << std::endl;
```

上述代码将构造一个 1 行 2 列、所有元素均为 3 的矩阵。

### 2.2.3 exceptions.hpp（无需修改）

提供调试相关的异常处理支持，可根据需求选择使用；若无需复杂异常处理，使用 C++ 标准 `throw` 语句即可。

`throw` 字符串使用示例：

```cpp
try {
    throw "have a try";
} catch (const char* c) {
    std::cout << c << std::endl;
}
```

以上代码会输出 `have a try` 并且在 `throw` 后立即终止程序。

```cpp
throw "have a try";
```

以上代码会出现 `terminate called after throwing an instance of 'char const*'` 并且程序会停顿几秒后才终止。

### 2.2.4 lru.hpp (Todo)

本次作业的核心实现文件，需完成以下四个类/结构的实现：

- `sjtu::double_list<T>`：双向链表，用于维护元素顺序；

- `sjtu::hashmap<Key, T, Hash, Equal>`：哈希表，实现键值对的插入、查询、删除；

- `sjtu::linked_hashmap<Key, T, Hash, Equal>`：继承自 hashmap，额外维护元素插入顺序；

- `sjtu::lru`：基于 linked_hashmap 实现 LRU 算法。这个类没有默认构造函数，构造时必须给定 size 参数。

**注意**：linked_hashmap 是 hashmap 的派生类，在模板派生类中调用基类成员函数时，需显式使用 `this` 指针（如 `this->insert()`）。

### 2.2.5 utility.hpp（无需修改）

实现 pair 类，用于存储键值对（如 hashmap 中的 Key-Value 对）。

使用示例：

```cpp
sjtu::pair<Integer, Matrix<int>> a(Integer(1), Matrix<int>(2, 2, 2));
sjtu::pair<Integer, Matrix<int>> p = new sjtu::pair<Integer, Matrix<int>>;
```

## 2.3 类结构关系

各核心类的继承/依赖关系如下（清晰展示实现层级）：

```Plain Text
lru
 └── linked_hashmap （继承+依赖）
      └── hashmap    （基类）
      └── double_list（依赖，维护顺序）
```

## 2.4 代码实现补充细节

### 2.4.1 模板默认参数（Hash = ...）

模板参数中 `Hash = ...` 表示该参数为默认模板参数：

- 若使用时不显式指定该参数，将自动使用默认值；

- 若需自定义哈希规则（如针对 Integer 等自定义类型），需显式指定 Hash 类型并实现哈希函数。

自定义 Hash 示例（以 Integer 类为例）：

```cpp
class Hash {
public:
    unsigned int operator()(Integer lhs) const {
        return std::hash<int>()(lhs.val); // 以 Integer 的 val 作为哈希依据
    }
};
```

`Equal = ...` 是同理的。

### 2.4.2 Hash / Equal 的使用方式

Hash 和 Equal 类的使用有两种常见方式，均可正确调用其重载的 () 运算符：

```cpp
Hash h;
int v1 = h(key);       // 方式1：创建对象后调用
int v2 = Hash()(key);  // 方式2：创建临时对象并调用
```

### 2.4.3 迭代器相关注意事项

解引用空指针或无效指针会导致未定义行为，因此，在实际编程中，应该始终确保在对迭代器进行解引用之前，检查迭代器是否有效。

在 C++ 的 STL（标准模板库）中，对于迭代器解引用时指向容器末尾的情况，通常不会提供特定的保护或处理方式。这是因为 STL 通常遵循零成本抽象（zero-overhead abstraction）的原则，不会为不常见或异常情况提供额外的开销或处理逻辑，因此在使用unordered map的时候遇到这种情况会无事发生。

而本次作业明确要求：解引用空指针或无效迭代器属于 **未定义行为**，出现此类情况时，**必须抛出异常**（抛出类型不做强制要求）。

而对于 insert 之后 hashmap 扩容的情况，remove 一个迭代器，这个迭代器不能继续使用的情况等，交给用户(测试代码)即可。

### 2.4.4 测试代码说明

提供的测试代码仅为 OJ 测试用例的子集，仅用于初步验证代码正确性，**无法覆盖所有测试点**。建议自行补充测试用例，避免遗漏边界情况。

### 2.4.5 LinkedHashMap 与 LRU 的核心区别

两者的核心差异在于「元素顺序的改变时机」，这是实现时的关键区分点：

- LinkedHashMap：**仅在插入元素时**改变元素顺序（维护插入顺序）；

- LRU：**插入和查询元素时**都会改变元素顺序（维护最近使用顺序）。

### 2.4.6 必须抛出异常的场景

以下三种情况，必须抛出异常（抛出类型不做强制要求），否则会导致测试不通过：

1. 迭代器非法移动（如 `begin()--`、`end()++`）；

2. 对指向空对象的迭代器解引用（如 `*end()`）；

3. 使用 `[]` 或 `at` 方法访问时下标越界或不存在。

## 2.5 杂项说明

1. linked_hashmap 可直接调用基类 hashmap 的所有成员函数（需注意模板参数匹配），因此基本只需要额外维护子类的双向链表即可；

2. double_list 仅需实现普通迭代器 (iterator)，无需实现常量迭代器 (const_iterator)；

3. sjtu::lru::print() 函数需严格遵循以下输出格式（否则可能导致测试点扣分）：

```cpp
void print() {
    sjtu::linked_hashmap<Integer, Matrix<int>, Hash, Equal>::iterator it;
    for (it = mem->begin(); it != mem->end(); it++) {
        std::cout << (*it).first.val << " "
                  << (*it).second << std::endl;
    }
}
```

4. 建议提交前用 valgrind 检测内存泄漏，推荐参数如下（复制即可使用）：

```bash
'--tool=memcheck',
'--leak-check=full',
'--exit-on-first-error=yes',
f'--error-exitcode=250',
'--quiet',
```

5. lru 类的成员函数后若添加了 const 修饰，可根据自身实现情况去掉（若成员变量未使用指针，通常可去掉）。

6. 你的 double_list 类的迭代器应该支持 `it = end()--` 之类的操作。容器为空时应当满足 `begin() == end()`。

# 3. 核心概念与任务说明

本次作业分为 3 个 Subtask，按顺序实现即可（后一个 Subtask 依赖前一个的实现），具体要求如下：

## Subtask 1: 实现 HashMap

LinkedHashMap 本质是“支持插入顺序访问”的 HashMap，因此需先实现基础的 HashMap（哈希表）功能。

### 3.1.1 核心概念

- 哈希表：存储键值对（Key-Value），通过 Key 快速索引 Value，核心优势是插入、查询、删除的平均时间复杂度为 O(1)；

- 键值对规则：同一个 Key 最多对应一个 Value，插入相同 Key 时，新 Value 会覆盖旧 Value；

- 哈希值：通过哈希函数（如 std::hash）将 Key 映射到一个整数，用于定位键值对在哈希表中的存储位置；

- 哈希碰撞：不同 Key 可能通过哈希函数得到相同哈希值，需选择合适的碰撞解决方法（如开链表法、线性探测法等）。

### 3.1.2 实现要求

在数据规模未知时，固定大小的 Hashmap 无法适配，因此需要实现**动态大小哈希表**，核心要点如下：

1. 核心参数：容量（Capacity，哈希表数组大小）和负载因子（LoadFactor，元素个数与容量的最大允许比例）；

2. 动态扩容：初始容量可设为较小值，当元素个数超过 **「容量 × 负载因子」** 时，扩大容量（如翻倍），保证效率与空间利用率；

3. 时间复杂度要求：所有成员函数（插入、查询、删除等）的平均时间复杂度为 O(1)；构造函数、clear 方法的平均时间复杂度也需满足 O(1)（参考 vector 动态扩容的时间复杂度计算方式）。

## Subtask 2: 实现 LinkedHashMap

在 HashMap 功能的基础上，额外维护元素的**插入顺序**，支持按插入顺序访问元素。

### 3.2.1 实现思路

通过维护一个**双向链表（double_list）**，实现插入顺序的记录：每次插入键值对时，除了将其插入 HashMap 中，同时将该键值对插入到双向链表的末尾；访问元素时，按双向链表的顺序遍历即可。

### 3.2.2 实现要求

以下三种操作的期望时间复杂度均需为 O(1)（均摊意义下）：

1. 插入键值对；

2. 通过 Key 查询对应的 Value；

3. 按插入顺序访问所有元素。

## Subtask 3: 实现 LRU Algorithm

基于 LinkedHashMap 实现 LRU (Least Recently Used) 算法，核心是实现“最近最少使用”的内存淘汰策略。

### 3.3.1 核心概念

LRU 是一种内存管理策略，适用于“内存容量有限”的场景：当内存已满，需淘汰「最近最少被使用」（“使用”指被插入或者被查询）的键值对，为新数据腾出空间。

核心参数：预设的内存大小 size（最多可存储的键值对数量）。

### 3.3.2 实现要求

#### 1. 插入操作（save）

将键值对 (K, V) 存入内存，执行逻辑如下：

- 首先查找内存中是否存在 Key = K：
        

    - 存在：更新该 Key 对应的 Value；

    - 不存在：检查内存是否已满（元素个数是否大于等于 size）：
                

        - 未满：直接将 (K, V) 存入内存；

        - 已满：查找最早被插入或被查询的键值对 (K', V')，将该键值对替换为 (K, V)。

#### 2. 查询操作（get）

通过 Key 查找内存中的对应元素，执行逻辑如下：

- 利用 HashMap 的哈希查找功能，判断 Key 是否存在；

    - 不存在：返回空指针；

    - 存在：返回指向该 Value 的指针。

# 4. 评分规则与截止时间（DDL）

## 4.1 提交要求

请于 **3 月 30 日** 前，将代码提交至 ACMOJ 进行评分。

## 4.2 评分构成（总分 100 分）

- ACMOJ 数据点得分：90 分（按测试用例通过率计分）；

- CR（代码评审）得分：10 分（按代码规范、可读性、内存管理等维度计分）。

## 4.3 分值分配

- HashMap：30%；

- LinkedHashMap：40%；

- LRU：30%。

# 5. Acknowledgement

感谢 2022 级蒋捷学长开发本作业题目。

感谢 2024 级龚王博学长和陈奕莱学长修缮本 README 文档 & 添加测试点。

如有作业相关问题，请联系项目发布者 `arccc`，邮箱地址：`ocwzazure@sjtu.edu.cn`。