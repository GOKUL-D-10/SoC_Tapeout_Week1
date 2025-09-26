## 🧩 Introduction to Optimization

Optimization is the process of improving a digital design to achieve **efficient performance**, **reduced area**, and **lower power consumption**.

It can be broadly divided into **Combinational Logic Optimization** and **Sequential Logic Optimization**.

---

## 1️⃣ Combinational Logic Optimization

**Goal:** Squeeze the logic to get the most optimized design which is efficient in terms of **power** and **area**.

**Methods of Optimization:**

- **Constant Propagation**: Direct optimization using fixed input values.
- **Boolean Logic Optimization**: Techniques like **K-Map** and **Quine-McCluskey**.

### 🔹 Constant Propagation

Example:

If `Y = ((A * B) + C)'` and `A = 0`, then `Y = C'`.

By keeping constant values for inputs, the circuit can be simplified.

### 🔹 Boolean Logic Optimization

Example:

`Y = a ? b ? c : (c ? a : 0) : (!c)`

Simplified Boolean logic:

`Y = A'C' + A(BC + AB'C)`

Optimized result:

`Y = A'C' + AC` ✅ (Equivalent to **A XNOR C**)

---

## 2️⃣ Sequential Logic Optimization

### 🔹 Basic Sequential Optimization

**Sequential Constant Propagation:**

- Consider output `q` of a D flip-flop connected to a NAND gate, with another input `A`. Let the output of NAND be `Y`, and D input is grounded.
- Whether reset signal is applied or not, `q = 0`, so `Y = 1`.

⚠ Note: For a **SET flip-flop**, the output `q` depends on the clock edge, so `q` may not immediately follow the SET input.

### 🔹 Advanced Sequential Optimization

1. **State Optimization**: Optimize **unused states** in the state machine.
2. **Cloning**:
    - If output of flip-flop `A` feeds combinational logic that drives `B` and `C`, and they are far apart:
    - Clone `A` into two flip-flops, feeding each combinational path separately.
3. **Retiming**:
    - Consider 3 flip-flops with combinational circuits `A` and `B` between them.
    - Delay: `A = 5ns`, `B = 3ns` → Clock frequency differs (`A = 200MHz`, `B = 333MHz`)
    - Shift logic from `A` to `B` to equalize delay (`4ns` each) → unified clock frequency (`250MHz`).

---
## ⚡ Combinational Logic Optimization Lab

we synthesize **combinational design files** with optimization techniques using **Yosys**.

We’ll explore how **constant propagation** and **Boolean optimization** simplify logic into smaller, faster, and more efficient netlists 🔧✨.

---

### 🔹 1. Synthesis Command

We use **Yosys** for synthesis. Below are the command flows for **single module** and **multiple module** synthesis.

🛠️ **For Single Module**

```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog design_name.v
synth -top module_name
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

**Explanation of Commands 📝**

- `yosys` → Start the synthesis tool ⚙️
- `read_liberty` → Load technology library 📚
- `read_verilog` → Load design file 📄
- `synth -top` → Run synthesis on top module 🎯
- `opt_clean -purge` → Remove unused wires, nets, or cells 🧹
- `abc -liberty` → Technology mapping using target library 🔗
- `show` → Display optimized netlist diagram 🎨

---

🛠️ **For Multiple Modules**

```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog design_name.v
synth -top main_module_name
flatten
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

**Explanation of Commands 📝**

- `flatten` → Merge hierarchical modules into a single module 🏗️
- Rest of commands same as **single module synthesis**.

---

### 🔹 2. opt_check.v 🧮

📌 A simple design showing **direct constant optimization**.

Equation simplifies from `y = a?b:0` → `y = ab`.

```verilog
module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
```

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_opt_check.png" width="80%"/>
</p>

---

### 🔹 3. opt_check2 🔗

📌 Shows how **distributive law** helps reduce circuit size.

Equation reduces to `y = a + b`.

```verilog
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_opt_check2.png" width="80%"/>
</p>

---

### 🔹 4. opt_check3 🔄

📌 Nested ternary operations simplified by constant propagation.

The logic shrinks down to a minimal netlist.

```verilog
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_opt_check3.png" width="80%"/>
</p>

---

### 🔹 5. opt_check4 🧩

📌 A more **complex nested logic** example using multiple ternary operators.

Synthesis reveals huge simplifications in gate-level form.

```verilog
module opt_check4 (input a , input b , input c , output y);
 assign y = a?(b?(a & c ):c):(!c);
endmodule
```

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_opt_check4.png" width="80%"/>
</p>

---

### 🔹 6. multiple_module_opt 🏗️

📌 Demonstrates synthesis on **multiple submodules**.

Hierarchical design is **flattened and optimized** automatically.

```verilog
module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule

module sub_module2(input a , input b , output y);
 assign y = a^b;
endmodule

module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;
sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));
assign y = c | (b & n1);
endmodule
```

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_multiple_modules_opt.png" width="80%"/>
</p>

---

### 🔹 7. multiple_module_opt2 ⚙️

📌 Another multi-module design showing **hierarchy simplification**.

Unused logic is cleaned up by **opt_clean -purge**.

