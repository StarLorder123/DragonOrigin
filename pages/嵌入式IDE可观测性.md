## 第 1 章：物理与链路层（信号与帧）
- ## 1.1 JTAG 接口引脚定义与电气特性
- JTAG（Joint Test Action Group，IEEE 1149.1）定义的不仅仅是一套调试协议，还规定了**标准接口引脚、电气时序以及状态机**。对于 MCU、SoC、FPGA、CPU 等芯片而言，JTAG 接口通常由 **4 根必需信号 + 1 根可选信号 + 若干辅助信号**组成。
- ### 1.1.1 JTAG 标准引脚定义
- 最常见的是 **20 Pin（ARM）、10 Pin Cortex、14 Pin TI、16 Pin MIPS** 等接口，但真正属于 IEEE1149.1 标准的只有下面几个信号。
  | -    | 引脚             | 全称         | 方向（目标芯片） | 功能 |
  | ---- | ---------------- | ------------ | ---------------- |
  | TCK  | Test Clock       | 输入         | JTAG 时钟        |
  | TMS  | Test Mode Select | 输入         | TAP 状态机控制   |
  | TDI  | Test Data In     | 输入         | 串行输入数据     |
  | TDO  | Test Data Out    | 输出         | 串行输出数据     |
  | TRST | Test Reset       | 输入（可选） | TAP 复位         |
- 除此之外一般还有：
  
  | 引脚   | 作用             |
  | ------ | ---------------- |
  | GND    | 地               |
  | VTREF  | 目标板IO参考电压 |
  | nRESET | 芯片系统复位     |
  | NC     | 保留             |
- ### 1.1.2各引脚详细说明
- **1. TCK（Test Clock）**
- 所有 JTAG 操作均由 TCK 驱动。
- 每个 TCK 上升沿：
	- 状态机变化
	- 数据移位
	- 指令更新
- 通常：1MHz、5MHz、10MHz、20MHz、50MHz
- 高速 FPGA 可以达到：100MHz+
- 电气要求
	- 输入数字时钟：VIL、VIH
	- 通常：CMOS、LVTTL
- **2. TMS（Test Mode Select）**
- 这是最重要的一根线。它控制：TAP State Machine（TAP状态机）
	- 例如：
	- ```
	    Run-Test/Idle
	    ↓
	    Shift-IR
	    ↓
	    Shift-DR
	    ↓
	    Update-DR
		```
- 整个 TAP 状态全部由：TMS+TCK决定。
	- 例如：
	- ```
	    TMS=1
	    Test-Logic-Reset
	    ↓
	    Run-Test
	    ↓
	    Select-DR
	    ↓
	    Select-IR
	    ```
- **3. TDI（Test Data In）**
  - 调试器发送数据：
	- ```
		Debugger
		↓
		TDI
		↓
		Instruction Register
		↓
		Data Register
	  ```
	- 例如发送：
	- ```
		EXTEST
		IDCODE
		BYPASS
		```
	- 或者：
	- ```
		CPU Debug Command
		```
  - 都是经 TDI。
- **4. TDO（Test Data Out）**
- 芯片返回：

- ```
	IDCODE
	CPU寄存器
	Memory
	Trace数据（部分实现）
	```
- 都是：
- ```
	TDO
	↓
	Debugger
	```
- TDO 只在：Shift-IR、Shift-DR状态输出有效。
- 其它时间一般：High-Z方便多个器件级联。
- **5. TRST（Test Reset）**
- 这是可选引脚。
- 作用：立即复位 TAP。否则只能：
- ```
	TMS=1
	连续5个TCK
	进入 Reset
	```
- 所以很多 MCU没有 TRST
- ARM Cortex-M 基本不用 TRST。

### 1.1.3JTAG 电气特性

- IEEE1149.1 并**没有规定具体电压值**，只规定逻辑行为，因此实际电气特性由芯片 I/O 标准决定。
- **输入电压**
	- 例如：
	- 3.3V CMOS
	- ```
		VIH ≥ 2.0V
		VIL ≤ 0.8V
		```
	- 1.8V IO：
	- ```
		VIH≈1.2V
		VIL≈0.4V
		```
- 所以：**JTAG 调试器必须适配目标板 IO 电压**。这也是为什么调试器都有：VTRE
- 这是很多人误解的一根线。注意：不是供电；而是：Reference Voltage
	- 例如：
	- 目标板：1.8V；调试器读取：VTREF=1.8V
	- 于是，输出：
	- ```
	TCK
	TMS
	TDI
	```
	- 全部变成：1.8V；而不是：3.3V；否则会烧芯片。
- **输入阻抗**
	- 通常：几十 kΩ，甚至100kΩ；内部：Pull-up、Pull-down
	- TDO 驱动能力
	- 一般：CMOS Push-Pull；输出：2\~8mA。不同厂家不同。

### 1.1.4 JTAG 时序关系

- 一次 Shift 操作示意：

```
TCK

 __    __    __
|  |__|  |__|  |__

TMS

0---------------

TDI

1---0---1---1---

TDO

x---1---0---1---

```

- 通常：
	- TDI 在 TCK 上升沿前建立（Setup）
	- TDI 在上升沿被采样
	- TDO 在下降沿或随后一段传播延迟后更新（具体实现依芯片而定）
	- 调试器在下一次采样窗口读取 TDO

### 1.1.5 JTAG 接口典型连接

```
          Debugger
        ┌────────────┐
        │            │
TCK ----┤------------├----> MCU
TMS ----┤------------├----> MCU
TDI ----┤------------├----> MCU
TDO <---┤------------├-----
TRST----┤------------├----> MCU
RESET---┤------------├----> nRESET
VTREF<--┤------------├-----
GND -----┤------------├-----
        └────────────┘

```

### 1.1.6 JTAG 引脚级联（Daisy Chain）

JTAG 支持多个器件共享同一组控制信号。

```
           TCK
────────────┬────────────┬──────────

           TMS
────────────┬────────────┬──────────

TDI
↓

+--------+
| Device1|
+--------+
    │
    ▼
+--------+
| Device2|
+--------+
    │
    ▼
+--------+
| Device3|
+--------+
    │
    ▼
TDO

```

其中：

- **TCK**：所有器件共享。
- **TMS**：所有器件共享。
- **TDI**：串联进入第一个器件。
- **TDO**：前一个器件连接到后一个器件的 TDI，最后一个器件的 TDO 返回调试器。

### 1.1.7 JTAG 与 SWD 引脚对比

| 功能   | JTAG        | SWD                   |
| ---- | ----------- | --------------------- |
| 时钟   | TCK         | SWCLK                 |
| 模式控制 | TMS         | 与数据复用为 SWDIO          |
| 数据输入 | TDI         | SWDIO                 |
| 数据输出 | TDO         | SWDIO                 |
| 复位   | TRST（可选）    | 无                     |
| 系统复位 | nRESET      | nRESET                |
| 最少信号 | 4～5 根       | 2 根（SWCLK、SWDIO）      |
| 支持级联 | 是           | 否                     |
| 标准   | IEEE 1149.1 | ARM Serial Wire Debug |