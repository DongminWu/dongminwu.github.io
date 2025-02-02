---
layout: post
title: Dynamo Deep Dive 01
date: 2025-01-30
categories: DynamoDeepDive
---

# 从零理解Pytorch Dynamo

在 PyTorch 的生态系统中，TorchDynamo（简称 Dynamo）是一个非常重要的组件。它不仅是 torch.compile 的核心，也是许多复杂错误的“罪魁祸首”。然而，Dynamo 的复杂性和灵活性让它成为了一个值得深入探讨的话题。本文将带你从零开始，逐步揭开 Dynamo 的神秘面纱，帮助你更好地理解它的工作原理和实际应用。
## 什么是 Dynamo？为什么要用 Dynamo？
**由一个问题说开**
PyTorch 以其动态计算图和灵活性深受开发者喜爱，但动态特性也带来了性能瓶颈。传统静态图编译器（如 TorchScript）需要用户修改代码，牺牲灵活性。
假设我们需要实现一个函数，根据输入张量的求和结果动态选择计算路径：

```python
def dynamic_flow(x):
    if x.sum() > 0:  # 动态条件，依赖输入值
        return x * 2
    else:
        return x + 1
```
**TorchScript 的限制**
TorchScript 要求静态化控制流，无法直接处理动态条件：

```python
import torch

@torch.jit.script  # 直接装饰会报错！
def dynamic_flow_torchscript(x: torch.Tensor) -> torch.Tensor:
    if x.sum() > 0:  # 触发错误：动态条件无法静态分析
        return x * 2
    else:
        return x + 1
```
以上代码会输出如下报错信息：
```
TracerWarning: Converting a tensor to a Python boolean might cause the trace to be incorrect.
```

解决方法：用户需手动将动态条件转换为静态可追踪形式，例如通过掩码操作：

```python
@torch.jit.script
def dynamic_flow_torchscript_fixed(x: torch.Tensor) -> torch.Tensor:
    # 用张量操作代替动态条件分支
    mask = (x.sum() > 0).float()
    return mask * (x * 2) + (1 - mask) * (x + 1)
```
用户被迫牺牲代码的可读性，以适应静态图编译器的限制。
**Dynamo 的解决方法**

Dynamo 无需修改代码，直接编译原生 Python 逻辑：

```python
@torch.compile  # 一行装饰，无需改动
def dynamic_flow_dynamo(x):
    if x.sum() > 0:
        return x * 2
    else:
        return x + 1

# 正常运行，自动生成两个子图（对应 if/else 分支）
x1 = torch.tensor([1.0, 2.0])  # sum=3 >0 → 触发乘法分支
x2 = torch.tensor([-1.0, -2.0]) # sum=-3 <0 → 触发加法分支

print(dynamic_flow_dynamo(x1))  # 输出: tensor([2., 4.])
print(dynamic_flow_dynamo(x2))  # 输出: tensor([0., -1.])
```

## TorchDynamo 如何工作？
### 目标 1: 捕获为静态图
**PEP 523 的帧评估 API 实现 FX 图**
Dynamo 钩入 CPython 的帧评估 API（PEP 523），在 Python 字节码执行之前对其进行动态修改。
- **CPython 钩子注入**
当使用 @torch.compile 装饰的函数被调用时，Dynamo 通过 PEP 523 注册一个自定义帧解释器，接管 CPython 对目标函数的执行权。
关键数据捕获
Dynamo 通过拦截和分析 Python 字节码的执行过程来构建中间表示（IR），逐条理解和处理 Python 代码中的操作，构建起对代码执行路径的追踪。例如，当程序中进行张量操作时，Dynamo 会把相关的操作记录下来。
- **构建符号化中间表示（IR）**
Dynamo 将 Python 对象封装为 VariableTracker 子类，形成统一的符号化表示：
    - TensorVariable：跟踪 PyTorch 张量操作
    - ListVariable：处理列表结构（记录元素间的依赖关系）
    - ConstantVariable：标记不可变常量（如整数、字符串）
    - UserDefinedObjectVariable：兜底类型（处理未特殊定义的对象）
- **对象包装逻辑**
通过 VariableBuilder._wrap 递归解析 Python 对象：

```python
def _wrap(obj):
    if isinstance(obj, torch.Tensor):
        return TensorVariable(obj)
    elif isinstance(obj, list):
        return ListVariable([_wrap(e) for e in obj])
    # ... 其他类型的匹配
```
确保所有操作可被 Dynamo 的 IR 追踪。
- **符号化执行字节码**
InstructionTranslatorBase 类实现 Python 字节码的符号执行逻辑，覆盖约 200 个指令：
示例：BUILD_LIST 指令处理

