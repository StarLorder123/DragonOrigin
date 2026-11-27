## LLVM 编译流程概述

LLVM 编译器的整体架构采用经典的三阶段设计，将前端、优化器和后端解耦，使得不同的语言前端可以共享同一套优化和代码生成基础设施。

### 整体编译流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   源代码    │────▶│   前端      │────▶│   优化器    │────▶│   后端      │
│  (C/C++等)  │     │  (Clang)    │     │    (LLVM)   │     │  (LLVM)     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                           │                    │                    │
                           ▼                    ▼                    ▼
                     ┌─────────┐          ┌─────────┐          ┌─────────┐
                     │  AST    │          │ LLVM IR │          │机器代码 │
                     └─────────┘          └─────────┘          └─────────┘
```

### 阶段一：前端（Frontend）

**代表**: Clang (C/C++/Objective-C)

**职责**:
1. **词法分析** (Lexical Analysis): 将源代码字符流转换为标记 (Token)
2. **语法分析** (Syntax Analysis): 根据语法规则构建抽象语法树 (AST)
3. **语义分析** (Semantic Analysis): 类型检查、名称解析、常量折叠等
4. **代码生成**: 将 AST 转换为 LLVM IR

**关键文件**:
- `clang/lib/Lex/` - 词法分析器
- `clang/lib/Parse/` - 语法分析器
- `clang/lib/Sema/` - 语义分析器
- `clang/lib/CodeGen/` - LLVM IR 代码生成

### 阶段二：优化器（Middle-end）

**核心**: LLVM Pass 系统

**职责**:
1. **平台无关优化**: 执行各种优化转换
2. **中间表示操作**: 在 LLVM IR 层面进行优化
3. **分析构建**: 构建控制流图、依赖关系等分析结果

**输入**: LLVM IR (文本形式 `.ll` 或二进制形式 `.bc`)
**输出**: 优化后的 LLVM IR

### 阶段三：后端（Backend）

**职责**:
1. **指令选择** (Instruction Selection): 将 LLVM IR 转换为目标机器指令
2. **寄存器分配** (Register Allocation): 将虚拟寄存器映射到物理寄存器
3. **指令调度** (Instruction Scheduling): 重排指令以提高流水线效率
4. **代码发射** (Code Emission): 生成机器码或汇编代码
5. **调试信息生成**: 生成 DWARF 或其他格式的调试信息

**关键组件**:
- `SelectionDAG` - 指令选择有向无环图
- `RegisterAllocators` - 寄存器分配器
- `MC Layer` - 机器码层
- `AsmPrinter` - 汇编打印器，负责生成汇编和调试信息

### LLVM IR 表示

LLVM IR 有三种等价形式：

| 形式 | 扩展名 | 说明 |
|------|--------|------|
| 文本形式 | `.ll` | 可读的文本格式，用于调试 |
| 二进制形式 | `.bc` | Bitcode，紧凑的二进制格式 |
| 内存形式 | - | LLVM 内部数据结构 |

**示例**:
```llvm
; 文本形式的 LLVM IR
define i32 @add(i32 %a, i32 %b) {
entry:
  %result = add i32 %a, %b
  ret i32 %result
}
```

### 编译流程图

```
源代码 (.c/.cpp)
       │
       ▼
┌──────────────────┐
│    Clang 前端    │
│  ┌────────────┐  │
│  │ 词法分析   │  │
│  └─────┬──────┘  │
│        ▼         │
│  ┌────────────┐  │
│  │ 语法分析   │  │
│  └─────┬──────┘  │
│        ▼         │
│  ┌────────────┐  │
│  │ 语义分析   │  │
│  └─────┬──────┘  │
│        ▼         │
│  ┌────────────┐  │
│  │ AST → IR   │  │
│  └────────────┘  │
└────────┬─────────┘
         │ LLVM IR
         ▼
┌──────────────────┐
│   LLVM 优化器    │
│  ┌────────────┐  │
│  │ 分析 Pass  │  │
│  │(Dominators,│  │
│  │ AliasAnal) │  │
│  └─────┬──────┘  │
│        ▼         │
│  ┌────────────┐  │
│  │ 转换 Pass  │  │
│  │(InstCombine,│ │
│  │ GVN, SCCP) │  │
│  └────────────┘  │
└────────┬─────────┘
         │ 优化后 IR
         ▼
