# 🌟Day 5: Advanced Synthesis & Optimization 🚀


# 🎯 Block 1: Inferred Latches – Hidden Hazards in RTL Design

🎯 **Learning Objective**

Understand how incomplete conditional statements in Verilog can unintentionally create **inferred latches**, and learn best practices to avoid them.

🧩 **Key Focus Areas**

- ⚡ Proper use of if–else statements
- 🧮 Complete case statements
- ❗ Problems caused by missing assignments
- 🔧 Techniques to avoid latch inference

---

## 💡 **1. If – Else Statements in Verilog**

If – else statements control conditional execution inside procedural blocks such as **always**, **initial**, **tasks**, or **functions**.

Syntax example
```verilog

if (condition) begin

// Code executes if condition is true

end else begin

// Code executes if condition is false

end
```

✨ Notes

- ✅ **condition**: Must evaluate to true (non-zero) or false (zero).
- ✅ **begin ... end**: Group multiple statements. Can be skipped for a single statement.
- 🟢 **else** is optional but recommended for clarity.

🔀 **Nested If–Else Example**

```verilog
if (condition1) begin

// Executes when condition1 is true

end else if (condition2) begin

// Executes when condition2 is true

end else begin

// Executes when neither condition is true

end
```

---

## 💡 **2. Inferred Latches**

⚠️ An **inferred latch** occurs when a variable inside a combinational always block is **not assigned in every possible path**.

This forces the synthesis tool to create storage (a latch) so that the signal holds its previous value—often **not intended** by the designer.

Example of a latch being inferred :

```verilog
module ex (

input wire a, b, sel,

output reg y

);

always @(a, b, sel) begin

if (sel == 1'b1)

y = a; // ❌ No 'else' → y is undefined when sel = 0

end

endmodule
```

## ⚠️ Problem :

- When **sel** is 0, `y` is never updated.
- The tool infers a latch to preserve the old value of `y`.
- Latches can create **timing hazards** and **unpredictable behavior**.

---

✅ **Best Practices to Avoid Latch Inference**

1️⃣ **Provide a Default Assignment**

Assign a safe value to the variable at the start of the block.

2️⃣ **Always Use Else or Default in Case**

Cover all possibilities explicitly.

Corrected Example using case

```verilog
module correct_example (

input wire a, b, sel,

output reg y

);

always @(a, b, sel) begin

case (sel)

1'b1 : y = a;

default : y = 1'b0; // ✅ Ensures y is always assigned

endcase

end

endmodule
```



### 🚀 **Takeaway**

By ensuring every signal inside a combinational block is assigned in **all possible paths**, you eliminate unintended latches, improving design reliability and synthesis quality.

---

## 🎓 **Lab 1: Incomplete If Statement**



🔍 **Objective:**

Understand how incomplete `if` statements can lead to **inferred latches** and unpredictable behavior in RTL synthesis.

---

💻 **Module Example:**
```verilog
module incomp_if (input i0, input i1, input i2, output reg y);

always @(*) begin

if (i0)

y <= i1; // ❌ No else branch: y retains previous value when i0 == 0

end

endmodule
```

---

![If](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/ifwave.png)

⚠️ **Observation:**

- When `i0` is `0`, the output `y` is **not assigned**, so the synthesis tool infers a latch to store its previous value.
- This can lead to **functional mismatches** and **timing hazards** in your design.
- Hence a D-Latch is Inferred

---
![If](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/ifnet.png)


✅ **Best Practice:**

- Always include an `else` branch to cover all conditions.
- Alternatively, assign a **default value** at the beginning of the block:

```verilog
always @(*) begin
    y = 1'b0; // Default assignment
    if (i0)
        y <= i1;
end

```

## Nested Incomplete If : 
### Verilog Code :
![If2](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/if2code.png)

### Waveform Generated on Simulation of the above RTL Code :
![If2](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/if2wave.png)

---

⚠️ **Observation:**

