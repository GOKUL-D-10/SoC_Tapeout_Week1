# Day 1: RTL Design, Simulation, and Synthesis 🚀

Welcome to Day 1! we'll dive into digital design using Verilog, simulate our work with the open-source tool **Icarus Verilog**, and learn the basics of logic synthesis with **Yosys**. This guide covers the essential labs and concepts to build a solid foundation.

---
##  Table of Contents

| Section | Topic |
|---------|-------|
| 1️⃣ | [The Core Components: Design, Testbench, & Simulator](#1-the-core-components-design-testbench--simulator) |
| 2️⃣ | [Setting Up Workshop files](#2-Setting-Up-Workshop-files) |
| 3️⃣ | [IVerilog](#3-IVerilog) |
| 4️⃣ | [GTKWave Analysis](#4-GTKWave-Analysis) |
| 5️⃣ | [YOSYS](#5-YOSYS) |
| 6️⃣ | [Summary](#7-Summary) |

---
## The Core Components: Design, Testbench, & Simulator
Before a chip is manufactured, its design must be created and verified using specific code and software tools.
* **Design:** The Verilog code that describes the intended logic of your circuit. It's the blueprint.
* **Testbench:** A separate Verilog file that acts as an automated tester, sending a sequence of inputs to your design to confirm it behaves correctly.
* **Simulator:** A software tool that runs the testbench and simulates the design's response, predicting how the physical circuit will behave.

<div align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/Simulator_Testbench.png" width="70%">
</div>

---
## Setting Up Workshop files
First, clone the Git repository, which contains all the necessary design, testbench, and library files for the workshop.
	
	git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git

---
## IVerilog
Icarus Verilog (iverilog) is an open-source tool that compiles and simulates Verilog HDL designs.
The design along with testbench compiled using iverilog to generate a `.vcd` file which further can be analysed by GTKWave.

<div align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/rtl%20flow.png" width="70%">
</div>
<br>

### Design file (`good_mux.v`):
```
module good_mux (input i0 , input i1 , input sel , output reg y);
always @ (*)
begin
	if(sel)
		y <= i1;
	else
		y <= i0;
end
endmodule
```
<br>

### Testbench file (`tb_good_mux.v`):
```
`timescale 1ns / 1ps
module tb_good_mux;
	// Inputs
	reg i0,i1,sel;
	// Outputs
	wire y;

        // Instantiate the Unit Under Test (UUT)
	good_mux uut (
		.sel(sel),
		.i0(i0),
		.i1(i1),
		.y(y)
	);

	initial begin
	$dumpfile("tb_good_mux.vcd");
	$dumpvars(0,tb_good_mux);
	// Initialize Inputs
	sel = 0;
	i0 = 0;
	i1 = 0;
	#300 $finish;
	end

always #75 sel = ~sel;
always #10 i0 = ~i0;
always #55 i1 = ~i1;
endmodule
```
<br>

### Run Simulation:
<div align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/iverilog%20command.jpeg" width="70%">
</div>

---
## GTKWave Analysis
GTKWave is an open-source waveform viewer used to visualize simulation results from digital designs.
Allows users to inspect signals over time, debug designs, and verify circuit behavior.

### Visualization Command
<div align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/gtkwave%20command.jpeg" width="70%">
</div>
<br>

### GTKWAVE Output (`good_mux.v`):
<div align="center">
	<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/gtkwave%20output.jpeg" width="70%">
</div>

---
## YOSYS
Yosys is an open-source Synthesizer for Verilog synthesis and RTL design.
It converts Verilog designs into gate-level netlists.

<div align="center">
	<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/yosys_flow.png" width="70%">
	<img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/netlist%20flow.png" width="70%">
</div>
<br>

### ⚡Flavours of Gates:
Standard cell libraries provide gates in multiple flavors:
* 2-input, 3-input, 4-input versions
* Fast, Medium, Slow drive strengths

Why do we need different flavors?<br>
**Timing Constraints**
1. *For setup time paths* (`tCLK > Tcq_A + Tcombi + Tsetup_B`):
   <br>	Tcombi must be minimized → use fast gates.
3. *For hold time paths* (`tHOLD_B < Tcq_A + Tcombi`):
   <br>	Need extra delay → use slow gates.

**Load Handling**
Load = capacitance. Faster charging/discharging requires wider transistors.
Wider cells → less delay, but more area & power.
Narrower cells → more delay, but less area & power.

**⚖️ Trade-off**
Using too many fast cells → circuit consumes more area & power.
Using too many slow cells → circuit becomes sluggish.
Optimal synthesis balances timing, power, and area by selecting appropriate flavors.

### Execution Command
```
yosys
-> read_liberty -lib ../lib/sky10_fd_sc_hd_tt_025C_1v80.lib
-> read_verilog good_mux.v
-> synth -top goodmux
-> abc -liberty ../lib/sky10_fd_sc_hd_tt_025C_1v80.lib
-> show
-> write_verilog -noattr good_mux_netlist.v
```
<br>

### Gate-Level Structure (`good_mux.v`)
<div align="center">
  <img src="https://github.com/GOKUL-D-10/SoC_Tapeout_Week1/blob/main/Day%201/images/netlist.jpeg" width="70%">
</div>
<br>

### Netlist (`good_mux.v`):
```
/* Generated by Yosys 0.57+153 (git sha1 6b3a7e244, g++ 14.2.0-19ubuntu2 -fPIC -O3) */

module good_mux(i0, i1, sel, y);
  input i0;
  wire i0;
  input i1;
  wire i1;
  input sel;
  wire sel;
  output y;
  wire y;
  wire 0;
  wire 1;
  wire 2;
  wire 3;
  sky130_fd_sc_hd_mux2_1 _4 (
    .A0(0),
    .A1(1),
    .S(2),
    .X(3)
  );
  assign 0 = i0;
  assign 1 = i1;
  assign 2 = sel;
  assign y = 3;
endmodule
```

---

## Summary
* ✅ Simulated Verilog design using iverilog
* ✅ Visualized waveforms in GTKWave
* ✅ Performed synthesis using Yosys
* ✅ Flavours of Gate and their need
* ✅ Generated a gate-level netlist with Sky130 cells