┌──────────────────┐
│   LLVM 后端      │
│  ┌────────────┐  │
│  │ 指令选择   │  │
│  │(SelectionDAG)│ │
│  └─────┬──────┘  │
│        ▼         │
│  ┌────────────┐  │
│  │ 寄存器分配 │  │
│  └─────┬──────┘  │
│        ▼         │
│  ┌────────────┐  │
│  │ 指令调度   │  │
│  └─────┬──────┘  │
│        ▼         │
│  ┌────────────┐  │
│  │ 代码发射   │  │
│  │ + 调试信息 │  │
│  └────────────┘  │
└────────┬─────────┘
         │
         ▼
    机器码 (.o/.exe) + 调试信息 (.dwarf/.pdb)
```

### JIT 编译

LLVM 还支持即时编译 (Just-In-Time Compilation)，通过 ORC (On-Request Compilation) 框架实现：

```
┌─────────────────────────────────────┐
│         ORC JIT 框架                │
├─────────────────────────────────────┤
│  ┌──────────┐  ┌──────────────────┐ │
│  │ IRTransforms│  │ObjectTransforms│ │
│  └──────────┘  └──────────────────┘ │
│  ┌──────────┐  ┌──────────────────┐ │
│  │  Compile │  │   Materialize    │ │
│  │  (IR→Obj)│  │   (Obj→Memory)   │ │
│  └──────────┘  └──────────────────┘ │
└─────────────────────────────────────┘
```

## LLVM IR 语言参考

LLVM IR 是 LLVM 编译器的核心表示形式，是一种类型化、静态单赋值（SSA）的中间表示。本节基于 LLVM 语言参考手册，介绍 LLVM IR 的关键概念。

### LLVM IR 的设计目标

LLVM IR 的设计目标包括：

1. **轻量级和低层次**：足够接近硬件，便于代码生成和优化
2. **类型安全**：提供类型信息，支持高级优化
3. **表达能力**：能够干净地表示所有高级语言
4. **可扩展性**：易于添加新的操作和优化

### 标识符

LLVM 标识符分为两种基本类型：

| 类型 | 前缀 | 示例 | 说明 |
|------|------|------|------|
| **全局标识符** | `@` | `@main`, `@global_var` | 函数、全局变量 |
| **局部标识符** | `%` | `%result`, `%x` | 寄存器名称、类型 |

标识符格式：
- **命名标识符**: `%foo`, `@DivisionByZero`
- **未命名标识符**: `%12`, `@2`（自动编号的临时变量）

### 类型系统

LLVM IR 支持丰富的类型系统：

```llvm
; 基本类型
void                           ; 空类型
i1                             ; 1位整数（布尔）
i8, i16, i32, i64, i128        ; 整数类型
float, double, half, bfloat    ; 浮点类型
x86_fp80, fp128, ppc_fp128     ; 特殊浮点类型

; 复合类型
[10 x i32]                     ; 数组：10个i32
{i32, i64, float}              ; 结构体
{i32, [4 x i8]}                ; 嵌套结构体
ptr                            ; 指针类型（opaque pointer）
<4 x i32>                      ; 向量：4个i32

; 函数类型
i32 (i32, i32)                 ; 接受2个i32，返回i32
ptr (ptr)                      ; 接受指针，返回指针
```

### 常量

```llvm
; 整数常量
42                             ; 十进制
0x2A                           ; 十六进制（需要类型上下文）

; 浮点常量
3.14159
1.0E+0                         ; 科学计数法

; 全局字符串
@.str = private constant [12 x i8] c"hello world\00"

; 数组常量
@arr = global [3 x i32] [i32 1, i32 2, i32 3]

; 结构体常量
@point = global {i32, i32} {i32 10, i32 20}

; undef 和 poison
undef                          ; 未定义值
poison                         ; 毒值（使用会导致未定义行为）

; null 指针
ptr null                       ; 空指针
```

### 指令示例

**算术指令**:
```llvm
; 整数算术
%sum = add i32 %a, %b          ; 加法
%diff = sub i32 %a, %b         ; 减法
%prod = mul i32 %a, %b         ; 乘法
%quot = sdiv i32 %a, %b        ; 有符号除法
%rem = srem i32 %a, %b         ; 有符号取余

