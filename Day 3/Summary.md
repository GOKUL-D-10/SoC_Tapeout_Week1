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