- When `i0` is `1`, the output `y` is  assigned to `i1`, and  When `i2` is `1`, the output `y` is  assigned to `i3`so the synthesis tool infers a latch to store value when both are zero.
- This can lead to **functional mismatches** and **timing hazards** in your design.
- Hence a D-Latch is Inferred
- Hence the Netlist will be :

### Netlist Generated :
![If2](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/if2net.png)
  


### 💡 **Pro Tip:**

Even in small combinational modules, **never leave branches incomplete**. This practice ensures RTL remains purely combinational and avoids inferred latches.


---


# 🎯 **Block 2: Problems in Case Statements – Incomplete and Wildcard Usage**


🔍 **Objective:**

Learn how incomplete or improperly used `case` statements in Verilog can lead to **inferred latches**, unpredictable behavior, and synthesis warnings.

---
### 💻 ** Problematic Module Example:**

```verilog
    module incomp_case(input i0 , input i1 , input i2 , input[1:0] sel , output reg y);
    always @(*) begin
        case(sel)
            2'b00 : y = i0;
            2'b01 : y = i1;
        endcase
    end
    endmodule
```
### Waveform Generated for the RTL Code Simulation :
![case](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/casewave.png)

### ⚠️ Problem Description:

Only sel = 2'b00 and sel = 2'b01 are assigned values.

For sel = 2'b10 or 2'b11, y is not assigned, causing synthesis tools to infer a latch to hold the previous value.

This can lead to unintended memory behavior, functional mismatches, and timing hazards.

### Netlist Generated for the RTL Code :
![case](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/casenet.png)


✅ Solution – Add a Default Case:
```verilog
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b11: y = 1'b0;      // Explicit assignment
        default: y = 1'b0;    // Safe catch-all
    endcase
end
```


### 💻 ** Overlapping Problematic Module Example:**

```verilog
module bad_case (

input i0, input i1, input i2, input i3,

input [1:0] sel,

output reg y

);

always @(*) begin

case(sel)

2'b00: y = i0;

2'b01: y = i1;

2'b10: y = i2;

2'b1?: y = i3; // ❌ Wildcard usage; sel == 2'b11 is ambiguous

endcase

end

endmodule
```
### Waveform Generated on Simulation of the above RTL Code :
![If2](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/badcasewave.png)

---

### ⚠️ **Observation:**

- Using a **wildcard (`?`)** can unintentionally leave **some input combinations uncovered**.
- For example, `2'b1?` covers `10` and `11`, but if not handled carefully, synthesis tools may still infer **latches** for other unassigned cases.
- Incomplete coverage in `case` statements is a **common source of unintended storage elements** in combinational logic.
  

### 🖼 Demo Images:

Generated Netlist:
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/badcasenet.png)

Netlist Simulation Result:
![Simulation](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/bcnwave.png)

⚠️ Observation: The simulation shows unexpected retention of previous values when the input does not match explicit cases. This confirms the latch inference issue.

### ✅ Solution – Complete Coverage:
```verilog

always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b11: y = i3;     // Explicitly assign all combinations
        default: y = 1'b0; // Safe catch-all
    endcase
end
```


### 💡 Best Practices:

Avoid ambiguous wildcards unless fully intentional.

Always include a default: case to handle all unmatched inputs.

Verify with simulation and synthesis reports to catch any inferred latches early.

### ✨ Pro Tip:

Covering all cases prevents glitches, ensures combinational behavior, and improves RTL reliability for downstream synthesis and layout.

---




## 🎉 Demo: Complete Case Statement – Proper Coverage

🔹 Objective:
Demonstrate a fully covered case statement in Verilog that avoids inferred latches and works correctly for all input combinations.

### 💻 Example Module – Complete Case:
```verilog
module comp_case (
    input i0, input i1, input i2, 
    input [1:0] sel, output reg y
);
    always @(*) begin
        case(sel)
            2'b00: y = i0;   // ✅ Explicitly assigned
            2'b01: y = i1;   // ✅ Explicitly assigned
            default: y = i2; // ✅ Default handles remaining cases (2'b10, 2'b11)
        endcase
    end
endmodule
```