; 位运算
%and = and i32 %a, %b          ; 按位与
%or = or i32 %a, %b            ; 按位或
%xor = xor i32 %a, %b          ; 按位异或
%shl = shl i32 %a, %b          ; 左移
%shr = ashr i32 %a, %b         ; 算术右移
```

**内存指令**:
```llvm
; 分配
%ptr = alloca i32              ; 在栈上分配 i32
%arr = alloca [10 x i32]       ; 在栈上分配数组

; 加载和存储
%val = load i32, ptr %ptr      ; 从指针加载
store i32 %val, ptr %ptr       ; 存储到指针

; 内存操作
call void @llvm.memcpy.p0.p0.i64(ptr %dst, ptr %src, i64 %n, i1 false)
call void @llvm.memset.p0.i64(ptr %dst, i8 %val, i64 %n, i1 false)
```

**控制流指令**:
```llvm
; 条件分支
br i1 %cond, label %then, label %else

; 无条件分支
br label %next

; 多路分支
switch i32 %val, label %default [
  i32 0, label %case0
  i32 1, label %case1
  i32 2, label %case2
]

; PHI 节点（SSA 汇合点）
%result = phi i32 [%a, %bb1], [%b, %bb2]
```

**函数调用**:
```llvm
; 普通调用
%result = call i32 @foo(i32 %arg1, i32 %arg2)

; 尾调用（优化调用帧）
%result = tail call i32 @foo(i32 %arg)

; 内联调用
%result = musttail call i32 @foo(i32 %arg)

; 通过指针调用
%result = call i32 %funcptr(i32 %arg)
```

### 函数定义

```llvm
; 函数定义
define i32 @add(i32 %a, i32 %b) {
entry:
  %result = add i32 %a, %b
  ret i32 %result
}

; 带属性的函数
define i32 @fast_add(i32 %a, i32 %b) #0 {
  %result = add i32 %a, %b
  ret i32 %result
}

; 声明外部函数
declare i32 @printf(ptr nocapture readonly, ...)

; 属性组
attributes #0 = { nounwind readnone "target-cpu"="haswell" }
```

### 模块结构

```llvm
; 完整的 LLVM 模块示例

; 目标信息
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"

; 全局变量
@global_counter = global i32 0, align 4
@.str = private unnamed_addr constant [13 x i8] c"hello world\0A\00"

; 函数声明
declare i32 @puts(ptr)

; 函数定义
define i32 @main() {
entry:
  %call = call i32 @puts(ptr @.str)
  ret i32 0
}

; 元数据
!llvm.module.flags = !{!0}
!0 = !{i32 1, !"wchar_size", i32 4}
```

### Well-Formedness（良好形成性）

LLVM IR 有一些"良好形成"的要求：

1. **支配性**: 值的定义必须支配所有使用
2. **类型一致性**: 操作数和结果类型必须匹配
3. **SSA 形式**: 每个虚拟寄存器只定义一次
4. **控制流完整性**: 基本块必须有正确的终止符

```llvm
; 错误示例：定义不支配使用
%result = add i32 1, %x        ; 错误：%x 在此处未定义
%x = add i32 %a, %b

