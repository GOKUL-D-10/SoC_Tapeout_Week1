# Day 2 of the SoC Workshop!

## 🕒 Topic: Timing Libraries, Synthesis Approaches, Flip-Flop Coding & Optimization
Welcome back to **Day 2** of our RTL design journey 🚀.<br>
Today, we’ll explore **timing libraries**, **synthesis styles**, and **flip-flop coding techniques**, along with some neat **optimization tricks**.

By the end of this session, you will:

✅ Understand `.lib` files and their importance in digital design<br>
✅ Compare **Hierarchical vs Flat Synthesis** with Yosys<br>
✅ Implement different **Flip-Flop styles** in Verilog and analyze with GTKWave<br>
✅ Learn smart **RTL optimizations** for multipliers

---
## 📂 Contents

- ⚡ [**Introduction to Timing .lib**](Introduction-to-Timing-.lib)
    - 📘 [SKY130 PDK](SKY130-PDK)
    - 🔍 [Decoding sky130_fd_sc_hd__tt_025C_1v80.lib](Decoding-sky130-fd-sc-hd-tt-025C-1v80.lib)
    - 📝 [Units and Exploration](Units-and-Exploration)
- 🛠 [**Hierarchical vs Flat Synthesis**](Hierarchical-vs-Flat-Synthesis)
    - 🏗 [Hierarchical Synthesis](Hierarchical-Synthesis)
    - 🧩 [Flattened Synthesis]([Flattened-Synthesis)
    - ⚖️ [Key Differences & CMOS Tip](Key-Differences-&-CMOS-Tip)
- 🔹 [**Flip-Flop Coding Styles**](Flip-Flop-Coding-Styles)
    - ⏱ [Async Reset DFF](Async-Reset-DFF)
    - 🔴 [Async Set DFF](Async-Set-DFF)
    - ⏹ [Sync Reset DFF](Sync-Reset-DFF)
    - 🔄 [Combined Async + Sync Reset DFF](Combined-Async-+-Sync-Reset-DFF)
- 🖥 [**Optimizations in RTL**](Optimizations-in-RTL)
    - ✖ [Multiplier by 2]([Multiplier-by-2)
    - ✖ [Multiplier by 9]([Multiplier-by-9)
- 📝 [**Summary**](Summary)

---
## ⚡ Introduction to Timing .lib

### 📘 SKY130 PDK

The **SKY130 PDK** 🏭 is an open-source kit based on **SkyWater 130nm CMOS technology** 💻⚡, containing libraries for synthesis, timing, power, and simulation 📊.


### 🔍 Decoding `sky130_fd_sc_hd__tt_025C_1v80.lib`

| 🏷 Field | 📖 Meaning |
| --- | --- |
| **tt** | Typical process corner ⚖️ |
| **025C** | Temperature = 25°C 🌡️ |
| **1v80** | Supply Voltage = 1.8V 🔋 |

👉 This defines the **PVT (Process, Voltage, Temperature)** conditions under which the library is valid.


### 📏 Units and Exploration

- ⏱ **Time** → ns
- 🔋 **Voltage** → V
- 💡 **Leakage Power** → nW
- ⚡ **Current** → mA
- 🪝 **Pull Resistance** → kΩ
- 🧪 **Load Capacitance** → pF

### 📖 Exploring `.lib`:
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/sky130lib.png?raw=true">
</div>

---
## Hierarchical vs Flat Synthesis

### Example: Multiple Modules Design
```
module sub_module2 (input a, input b, output y);
  assign y = a | b;
endmodule

module sub_module1 (input a, input b, output y);
  assign y = a & b;
endmodule

module multiple_modules (input a, input b, input c, output y);
  wire net1;
  sub_module1 u1(.a(a), .b(b), .y(net1));  // net1 = a & b
  sub_module2 u2(.a(net1), .b(c), .y(y));  // y = (a & b) | c
endmodule
```

## 🛠 Hierarchical vs Flat Synthesis

### 🏗 Hierarchical Synthesis

💡 Retains module boundaries → easier debugging 🐞 and modular design 🧱

**Yosys Flow:**

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr multiple_modules_hier.v

```

📌 **Command Purposes:**

- 📚 `read_liberty` → load timing library
- 📄 `read_verilog` → read RTL code
- 🏷 `synth -top` → mark top module
- ⚡ `abc` → technology mapping
- 👀 `show` → visualize netlist
- ✍ `write_verilog` → dump netlist

📷 **Hierarchical Netlist:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/multiple_module_netlist.png?raw=true" width="70%"/>
</div>

✅ **Advantages:** Easy debug 🐞, modular 🔗<br>
❌ **Disadvantages:** Limited optimization ⚠️


### 🧩 Flattened Synthesis

💡 Merges all modules into one → enables cross-module optimizations 🌐

**Yosys Flow:**

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
show
write_verilog -noattr multiple_modules_flat.v

```

📷 **Flattened Netlist:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/flat_netlist.png?raw=true" width="70%">
</div>

✅ **Advantages:** Better optimization ⚡<br>
❌ **Disadvantages:** Harder debugging 🐛, runtime ⏳

---

### ⚖️ Key Differences & CMOS Tip

- 🏗 **Hierarchical** → Preserves structure 🧱
- 🌐 **Flattened** → Maximum optimization 🚀

📌 **CMOS Fact:**

- ❌ OR gates = stacked PMOS → high resistance
- ✅ OR = implemented using NAND + inverters 🌀

**OR gate (CMOS Logic):**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/OR_cmos.png?raw=true" width="30%">
</div>

**NAND gate (CMOS Logic):**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/NAND_cmos.png?raw=true" width="30%">
</div>

**OR gate using NAND:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/OR_using_NAND.png?raw=true" width="30%">
</div>

---
## 🔹 Flip-Flop Coding Styles

### ⏱ D Flip-Flop with Asynchronous Reset
- Reset works **immediately**, ignoring the clock ⏰❌.
- Sets output `q` to `0` instantly when reset is high 🟥⚡.
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/DFF_Async_Reset.png" width="70%">
</div>
<br>
```verilog
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @ (posedge clk, posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```

📊 **GTKWave:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/GTK_DFF_Async_Reset.png?raw=true">
</div>

🪛 **Synthesized Netlist:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/Netlist_DFF_Async_Reset.png?raw=true">
</div>
<br>

---
### 🔴 D Flip-Flop with Asynchronous Set
- Set works **immediately**, ignoring the clock ⏰✅.
- Forces output `q` to `1` instantly when set is high 🟩⚡.
- 
```verilog
module dff_async_set (input clk, input async_set, input d, output reg q);
  always @ (posedge clk, posedge async_set)
    if (async_set)
      q <= 1'b1;
    else
      q <= d;
endmodule
```

📊 **GTKWave:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/GTK_DFF_Async_Set.png?raw=true">
</div>

🪛 **Synthesized Netlist:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/Netlist_DFF_Async_Set.png?raw=true">
</div>
<br>

---
### ⏹ D Flip-Flop with Synchronous Reset
- Reset waits for the **clock edge** ⏰🕹️.
- Output `q` becomes `0` only at the rising clock edge 🟥⏳.
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/DFF_Sync_Reset.png" width="70%">
</div>
<br>

```verilog
module dff_syncres (input clk, input sync_reset, input d, output reg q);
  always @ (posedge clk)
    if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```

📊 **GTKWave:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/GTK_DFF_Sync_Reset.png?raw=true">
</div>

🪛 **Synthesized Netlist:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/Netlist_DFF_Sync_Reset.png?raw=true">
</div>
<br>

---

### 🔄 D Flip-Flop with Both Sync & Async Reset
- Combines **asynchronous reset** and **synchronous reset** ⚡⏹.
- Output `q` can be reset immediately or at the clock edge depending on condition 🟥🕹️.
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/DFF_Async_Sync_Reset.png" width="70%">
</div>
<br>

```verilog
module dff_asyncres_syncres (input clk, input async_reset, input sync_reset, input d, output reg q);
  always @ (posedge clk, posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```

📊 **GTKWave:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/GTK_DFF_Async_Sync_Reset.png?raw=true">
</div>
<br>

---

## 🖥 Optimizations in RTL

### ✖2️⃣ Multiplier by 2

```verilog
module mul2 (input [2:0] a, output [3:0] y);
  assign y = a * 2;   // Optimized as {a, 0}
endmodule
```

📜**Netlist Code:**
```verilog
/* Generated by Yosys 0.57+153 (git sha1 6b3a7e244, g++ 14.2.0-19ubuntu2 -fPIC -O3) */

module mul2(a, y);
  input [2:0] a;
  wire [2:0] a;
  output [3:0] y;
  wire [3:0] y;
  assign y = { a, 1'h0 };
endmodule
```

🪛 **Synthesized Netlist:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/Netlist_mult2.png?raw=true" width="70%"/>
</div>
<br>

---

### ✖9️⃣ Multiplier by 9

```verilog
module mult8 (input [2:0] a, output [5:0] y);
  assign y = a * 9;   // Optimized as {a, a}
endmodule
```

📜**Netlist Code:**
```verilog
/* Generated by Yosys 0.57+153 (git sha1 6b3a7e244, g++ 14.2.0-19ubuntu2 -fPIC -O3) */

module mult8(a, y);
  input [2:0] a;
  wire [2:0] a;
  output [5:0] y;
  wire [5:0] y;
  assign y = { a, a };
endmodule
```

🪛 **Synthesized Netlist:**
<div align="center">
<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%202/images/Netlist_mult8.png?raw=true" width="70%"/>
</div>
<br>

---
## 🗂️ NOTES & TIPs
- ⚡ **Use `dfflibmap`** after synthesis to map your RTL flip-flops to technology-specific cells.
- 📚 Always **read the Liberty library** first (`read_liberty -lib …`) to guide the synthesizer.
- 🖥 For simulation, compile both **RTL and testbench** in Icarus Verilog before viewing in GTKWave.
- 🧩 **Async reset/set DFFs** will override the clock, while **sync reset DFFs** trigger only on the clock edge.
- 🪛 **Netlist check:** Use `show` in Yosys to visualize flip-flops and connections.
- ⚡ **Optimization tip:** Avoid unnecessary complex logic; use direct assignments for simple multipliers `{a,0}` instead of full multiplier logic.

---

## 📝 Summary

✨ Today, you explored:

- ⚡ `.lib` files and **PVT**
- 🏗 Hierarchical vs 🧩 Flattened synthesis
- ⏱ 🔴 ⏹ 🔄 Different **DFF styles**
- ✖ Optimized multipliers (×2, ×9)

🚀 Keep experimenting with **Yosys, GTKWave & SKY130 PDK** to master RTL design 💡