### ✨ Key Highlights:

🌟 All input combinations are covered either explicitly or via default.

🚀 Prevents unintended latches in combinational logic.

🔹 Simplifies synthesis and simulation verification.

💡 Ensures predictable and robust RTL behavior.

### 🖼 Demo Images:


* Simulation Result:
![Simulation](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/compcasewave.png)


* Generated Netlist:
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/compcasenet.png)



### Observation: 
```
The simulation confirms that outputs are correct for all input combinations and no latches are inferred.
```

### ✅ Best Practices Reminder:

 * Always cover all possible values of your case selector.

 * Use a default: branch for any remaining combinations.

 * Validate RTL behavior via simulation before synthesis.

 * Combine good coding practices with clear comments to make designs readable.

---

### 🎯 Pro Tip:

A complete case statement is one of the simplest ways to write safe combinational logic and avoid subtle synthesis bugs.

Use badges, emojis, and descriptive comments in documentation to make your repository visually appealing and learner-friendly!



## 🎉 Demo: Partial Assignment in Case Statements – Beware of Unassigned Signals

### 🔹 **Problem Overview:**

- When **multiple outputs** are assigned inside a case statement, it’s crucial that **all outputs are assigned in all branches**.
- Failure to do so can lead to **inferred latches** for the unassigned signals.
- In this example, `x` is **not assigned** in the `2'b01` branch, which can cause an **unintended latch** for `x`.

---

### 💻 **Partial Assignment Code:**

```verilog
module partial_case_assign (
    input i0, input i1, input i2,
    input [1:0] sel,
    output reg y, output reg x
);
always @(*) begin
    case(sel)
        2'b00: begin
            y = i0;
            x = i2;
        end
        2'b01: y = i1;           // ❌ x is not assigned here
        default: begin
            x = i1;
            y = i2;
        end
    endcase
end
endmodule

```

---

### 🖼 **Demo Image – Generated Netlist:**

![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/partialnet.png)

> Observation: The netlist shows that x is inferred as a latch in the branch where it is not explicitly assigned. This is undesirable for combinational logic.


---

### ✅ **Solution – Ensure All Outputs Are Assigned:**

```verilog
always @(*) begin
    case(sel)
        2'b00: begin
            y = i0;
            x = i2;
        end
        2'b01: begin
            y = i1;
            x = i1;      // ✅ Assign x in this branch
        end
        default: begin
            x = i1;
            y = i2;
        end
    endcase
end

```


### 💡 **Key Takeaways:**

- Always **assign every output signal in all branches** of a case statement.
- Use **default branches** to cover remaining combinations.
- Simulation and netlist verification help **catch inferred latches early**.
- Proper signal assignment ensures **clean combinational logic** and predictable behavior.

---

### 🎯 **Pro Tip:**

- In multi-output case statements, **think of each output independently** and make sure no branch leaves any signal undefined.
- Combine this with **visual netlist checks** to confirm all assignments are correctly mapped.
  
  ---

## Common Verilog RTL Issues & Best Practices when using If and Case Statements 

### **1. Problems & Solutions Table**

| **Problem** | **Solution** |
| --- | --- |
| Incomplete `case` | Use a `default` branch to cover all possible cases. |
| Incomplete `if` | Use an `else` branch to ensure all conditions are handled. |
| Partial assignment | Assign **all outputs** in every segment of `case` or `if` to avoid latches. |

---

### **2. Best Practices & Notes**

- **Use `reg` inside `always`**:
    
    If `if` or `case` statements are used inside an `always` block, the variables assigned must be declared as `reg`.
    
- **Priority difference between `if` and `case`**:
    - `if-else` gives **priority** to the first true condition.
    - `case` does **not have inherent priority**; overlapping cases can cause unexpected behavior.
- **Avoid overlapping cases**:
    
    Overlapping case conditions may cause **incorrect RTL simulation** results, though **netlist simulation** might still work as expected.
    