; 正确示例
%x = add i32 %a, %b
%result = add i32 1, %x        ; 正确：%x 已定义
```

## LLVM 优化算法概述

LLVM 的优化系统基于 **Pass** 的概念，每个 Pass 执行特定的优化任务。优化 Pass 分布在以下几个主要目录：

| 目录 | 功能 | 典型 Pass |
|------|------|-----------|
| `llvm/lib/Transforms/Scalar` | 标量优化 | GVN, SCCP, DSE, LICM, SROA, EarlyCSE |
| `llvm/lib/Transforms/Vectorize` | 向量化优化 | LoopVectorize, SLPVectorizer, LoadStoreVectorizer |
| `llvm/lib/Transforms/IPO` | 过程间优化 | Inliner, GlobalOpt, GlobalDCE, FunctionAttrs |
| `llvm/lib/Transforms/InstCombine` | 指令合并 | InstCombine |
| `llvm/lib/Transforms/Utils` | 优化工具 | 各种辅助转换 |

### 优化级别

LLVM 支持多个优化级别（-O0 到 -O3, -Os, -Oz），每个级别启用不同的 Pass 组合：

```
-O0: 不优化（调试模式）
-O1: 基础优化，减少编译时间
-O2: 标准优化级别，平衡编译时间和性能
-O3: 激进优化，最大化性能
-Os: 优化代码大小
-Oz: 最小化代码大小
```

## Pass 管理器架构

### 新 Pass 管理器 (New Pass Manager)

LLVM 使用新 Pass 管理器（NPM）来组织和执行优化 Pass。核心类位于：

**关键文件**:
- `llvm/include/llvm/Passes/PassBuilder.h` - PassBuilder 头文件
- `llvm/lib/Passes/PassBuilder.cpp` - PassBuilder 实现
- `llvm/lib/Passes/PassBuilderPipelines.cpp` - 优化流水线定义
- `llvm/include/llvm/Passes/OptimizationLevel.h` - 优化级别定义

**文件**: `llvm/include/llvm/Passes/PassBuilder.h`

```cpp
class PassBuilder {
  LLVMContext &Context;
  const PassesOptions &Opts;
  TargetMachine *TM;
  PipelineTuningOptions PTO;
  std::optional<PGOOptions> PGOOpt;
  PassInstrumentationCallbacks *PIC;
  // ...
};
```

### PipelineTuningOptions 结构

控制各种优化的开关：

```cpp
class PipelineTuningOptions {
public:
  bool LoopInterleaving = true;      // 循环交织
  bool LoopVectorization = true;     // 循环向量化
  bool SLPVectorization = false;     // SLP 向量化
  bool LoopUnrolling = true;         // 循环展开
  bool LoopInterchange = true;       // 循环交换
  bool LoopFusion = false;           // 循环融合
  bool MergeFunctions;               // 函数合并
  int InlinerThreshold = -1;         // 内联阈值
  // ...
};
```

### Pass 执行流程

```
Module
  ├─ ModulePass
  │    └─ 处理整个模块
  ├─ CGSCCPass (Call Graph SCC Pass)
  │    └─ 处理调用图的强连通分量
  │         └─ FunctionPass
  │              └─ 处理单个函数
  │                   ├─ LoopPass
  │                   │    └─ 处理单个循环
  │                   └─ 各种分析 Pass
  └─ 各种模块级 Pass
```

## 核心优化 Pass 详解

### 1. InstCombine（指令合并）

**位置**: `llvm/lib/Transforms/InstCombine/`

**功能**: 将指令合并为更简单或更标准的形式，消除冗余计算。这是 LLVM 中最重要的优化 Pass 之一。

**设计原则**（来自源码注释）:
```cpp
// InstructionCombining.cpp 头部注释
// This pass combines things like:
//    %Y = add i32 %X, 1
//    %Z = add i32 %Y, 1
// into:
//    %Z = add i32 %X, 2
//
// This is a simple worklist driven algorithm.
```

**规范化保证**:
```cpp
// 1. 二元运算符的常量操作数移到 RHS
// 2. 位运算与常量：先移位，再 or，再 and，最后 xor
// 3. 比较指令从 <,>,<=,>= 转换为 ==,!= (如果可能)
// 4. 布尔值上的 cmp 转换为逻辑运算
// 5. add X, X 表示为 (X*2) => (X << 1)
// 6. 乘以 2 的幂转换为移位
```

**核心类**（源码对照）:
```cpp
// llvm/lib/Transforms/InstCombine/InstructionCombining.cpp

class InstCombiner {
  // 工作列表驱动的算法
  InstructionWorklist Worklist;
  
  // 主入口 - 运行指令合并
  bool run();
  
  // 访问者模式 - 各类指令的简化方法
  Instruction *visitAdd(BinaryOperator &I);
  Instruction *visitMul(BinaryOperator &I);
  Instruction *visitCall(CallInst &I);
  Instruction *visitPHI(PHINode &I);
  Instruction *visitSelect(SelectInst &I);
  // ...
  