```python
def BUILD_LIST(self, inst):
    items = self.popn(inst.argval)  # 弹出栈顶 N 个元素
    self.push(ListVariable(items))  # 构造 ListVariable 压入堆栈
```
- **动态状态维护**
维护符号堆栈、变量作用域、控制流跳转等状态，完全模拟 CPython 行为。
- **PyTorch 操作拦截**
当执行到 `torch.add` 等张量运算时：
操作参数被转换为 TensorVariable 对象
生成对应的 `fx.Proxy` 节点（封装 `fx.Node`）
通过 OutputGraph 将节点记录到 FX 图中
- **FX 图生成机制**
Proxy 对象驱动建图，所有张量操作通过 `fx.Proxy` 记录到 FX 图：

```python
def wrap_fx_proxy(proxy):
    def wrapper(*args, **kwargs):
        # 记录操作到 FX 图
        return OutputGraph.create_proxy('call_function', torch.add, args, kwargs)
    return wrapper
```
- **节点类型**：包括 call_function、call_method、call_module 等
- **数据流追踪**：符号执行过程中，通过 VariableTracker 的依赖关系自动构建计算图边（edges）。
- **回退到 Python 完成编译**
帧评估 API 的 C 层捕获数据后，将控制权交还给 Dynamo 的 Python 代码。
- **图优化与输出**
生成的 FX 图经过一系列优化（如算子融合、无效代码消除），最终交给 TorchInductor 等后端编译为高效代码。
**关键调试手段**
- **日志分析**
使用 TORCH_LOGS=dynamo 查看字节码执行轨迹：
TRACE LOAD_GLOBAL y [TorchInGraphFunctionVariable(<built-in method any>), TensorVariable()]
确认对象是否被正确追踪为预期的 VariableTracker 类型。
- **类型推断检查**
当出现 UserDefinedObjectVariable 误包装时，需检查 VariableBuilder._wrap 的分支逻辑，补充特定类型的处理规则。


**线性化追踪**
Dynamo 的核心功能是将函数的执行过程线性化。这意味着它会将函数中的所有操作按顺序记录下来，忽略任何控制流（如条件语句或循环）。例如：

```python
import torch

@torch.compile
def mse(x, y):
    z = (x - y) ** 2
    return z.sum()

x = torch.randn(200)
y = torch.randn(200)
mse(x, y)
```

通过设置环境变量 TORCH_LOGS=graph_code，我们可以看到 Dynamo 生成的图：

```python
def forward(l_x_: torch.Tensor, l_y_: torch.Tensor):
    sub = l_x_ - l_y_
    z = sub ** 2
    sum_1 = z.sum()
    return (sum_1,)
```
可以看到，Dynamo 将 z = (x - y) ** 2 拆分成了两个操作：sub = l_x_ - l_y_ 和 z = sub ** 2。这种线性化的表示方式使得后续的优化和编译变得更加简单。

**控制流的处理**
Dynamo 会忽略函数中的控制流，只记录实际执行的操作。例如：


```python
import torch

@torch.compile
def fn(x, n):
    y = x ** 2
    if n >= 0:
        return (n + 1) * y
    else:
        return y / n

x = torch.randn(200)
fn(x, 2)
```
生成的图如下：

```python
def forward(l_x_: torch.Tensor):
    y = l_x_ ** 2
    mul = 3 * y
    return (mul,)
```
可以看到，Dynamo 完全忽略了 if 语句，只记录了实际执行的操作。这是因为 Dynamo 的图是基于输入的具体值生成的，因此图的生成依赖于输入。
### 目标 2: 保证动态性
#### Guards
我们先讨论下什么是动态特性:
动态特性 (Dynamic Behavior) 的本质：动态特性指 Python 程序在运行时可灵活改变行为的能力，典型表现为：
动态类型：变量类型在运行时可变（如 x 初始为 int，后变为 str）
数据依赖控制流：if/for 分支条件依赖输入值（如 if x > 0: 的执行路径由 x 决定）
动态数据结构：列表/字典内容在运行时变化（如 list.append 动态扩展长度）
反射与元编程：通过 eval()、getattr() 等动态操作代码结构
这些特性使得 Python 程序灵活，但给静态编译（如 PyTorch 的图生成）带来挑战——无法提前确定程序所有可能的执行路径。