---

# Block 3 : 🔄 For Loops in Verilog

### 🔹 Overview

A for loop in Verilog is used inside procedural blocks (always, initial, tasks/functions) to execute repetitive statements based on a loop counter.

### 🔹 Syntax

```verilog
for (initialization; condition; increment) begin
    // Statements to execute
end
```


### Key Points:

✅ Must be inside procedural blocks.

✅ Synthesizable only if the number of iterations is fixed at compile time.

✅ Commonly used for MUXes, arrays, and repetitive assignments.

### 🔹 Example: 4-to-1 MUX Using a For Loop
```verilog
module mux_4to1_for_loop (
    input wire [3:0] data, // 4 input lines
    input wire [1:0] sel,  // 2-bit select
    output reg y           // Output
);
    integer i;
    always @(data, sel) begin
        y = 1'b0; // Default output
        for (i = 0; i < 4; i = i + 1) begin
            if (i == sel)
                y = data[i];
        end
    end
endmodule
```

### ⚡ Explanation

Default Assignment: y = 1'b0; prevents inferred latches. ⚠️

Loop Index i: Iterates through all inputs to check which matches the select signal.

Condition Check: if (i == sel) selects the correct input line. ✅

Scalable Design: Easy to extend for 8-to-1, 16-to-1 MUXes without writing long case statements.

### 🌟 Advantages

✅ Reduces code repetition.

✅ Ensures readability and maintainability.

✅ Synthesizes to the same hardware as a case-based MUX.

---

### Example 

Lets have a look at a ``Example`` code along with its waveform and Netlist Generation : 

```verilog
    module mux_generate (
    input i0, input i1, input i2, input i3,
    input [1:0] sel,
    output reg y
);
wire [3:0] i_int;
assign i_int = {i3, i2, i1, i0};
integer k;
always @(*) begin
    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y = i_int[k];
    end
end
endmodule
```

### 🔎 Simulation Waveform 📈

- The simulation waveform will illustrate how the **for loop selects one of the 4 inputs** (`i0–i3`) based on the **2-bit selector**.
- At each toggle of `sel`, the output `y` reflects the **corresponding input line** ✅.
- Each iteration of the loop is unrolled at synthesis, so **MUX behavior is clear and efficient** 🔬.
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/generatewave.png)

---

### 🏗️ Netlist View 🖼️

- When synthesized, the netlist will explicitly show a **4-to-1 multiplexer** built from logic gates.
- The `for` loop in RTL does **not remain a loop in hardware**; instead, it generates a real mux structure ⚡.
- With **Yosys + GTKWave**, you can confirm:
    - Correct functionality via simulation.
    - The actual gate-level design in the netlist 🔍.
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/generatenet.png)

---

### 🌟 Why Use For Loop Here?

- 🧩 **Compact Code** → Instead of writing multiple `case` or `if` blocks, a simple loop describes the logic.
- ⚙️ **Automation** → Cleaner RTL for scalable designs.
- ✅ **Synthesis Friendly** → As the loop has a **fixed number of iterations**, synthesis tools expand it properly into combinational hardware.

---

### ✅ Advantages of Generate Blocks

- ⚡ **Automation** – Save time writing repetitive instantiations.
- 🔄 **Scalability** – Easy to extend from 4 gates → 8, 16, or more by changing loop bounds.
- 🧩 **Hierarchy Naming** – Each generated instance has a unique hierarchical name.
- 📏 **Parameterization** – Great for designs where structure size depends on parameters.

✨ Generate blocks are a powerful Verilog feature that makes your designs scalable, maintainable, and cleaner. With waveform simulation and netlist inspection, you can clearly see how repetitive hardware is built automatically! 🚀

---



# ⚙️ Block 4: Generate Blocks in Verilog 🧩


## 🔹 What is a Generate Block?

A **generate block** in Verilog is used to **create repeated hardware structures** at *compile time*.

It allows you to **automatically replicate logic or module instances** instead of writing them manually.

### ✨ Common use cases:

