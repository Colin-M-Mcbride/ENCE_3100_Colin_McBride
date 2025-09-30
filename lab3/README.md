# Lab 3 – Latches, Flip-Flops, and Registers
**Course:** ENCE 3100  
**Name:** *Your Name*  
**Date:** *Date*  

---

## Purpose
The purpose of this lab is to explore how latches and flip-flops operate and how they can be described in Verilog. We implemented RS latches, D latches, master–slave flip-flops, and registers, and then verified them using simulation and on the DE2 board.

---

## Objectives
1. Implement and simulate a gated RS latch.  
2. Design and test a gated D latch.  
3. Construct a master–slave D flip-flop.  
4. Compare gated latches with positive- and negative-edge triggered flip-flops.  
5. Build a register with 7-segment displays for stored vs. current values.  

---

## Part I – Gated RS Latch
**Code:**  
![Part I Code](img/p1code.png)  

**Simulation Waveform:**  
![Part I Waveform](img/p1wave.png)  

**Observation:**  
- Output Q follows S and R only when clock = 1.  
- Invalid state occurs when both S and R are high.  

---

## Part II – Gated D Latch
**Code:**  
![Part II Code](img/p2code.png)  

**Simulation Waveform:**  
![Part II Waveform](img/p2wave.png)  

**Observation:**  
- Transparent when Clk = 1.  
- Holds value when Clk = 0.  

---

## Part III – Master-Slave D Flip-Flop
**Code:**  
![Part III Code](img/p3code.png)  

**Simulation Waveform:**  
![Part III Waveform](img/p3wave.png)  

**Observation:**  
- Output changes only on clock edges.  
- Demonstrates edge-triggered behavior.  

---

## Part IV – Storage Elements Comparison
**Code:**  
![Part IV Code](img/p4code.png)  

**Simulation Waveform:**  
![Part IV Waveform](img/p4wave.png)  

**Observation:**  
- Latch is transparent during high clock.  
- Pos-edge FF updates on rising edge.  
- Neg-edge FF updates on falling edge.  

---

## Part V – Register and 7-Segment Display
**Code:**  
![Part V Code](img/p5code.png)  

**Board Test:**  
- Switches (SW) load the B value.  
- Pressing KEY1 stores A.  
- HEX7–4 show A, HEX3–0 show B.  

---

## Results
- All designs simulated correctly.  
- Latches and flip-flops worked as expected.  
- Register stored and displayed values properly.  

---

## Conclusion
This lab highlighted the differences between latches and flip-flops, and demonstrated how storage elements are used in practical designs. The project reinforced the importance of clocking and edge sensitivity in sequential circuits.