  // 插入指令时自动添加到工作列表
  void InsertHelper(Instruction *I, ...) {
    Worklist.add(I);
  }
};
```

**统计信息**（源码）:
```cpp
STATISTIC(NumWorklistIterations, "Number of instruction combining iterations");
STATISTIC(NumCombined, "Number of insts combined");
STATISTIC(NumConstProp, "Number of constant folds");
STATISTIC(NumDeadInst, "Number of dead inst eliminated");
STATISTIC(NumSunkInst, "Number of instructions sunk");
STATISTIC(NumExpand, "Number of expansions");
STATISTIC(NumFactor, "Number of factorizations");
STATISTIC(NumReassoc, "Number of reassociations");
```

**实现文件结构**:
| 文件 | 功能 |
|------|------|
| `InstructionCombining.cpp` | 主入口，工作列表算法 |
| `InstCombineAddSub.cpp` | 加减法优化：`(X+C1)+C2 => X+(C1+C2)` |
| `InstCombineAndOrXor.cpp` | 位运算优化：`(X&Y)|(X&Z) => X&(Y|Z)` |
| `InstCombineCalls.cpp` | 调用指令优化 |
| `InstCombineCasts.cpp` | 类型转换优化：`trunc(sext(X)) => X` |
| `InstCombineCompares.cpp` | 比较指令优化 |
| `InstCombineLoadStoreAlloca.cpp` | 内存操作优化 |
| `InstCombineMulDivRem.cpp` | 乘除法优化：`X*8 => X<<3` |
| `InstCombinePHI.cpp` | PHI 节点优化 |
| `InstCombineSelect.cpp` | select 优化：`sel(X>0, X, 0) => max(X, 0)` |
| `InstCombineShifts.cpp` | 移位优化：`(X<<C1)>>C2 => X<<(C1-C2)` |

### 2. GVN（全局值编号）

**位置**: `llvm/lib/Transforms/Scalar/GVN.cpp`

**功能**: 消除完全冗余的指令，执行简单的死代码消除和部分冗余消除（PRE）。

**核心数据结构**（源码对照）:
```cpp
// llvm/lib/Transforms/Scalar/GVN.cpp

// 值编号表 - 为表达式分配唯一编号
class GVNValueTable {
  struct Expression {
    uint32_t Opcode;
    bool Commutative = false;
    Type *Ty = nullptr;
    SmallVector<uint32_t, 4> VarArgs;  // 操作数的值编号
    AttributeList Attrs;
    
    bool operator==(const Expression &Other) const {
      if (Opcode != Other.Opcode) return false;
      if (Ty != Other.Ty) return false;
      if (VarArgs != Other.VarArgs) return false;
      // ...
    }
    
    friend hash_code hash_value(const Expression &E) {
      return hash_combine(E.Opcode, E.Ty, 
                         hash_combine_range(E.VarArgs));
    }
  };
  
  DenseMap<Expression, uint32_t> ExpressionToValueNumber;
};

// 领导者表 - 记录每个值编号对应的值
class GVNLeaderMap {
  struct LeaderTableEntry {
    AssertingVH<Value> Val;
    const BasicBlock *BB;
  };
  
  // 值编号 -> 领导者列表
  DenseMap<uint32_t, LeaderListNode> NumToLeaders;
};
```

**统计信息**（源码）:
```cpp
STATISTIC(NumGVNInstr, "Number of instructions deleted");
STATISTIC(NumGVNLoad, "Number of loads deleted");
STATISTIC(NumGVNPRE, "Number of instructions PRE'd");
STATISTIC(NumGVNBlocks, "Number of blocks merged");
STATISTIC(NumGVNSimpl, "Number of instructions simplified");
STATISTIC(NumGVNEqProp, "Number of equalities propagated");
STATISTIC(NumPRELoad, "Number of loads PRE'd");
```

**GVNPass 核心实现**（源码）:
```cpp
class GVNPassImpl {
  // 值编号表
  GVNValueTable VN;
  
  // 领导者映射
  GVNLeaderMap LeaderMap;
  
  // 执行 GVN
  bool runGVN(Function &F) {
    // 1. 为每条指令计算值编号
    // 2. 查找已有的领导者
    // 3. 如果找到相同的值编号，用领导者替换
    // 4. 否则，添加新的领导者
  }
  
  // 检查值是否在块中完全可用
  bool isValueFullyAvailableInBlock(Value *V, BasicBlock *BB);
  
  // PRE: 部分冗余消除
  bool performPRE(Function &F);
};
```

**关键依赖**:
- `MemoryDependenceAnalysis`: 内存依赖分析
- `AliasAnalysis`: 别名分析
- `DominatorTree`: 支配树
- `TargetLibraryInfo`: 目标库信息

### 3. SCCP（稀疏条件常量传播）

**位置**: `llvm/lib/Transforms/Scalar/SCCP.cpp`

**功能**: 假设值为常量，除非证明 otherwise；假设基本块不可达，除非证明 otherwise。

**核心算法**（源码对照）:
```cpp
// llvm/lib/Transforms/Scalar/SCCP.cpp