**Guards 的核心机制**
条件收集
在符号化执行（Symbolic Execution）过程中，Dynamo 自动收集所有影响程序行为的条件：
    - 类型约束：___check_type_id(L['b'], 94334122025024)（验证变量 b 的类型）
    - 值约束：L['b'] == 'Hello'（验证变量 b 的值）
    - 结构约束：len(L['l']) == 2（验证列表长度）
示例：

```python
@torch.compile
def fn(a, b):
    return a * len(b)
```
输入 b="Hello" 时生成 Guards：

```python
___check_type_id(L['b'], 94334122025024)  # 类型为 str
L['b'] == 'Hello'                         # 值为 "Hello"
```
**条件溯源 (Sources)**
Sources 追踪变量的来源路径，确保 Guards 精确绑定到输入对象：
    - LocalSource：局部变量（如函数参数 x）
    - GlobalSource：全局变量（如模块级常量）
    - GetItemSource：容器元素（如 y[0] 追踪到 y 是局部变量）
示例：

```python
def fn(x, y: list):
    return x * y[0]
```

y[0] 的 Source 为 GetItemSource(LocalSource('y'), 0)
生成的 Guard：___check_type_id(L['y'][0], Tensor)（验证 y[0] 是张量）
缓存验证
每次函数调用时，Dynamo 检查新输入是否满足所有 Guards：
满足条件 → 复用缓存的 FX 图
不满足条件 → 触发重新编译并生成新 Guards
示例：

```python
fn(torch.randn(8), ["Hi", "Hello"])  # 触发编译，生成 Guards
fn(torch.randn(8), ["Hi"])           # `len(L['l']) == 2` 失败 → 重新编译
```

**Guards 如何保证动态性**

**保留动态控制流**
允许函数内部使用 if/for，只要分支条件被 Guards 覆盖：

```python
@torch.compile
def fn(x):
    if x.sum() > 0:  # Guard: x.sum() > 0
        return x * 2
    else:
        return x + 1
```
输入 x1（满足 sum > 0）生成 x * 2 的图
输入 x2（不满足）生成 x + 1 的图
**支持动态数据结构**

```python
@torch.compile
def fn(x, lst):
    return x * len(lst)  # Guard: len(lst) == N
```
输入 lst1（长度 3）生成 x * 3 的图
输入 lst2（长度 5）触发重新编译，生成 x * 5 的图
**兼容反射操作**

```python
@torch.compile
def fn(obj, attr_name):
    value = getattr(obj, attr_name)  # Guard: hasattr(obj, attr_name)
    return value * 2
```
输入 obj1 含属性 "weight" 生成对应图
输入 obj2 不含该属性 → 触发回退到 Python 解释器
目标 3: 支持动态 Shape
什么是动态 Shape？
动态 Shape 是指在模型编译（如 PyTorch 的 torch.compile）过程中，某些张量维度的大小（如 batch_size、序列长度等）不是固定值，而是可以接受不同的输入值。例如：

```python
# 输入 a 的 shape[0] 可能是 4 或 8，但模型需要支持动态变化
a = torch.randn(4, 3)
b = torch.randn(8, 3)
```
Dynamo 定义的动态 Shape 通过**符号整数（SymInt）**实现。SymInt 看起来像普通整数，但会在计算图中记录操作，生成一个能适配不同输入大小的通用计算图。
Dynamo 是如何实现的
默认静态，按需动态
Dynamo 在构建图时，会根据调用时的实际输入值来构建对应的 IR。如果某个 Shape 属性（如张量的某个维度大小）在初始调用时是固定值，并且在后续调用中这个值没有发生变化，那么它在这个图中可能表现为静态的。而一旦调用时 Shape 属性的值发生了变化，并且这个变化影响到计算图的构建，Dynamo 就会启用 SymInt 来符号化表示这个动态 Shape，以便生成一个通用的、适用于不同 Shape 值的计算图。例如：

```python
@torch.compile
def fn(a, b):
    return a.shape[0] * a * b

# 第一次调用：输入 shape 为 (4, 3)
fn(torch.randn(4, 3), torch.randn(4, 3))  # 生成静态图，shape[0]=4

# 第二次调用：输入 shape 为 (8, 3)
fn(torch.randn(8, 3), torch.randn(8, 3))  # 触发动态 Shape，生成符号化图
```
输出图的变化：