```verilog
module sub_module(input a , input b , output y);
 assign y = a & b;
endmodule

module multiple_module_opt2(input a , input b , input c , input d , output y);
wire n1,n2,n3;
sub_module U1 (.a(a) , .b(1'b0) , .y(n1));
sub_module U2 (.a(b), .b(c) , .y(n2));
sub_module U3 (.a(n2), .b(d) , .y(n3));
sub_module U4 (.a(n3), .b(n1) , .y(y));
endmodule
```

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_multiple_module_opt2.png" width="80%"/>
</p>


---
## 🔄 Sequential Logic Optimization

we explore **sequential optimization** 🕒⚡ using Yosys. By applying constant propagation and simplification on **flip-flops**, we achieve **optimized designs** that are smaller, faster, and more efficient.

---

### 🔹 1. Synthesis Command

Commands used for **sequential design synthesis** in Yosys.

This flow helps map flip-flops into the technology library 🔧📦.

```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog design_name.v
synth -top module_name
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show

```

**Command Explanation 📝**

- `yosys` → Launch synthesis tool ⚙️
- `read_liberty` → Import cell library 📚
- `read_verilog` → Load design file 📄
- `synth -top` → Synthesize top-level module 🎯
- `dfflibmap` → Map DFFs to library flip-flops ⏱️
- `abc -liberty` → Optimize & tech map using ABC 🔗
- `show` → Display optimized netlist 🖼️

---

### 🔹 2. dff_const1.v 🕹️

📌 Flip-flop always outputs **constant ‘1’** after reset.

GTKWave shows reset clears to `0`, then holds `1`.

```verilog
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end
endmodule
```

📊 **GTKWave Analysis:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/GTKWave_dff_const1.png" width="80%"/>
</p>

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_dff_const1.png" width="80%"/>
</p>

---

### 🔹 3. dff_const2.v 🎛️

📌 Output is **always 1**, regardless of reset or clock.

Yosys optimizes this to a constant wire 🟢.

```verilog
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end
endmodule
```

📊 **GTKWave Analysis:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/GTKWave_dff_const2.png" width="80%"/>
</p>

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_dff_const2.png" width="80%"/>
</p>

---

### 🔹 4. dff_const3.v ⏳

📌 Two registers interact: one keeps constant `1`, other follows delayed update.

GTKWave shows sequential dependency in action 🔄.

```verilog
module dff_const3(input clk, input reset, output reg q);
reg q1;
always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end
endmodule
```

📊 **GTKWave Analysis:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/GTKWave_dff_const3.png" width="80%"/>
</p>

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_dff_const3.png" width="80%"/>
</p>

---

### 🔹 5. dff_const4.v ⚡

📌 Both registers reset to `1` and always stay at `1`.

This collapses into constant logic output 🚀.

```verilog
module dff_const4(input clk, input reset, output reg q);
reg q1;
always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b1;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end
endmodule
```

📊 **GTKWave Analysis:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/GTKWave_dff_const4.png" width="80%"/>
</p>

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_dff_const4.png" width="80%"/>
</p>

---

### 🔹 6. dff_const5.v 🔐

📌 Reset clears both registers to `0`, but sequentially they stabilize to `1`.

GTKWave confirms one-cycle delay before output updates ⏱️.

```verilog
module dff_const5(input clk, input reset, output reg q);
reg q1;
always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b0;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end
endmodule
```

📊 **GTKWave Analysis:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/GTKWave_dff_const5.png" width="80%"/>
</p>

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_dff_const5.png" width="80%"/>
</p>

---
## 🌀 Sequential Optimization for Unused Outputs

we explore how **unused outputs in sequential circuits** can be optimized 🛠️.

Yosys cleverly removes unnecessary flip-flops, keeping only what’s required ⚡.

---

### 🔹 1. counter_opt.v ⏱️

📌 Only the **least significant bit** (`c[0]`) is used.

Thus, the design reduces to a **toggle flip-flop** 🔄.

```verilog
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count[0];

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end
endmodule
```

📊 **GTKWave Analysis:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/GTKWave_Counter_opt.png" width="80%"/>
</p>

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_counter_opt.png" width="80%"/>
</p>

---

### 🔹 2. counter_opt2.v 🧮

📌 Here, **all 3 counter bits** are needed to detect `count = 100`.

So optimization **retains the full counter logic** 🏗️.

```verilog
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = (count[2:0] == 3'b100);

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end
endmodule
```

📷 **Synthesized Netlist:**

<p align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%203/images/Netlist_counter_opt2.png" width="80%"/>
</p>

---

## Summary
Day 3 focused on **optimizing digital designs** to reduce area, power, and improve performance.

- **Combinational Optimization:** Simplified logic using **constant propagation** and **Boolean optimization**, observed netlist improvements in `opt_check` designs 🧩.
- **Sequential Optimization:** Optimized D flip-flops with constants (`dff_const` series) and reduced unnecessary logic 🔄.
- **Unused Outputs:** Removed unneeded flip-flops in counters (`counter_opt`) to save area and power ⚡.

✅ **Takeaway:** Optimization makes designs **smaller, faster, and more efficient**, preparing SoC circuits for real-world applications.