static bool runSCCP(Function &F, const DataLayout &DL,
                    const TargetLibraryInfo *TLI, 
                    DominatorTree &DT, AssumptionCache &AC) {
  SCCPSolver Solver(DL, ...);
  
  // 添加谓词信息（用于更精确的分析）
  Solver.addPredicateInfo(F, DT, AC);
  
  // 标记函数入口块可达
  Solver.markBlockExecutable(&F.front());
  
  // 跟踪参数值（基于属性）
  for (Argument &AI : F.args())
    Solver.trackValueOfArgument(&AI);
  
  // 求解直到不动点
  bool ResolvedUndefs = true;
  while (ResolvedUndefs) {
    Solver.solve();
    ResolvedUndefs = Solver.resolvedUndefsIn(F);
  }
  
  // 应用结果
  for (BasicBlock &BB : F) {
    if (!Solver.isBlockExecutable(&BB)) {
      // 将不可达块转为 unreachable
      changeToUnreachable(&*BB.getFirstNonPHIIt(), ...);
    }
  }
  
  // 移除不可达块
  for (BasicBlock *DeadBB : BlocksToErase)
    DTU.deleteBB(DeadBB);
}
```

**SCCPSolver 核心逻辑**:
```cpp
class SCCPSolver {
  // 值的状态
  enum LatticeState {
    Undefined,    // 未定义（初始状态）
    Constant,     // 已知常量
    Overdefined   // 非常量（多个可能值）
  };
  
  // 值格
  DenseMap<Value*, LatticeVal> StateMap;
  
  // 可执行块集合
  SmallPtrSet<BasicBlock*, 16> ExecutableBlocks;
  
  // 核心操作
  void markBlockExecutable(BasicBlock *BB);
  void markConstant(Value *V, Constant *C);
  void markOverdefined(Value *V);
  
  // 获取格的值
  LatticeVal getLatticeValue(Value *V);
};
```

**统计**:
```cpp
STATISTIC(NumInstRemoved, "Number of instructions removed");
STATISTIC(NumDeadBlocks, "Number of basic blocks unreachable");
STATISTIC(NumInstReplaced, "Number of instructions replaced");
```

### 4. DSE（死存储消除）

**位置**: `llvm/lib/Transforms/Scalar/DeadStoreElimination.cpp`

**功能**: 基于 MemorySSA 消除不会被读取的存储。

**核心算法**（源码注释）:
```cpp
// DeadStoreElimination.cpp 头部注释
// The code below implements dead store elimination using MemorySSA.
//
// For all MemoryDefs StartDef:
// 1. Get the next dominating clobbering MemoryDef (MaybeDeadAccess)
// 2. Check that there are no reads between MaybeDeadAccess and StartDef
// 3. For each found CurrentDef, check:
//    a. No barrier instructions between CurrentDef and StartDef
//    b. StartDef is executed whenever CurrentDef is executed
//    c. StartDef completely overwrites CurrentDef
// 4. Erase CurrentDef from the function and MemorySSA
```

**覆盖结果枚举**（源码）:
```cpp
enum OverwriteResult {
  OW_Begin,                        // 部分覆盖（开头）
  OW_Complete,                     // 完全覆盖
  OW_End,                          // 部分覆盖（结尾）
  OW_PartialEarlierWithFullLater,  // 部分覆盖
  OW_MaybePartial,                 // 可能部分覆盖
  OW_None,                         // 无覆盖
  OW_Unknown                       // 未知
};
```

**统计**（源码）:
```cpp
STATISTIC(NumRemainingStores, "Number of stores remaining after DSE");
STATISTIC(NumRedundantStores, "Number of redundant stores deleted");
STATISTIC(NumFastStores, "Number of stores deleted");
STATISTIC(NumCompletePartials, "Number of stores dead by later partials");
STATISTIC(NumModifiedStores, "Number of stores modified");
STATISTIC(NumGetDomMemoryDefPassed, "Number of valid candidates");
STATISTIC(NumDomMemDefChecks, "Number of iterations in getDomMemoryDef");
```

**关键依赖**:
- `MemorySSA`: 内存静态单赋值形式
- `AliasAnalysis`: 别名分析
- `PostDominators`: 后支配树
- `MustExecute`: 必须执行分析

### 5. LoopVectorize（循环向量化）

**位置**: `llvm/lib/Transforms/Vectorize/LoopVectorize.cpp`

**功能**: 将循环的连续迭代合并为单次"宽"迭代，使用 SIMD 指令。

**源码注释**（设计说明）:
```cpp
// LoopVectorize.cpp 头部注释
// This is the LLVM loop vectorizer. This pass modifies 'vectorizable' loops
// and generates target-independent LLVM-IR.
// The vectorizer uses the TargetTransformInfo analysis to estimate the costs
// of instructions in order to estimate the profitability of vectorization.
//
// The loop vectorizer combines consecutive loop iterations into a single
// 'wide' iteration. After this transformation the index is incremented
// by the SIMD vector width, and not by one.
//
// This pass has four parts:
// 1. The main loop pass that drives the different parts.
// 2. LoopVectorizationLegality - A unit that checks for the legality
//    of the vectorization.
// 3. LoopVectorizationPlanner - A unit that builds VPlans for the candidate
//    vectorization factors, selects the best one and executes it.
// 4. LoopVectorizationCostModel - A unit that checks for the profitability
//    of vectorization.
```

**统计信息**（源码）:
```cpp
STATISTIC(LoopsVectorized, "Number of loops vectorized");
STATISTIC(LoopsAnalyzed, "Number of loops analyzed for vectorization");
STATISTIC(LoopsEpilogueVectorized, "Number of epilogues vectorized");
STATISTIC(LoopsEarlyExitVectorized, "Number of early exit loops vectorized");
STATISTIC(LoopsPartialAliasVectorized, 
          "Number of partial aliasing loops vectorized");