```python
# 静态图（第一次调用）
def forward(l_a_, l_b_):
    mul = 4 * l_a_  # 4 是固定值
    return mul * l_b_

# 动态图（第二次调用）
def forward(s0, l_a_, l_b_):
    getitem = l_a_.size()[0]  # 符号化为 s0
    mul = getitem * l_a_      # 使用符号 s0
    return mul * l_b_
```
**符号整数（SymInt）**
当 Dynamo 检测到形状发生变化时，它会使用 SymInt 来表示这些形状。SymInt 会记录所有在其上执行的操作，并将这些操作记录在生成的 FX 图中。例如：

```python
x = a.shape[0]  # x 是 SymInt 类型（如 s0）
y = x * 2       # 记录操作：mul(s0, 2)
```
当输入 Shape 变化时，Dynamo 会插入守卫（Guard）检查条件，决定是否触发重新编译。例如：

```python
# 第一次调用后的 Guard（静态）
check_tensor(L['a'], size=[4, 3])

# 第二次调用后的 Guard（动态）
check_tensor(L['a'], size=[None, 3])  # None 表示动态维度
2 <= L['a'].size()[0]                # 防止维度为 0 或 1
```
**0 和 1 的特例处理**
即使标记为动态维度，如果实际值是 0 或 1，Dynamo 会强制转为静态。例如：

```python
@torch.compile
def fn(a):
    return a + 1

fn(torch.randn(0, 3))  # 输入 shape 为 (0, 3)
```
守卫：

```python
L['a'].size()[0] == 0  # 强制特例化处理
```
原因：
空张量（维度为 0）需要特殊处理（如避免非法内存访问）。
维度为 1 时可能影响张量的连续性（Stride）。
**鸭子形状（Duck Shaping）**
如果多个符号在追踪时有相同值，Dynamo 会统一为同一个符号，并添加相等性守卫。例如：

```python
def fn(a, b):
    # 假设 a.shape[0] 和 b.shape[0] 均为 8
    return a + b
```
生成的 Guard：

```python
L['a'].size()[0] == L['b'].size()[0]  # 统一为 s0
```
这减少了符号数量，使得编译器可以生成更高效的通用代码。
**动态条件分支**
当代码中存在依赖 Shape 的条件分支时，Dynamo 会插入复杂的守卫。例如：

```python
@torch.compile(dynamic=True)
def fn(a):
    if a.shape[0] * 2 < 16:  # 符号表达式触发守卫
        return a
    else:
        return a + 1

fn(torch.randn(8))  # 触发守卫：2*s0 >= 16
```
守卫：

```python
2 * L['a'].size()[0] >= 16  # 动态条件的分支守卫
```
允许 Dynamo 在运行时根据输入 Shape 选择正确的代码路径。
### 目标 3:解决无法追踪的 Python 代码问题
**核心思想**
当 Dynamo 遇到无法追踪的操作（如 print()、第三方库调用）时，会暂停当前图的生成，将代码分为多个子图：
    - 前段图：执行到无法追踪的操作之前的代码。
    - 后段图：在无法追踪的操作之后继续生成的新图。
    - 运行时行为：CPython 按顺序执行前段图 → 原生 Python 代码（无法追踪的部分） → 后段图。

#### 实现机制
**字节码重写**
Dynamo 通过 PEP 523 修改 CPython 的帧执行机制，重写原函数的字节码：
- 首次调用：
生成 FX 图 → 编译为高效代码（如通过 Inductor）。
替换原函数的字节码为调用编译后的代码。
- 后续调用：
检查守卫（Guards）确保输入类型一致，复用已编译的代码。
图中断的字节码结构
原函数被拆分为多个「续延函数」（Continuation Function），通过修改后的字节码协调多个子图的执行，并维护局部变量/全局状态。例如：

```python
@torch.compile
def fn(a):
    b = a + 2     # 被追踪为图 1
    print("Hi")   # 触发图中断（无法追踪）
    return b + a  # 被追踪为图 2
```
重写后的字节码逻辑：
    - __compiled_fn_0：执行 a + 2（前段图）。
    - print("Hi")：由 CPython 原生执行。
    - __resume_at_14_1：执行 b + a（后段图）。
    - 变量传递：通过 STORE_FAST/LOAD_FAST 维护变量状态（如 graph_out_0 存储中间结果）。
    
## 最后
PyTorch Dynamo 通过动态追踪和符号化执行，解决了 PyTorch 动态计算图与静态编译之间的矛盾。它通过 Guards 和动态 Shape 支持，实现了对动态特性的无缝支持，同时通过 Graph Breaks 解决了无法追踪的 Python 代码问题。Dynamo 的灵活性和强大功能，使其成为 PyTorch 生态系统中不可或缺的一部分。