- Repeating **logic structures** (e.g., gates, adders).
- Instantiating **multiple modules**.
- Creating **parameterized hardware**.

---

### 🔹 Key Ingredients

- **`genvar`** → Special variable type used in `generate` loops.
- **`for` loop inside generate`** → Defines how many copies of the logic/module to create.
- **Naming blocks (`: gen_loop`)** → Helpful for hierarchy debugging in simulation/netlist.

---

### 🔹 Example: Generate AND Gates

```verilog
genvar i;
generate
    for (i = 0; i < 4; i = i + 1) begin : gen_loop
        and_gate and_inst (.a(in[i]), .b(in[i+1]), .y(out[i]));
    end
endgenerate

```

✅ This will **instantiate 4 AND gates** automatically, connecting inputs and outputs systematically.

No need to write 4 separate instantiations manually 🚀.

---
 
# Block 5 : 🔀 DEMUX Implementations – Case vs For Loop


## 📌 1:8 DEMUX Using Case Statement 🧾

### 🔎 Simulation Waveform 📈

- The waveform shows how the **input signal `i`** is routed to one of the **eight outputs (o0–o7)** depending on the **3-bit select line**.
- At each change in `sel`, only one output goes **HIGH**, while others remain **LOW**.
- Easy to verify correctness step by step.
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/demuxwave.png)
  

### 🏗️ Netlist View 🖼️

- The synthesized netlist expands the **case-based conditional logic** into a network of gates.
- Each output is properly mapped to the input based on the select bits.
- This structure clearly highlights the **decoder-like expansion** required for DEMUX logic.
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/demuxnet.png)
---

## 📌 1:8 DEMUX Using For Loop ♻️

### 🔎 Simulation Waveform 📈

- The waveform again shows only **one output active at a time**, depending on `sel`.
- The **for loop version produces identical behavior** to the case-based DEMUX ✅.
- Using GTKWave, you’ll see that as `sel` iterates, the selected output follows `i`.
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/demuxgenwave.png)


### 🏗️ Netlist View 🖼️

- The for loop is **completely unrolled by the synthesis tool**.
- Netlist explicitly shows **eight separate logic paths**, just like in the case-based version.
- This demonstrates that **for loops don’t exist in hardware** — they’re just a compact way to describe repetitive logic.
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/demuxgennet.png)
---

## ⚖️ Case vs For Loop – Quick Comparison

- 🧾 **Case Statement** → Clear, explicit, and beginner-friendly.
- ♻️ **For Loop** → Cleaner code, scalable for larger DEMUX structures.
- 🏁 **Netlist Result** → Both approaches synthesize into identical hardware.

### ✨ **Takeaway:** Both styles are valid — but **for loops provide elegance** in code, especially when dealing with larger DEMUX or repetitive logic! 🎯

---


# Block 6 : ⚡ Ripple Carry Adder with Generate Block 🛠️

### 🟢 **Code for 8-bit RCA using Generate Block**

```verilog
// 8-bit Ripple Carry Adder with Generate Block
module rca (
    input [7:0] num1,
    input [7:0] num2,
    output [8:0] sum
);

    wire [7:0] int_sum;   // intermediate sums
    wire [7:0] int_co;    // intermediate carries

    genvar i;
    generate
        for (i = 1; i < 8; i = i + 1) begin
            fa u_fa_1 (
                .a(num1[i]),
                .b(num2[i]),
                .c(int_co[i-1]),
                .co(int_co[i]),
                .sum(int_sum[i])
            );
        end
    endgenerate

    // First FA (LSB with carry-in = 0)
    fa u_fa_0 (
        .a(num1[0]),
        .b(num2[0]),
        .c(1'b0),
        .co(int_co[0]),
        .sum(int_sum[0])
    );

    // Final output
    assign sum[7:0] = int_sum;
    assign sum[8]   = int_co[7];

endmodule

// 👉 Full Adder Module
module fa (input a, input b, input c, output co, output sum);
    assign {co, sum} = a + b + c;
endmodule

```

---

### 🔍 **Explanation**

- ✨ `genvar` + `generate` → creates repeated instances of Full Adders (FA).
- 💡 Carry of each FA is passed to the next FA in sequence → **Ripple Carry**.
- 🏗️ Structurally builds hardware → closer to how **real adders** work in silicon.
- 🚀 **Scalable design**: easily modified for 16-bit, 32-bit… just change loop range!

---

### 🎬 **RTL Simulation View**
![Netlist](https://github.com/Gowtham007007/Week-1_RISC-V_Tapeout/blob/main/Day_5/Images/rcatb.png)

- `sum = num1 + num2` (8-bit addition with 1-bit carry-out).
- Ripple effect: carries propagate from LSB → MSB in sequence.

---

## ⭐ Advantages of Generate Block

- 🔄 Eliminates manual instantiation of 8 FAs.
- 📏 Scales easily to larger adders.
- 📚 Clean, compact, and professional coding style.


# Summary
<div align = center>
# 🌟 **Day 5: Generate Blocks, Mux, Demux & RCA** 🚀
</div>


## 🔹 **1. Case Statements & Good Practices**

- ⚠️ **Incomplete `case` / `if`** → Can cause *latch inference*.
- ✅ Always use **default in case** and **else in if**.
- 📝 **Partial assignment** should be avoided → assign all outputs in every condition.
- ⚡ Priority:
    - `if` → priority-based
    - `case` → parallel comparison
- 🚫 Avoid **overlapping cases** → may lead to *RTL vs Netlist mismatches*.



## 🔹 **2. For Loops for Multiplexers**

- 🏗️ Replacing **case-based mux** with **for loop + if condition**.
- 💡 Cleaner, compact code.
- 🔬 Waveform shows **output following selected input**.
- 🖼️ Netlist → Synthesizer expands loop into multiple gates.



## 🔹 **3. Demultiplexer Designs**

### 🔸 **Demux using Case**

- Uses **case(sel)** → routes input `i` to correct output line.
- 🖥️ Simulation: only one output goes high based on `sel`.
- ⚙️ Netlist: expands into 8-output routing logic.

### 🔸 **Demux using For Loop**

- `for (k=0; k<8; k++)` → compares `k` with `sel`.
- Compact, scalable, avoids long case structures.
- Both **waveform** 📈 and **netlist** 🖼️ match with the case-based version.



## 🔹 **4. Ripple Carry Adder (RCA) with Generate Block**

- Built **8-bit RCA** using **generate-for** to replicate Full Adders.
- Carry ripples from LSB → MSB.
- 🖥️ RTL Simulation: outputs show correct addition.
- ⚙️ Netlist Simulation: synthesizer expands into real gate-level FA chain.
- ⭐ Advantages:
    - Scalable ✔️
    - Cleaner code ✔️
    - Hardware-accurate ✔️



# 📖 **Overall Takeaways from Day 5**

✅ Learned **best practices** to avoid mismatches in RTL vs Netlist.

✅ Explored **for loops vs case statements** in mux/demux design.

✅ Understood **simulation vs netlist views** for each code.

✅ Designed a **scalable RCA** with generate block 🏗️.


✨ Today was all about **automation, scalability, and clean coding in Verilog** — making designs **reusable, modular, and synthesizable** 🔬⚡.

---

# 🌟 **Day 5 Completed – Week 1 Wrapped Up!** 🎉

## ✨ *"Great designs aren’t written once, they’re generated infinitely."* ⚡

## ✨ *"Verilog is not just code – it’s hardware coming alive."* 💡

---

### 🗓️ **Week 1 – Journey So Far**

- ✅ Day 1 → RTL Design & Basics
- ✅ Day 2 → RTL Simulation & Gate-Level Insights
- ✅ Day 3 → If-Else, Case, Priority Concepts
- ✅ Day 4 → Netlist Generation & Issues
- ✅ Day 5 → For-Loops, Generate Blocks, RCA Design