```

**架构**（四个主要部分）:

```
LoopVectorize Pass
├─ LoopVectorizationLegality    // 检查向量化合法性
├─ LoopVectorizationPlanner     // 构建 VPlan，选择最优方案
├─ LoopVectorizationCostModel   // 评估向量化收益
└─ VPlan Infrastructure         // 向量化计划基础设施
```

**关键类**（源码对照）:
```cpp
// 主 Pass
class LoopVectorizePass : public PassInfoMixin<LoopVectorizePass> {
  // 运行向量化
  PreservedAnalyses run(Loop *L, LoopAnalysisManager &AM, ...) {
    // 1. 检查合法性
    LoopVectorizationLegality LVL(L, ...);
    if (!LVL.canVectorize()) return PreservedAnalyses::all();
    
    // 2. 评估成本
    LoopVectorizationCostModel CM(L, ...);
    unsigned VF = CM.selectVectorizationFactor();
    if (VF == 1) return PreservedAnalyses::all();  // 不向量化
    
    // 3. 构建并执行计划
    LoopVectorizationPlanner Planner(L, ...);
    Planner.plan(VF);
    Planner.execute();
    
    return PreservedAnalyses::none();
  }
};

// 合法性检查
class LoopVectorizationLegality {
  // 检查循环是否可以向量化
  bool canVectorize() {
    // 检查条件：
    // - 循环有已知的.trip count
    // - 没有不可向量化的指令
    // - 内存访问可以分析
    // - 没有复杂的控制流
    return hasKnownTripCount() && 
           allInstructionsVectorizable() &&
           canAnalyzeMemoryAccesses();
  }
};

// 成本模型
class LoopVectorizationCostModel {
  // 计算向量化成本
  InstructionCost getInstructionCost(Instruction *I) {
    // 使用 TTI 获取目标特定的成本
    return TTI.getInstructionCost(I, TTI.TCK_RecipThroughput);
  }
  
  // 选择最优向量宽度
  unsigned selectVectorizationFactor() {
    // 尝试不同的 VF（2, 4, 8, ...）
    // 选择成本最低的
    for (unsigned VF = MaxVF; VF >= 2; VF /= 2) {
      Cost = calculateCost(VF);
      if (Cost < BestCost) BestVF = VF;
    }
    return BestVF;
  }
};
```

**参考论文**:
- D. Nuzman and R. Henderson. *Multi-platform Auto-vectorization*
- Karrenberg, R. and Hack, S. *Whole Function Vectorization*
- Dorit Nuzman, Ira Rosen and Ayal Zaks. *Auto-Vectorization of Interleaved Data for SIMD*

