# Lab 3 – Latches, Flip-Flops, and Registers

---

## Part I – Gated RS Latch
**Code:**  
![Part I Code](img/p1code.png)  

**Observation:**  
- Output `Q` changes only when the clock input is high.  
- Invalid state occurs when both `S` and `R` are high.  

---

## Part II – Gated D Latch
**Code:**  
![Part II Code](img/p2code.png)  

**Observation:**  
- Transparent when `Clk = 1`.  
- Holds its value when `Clk = 0`.  

---

## Part III – Master-Slave D Flip-Flop
**Code:**  
![Part III Code](img/p3code.png)  

**Observation:**  
- Output updates only on clock edges.  
- Demonstrates edge-triggered behavior.  

---

## Part IV – Storage Elements Comparison
**Code:**  
![Part IV Code](img/p4code.png)  

**Observation:**  
- Behavioral latch is transparent while `Clk = 1`.  
- Positive-edge FF updates on rising edge.  
- Negative-edge FF updates on falling edge.  

---

## Part V – Register and 7-Segment Display
**Code:**  
![Part V Code](img/p5code.png)  

**Board Test:**  
- Switches (`SW`) select the B value.  
- Pressing `KEY1` stores A into the register.  
- `KEY0` resets the stored value.  
- HEX7–HEX4 show stored A, HEX3–HEX0 show current switches (B).  

---

## Results
- RS latch, D latch, and master–slave flip-flop all worked as expected.  
- Behavioral latch and edge-triggered flip-flops synthesized correctly on the FPGA.  
- Register successfully stored values and displayed them on the 7-segment LEDs.  

---

## Conclusion
This lab demonstrated how latches and flip-flops differ in operation, and how storage elements can be applied in practical digital systems. Implementing them in both gate-level and behavioral Verilog gave insight into synthesis results and FPGA hardware behavior.
