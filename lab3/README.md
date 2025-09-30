# Lab 3 Report  
**Written by Colin McBride**  

-----------------------------------------------------------------------------------
# Part 1  

Main.v of Part 1:  
![IMG](img/p1code.png)  

- **Purpose:**  
  Implement a gated RS latch using logic gates in Verilog.  

- **Inputs:**  
  - `Clk` – Clock signal  
  - `S` – Set input  
  - `R` – Reset input  

- **Outputs:**  
  - `Q` – Latch output  

- **Operation:**  
  - When `Clk = 1`, `Q` follows the inputs `S` and `R`.  
  - When `Clk = 0`, the latch holds its previous value.  
  - The state where `S = 1` and `R = 1` is invalid.  

-----------------------------------------------------------------------------------
# Part 2  

Main.v of Part 2:  
![IMG](img/p2code.png)  

- **Purpose:**  
  Implement a gated D latch using Verilog.  

- **Inputs:**  
  - `D` – Data input  
  - `Clk` – Clock signal  

- **Outputs:**  
  - `Q` – Latch output  

- **Operation:**  
  - When `Clk = 1`, the latch is transparent (`Q = D`).  
  - When `Clk = 0`, the output `Q` holds the last value.  
  - Uses `S = D` and `R = ~D` internally to drive an RS latch.  

-----------------------------------------------------------------------------------
# Part 3  

Main.v of Part 3:  
![IMG](img/p3code.png)  

- **Purpose:**  
  Build a Master–Slave D flip-flop using two gated D latches.  

- **Inputs:**  
  - `D` – Data input  
  - `Clk` – Clock signal  

- **Outputs:**  
  - `Q` – Flip-flop output  

- **Operation:**  
  - The master latch is enabled when `Clk = 1`.  
  - The slave latch is enabled when `Clk = 0`.  
  - Together, this forms an edge-triggered flip-flop that updates only on the clock’s rising edge.  

-----------------------------------------------------------------------------------
# Part 4  

Main.v of Part 4:  
![IMG](img/p4code.png)  

- **Purpose:**  
  Compare a gated D latch with positive-edge and negative-edge triggered flip-flops.  

- **Inputs:**  
  - `D` – Data input  
  - `Clk` – Clock signal  
  - `reset_n` – Active-low reset  

- **Outputs:**  
  - `Q_latch` – Behavioral gated D latch output  
  - `Q_pos` – Positive-edge D flip-flop output  
  - `Q_neg` – Negative-edge D flip-flop output  

- **Operation:**  
  - **Latch:** Transparent when `Clk = 1`.  
  - **Pos-edge FF:** Updates `Q` on the rising edge of the clock.  
  - **Neg-edge FF:** Updates `Q` on the falling edge of the clock.  

-----------------------------------------------------------------------------------
# Part 5  

Main.v of Part 5:  
![IMG](img/p5code.png)  

- **Purpose:**  
  Implement a 16-bit register and 7-segment display system on the DE2 board.  

- **Inputs:**  
  - `SW[9:0]` – Switches for input values  
  - `KEY1` – Clock input (store operation)  
  - `KEY0` – Active-low reset  

- **Outputs:**  
  - `HEX7–HEX4` – Stored value `A`  
  - `HEX3–HEX0` – Current switch value `B`  

- **Operation:**  
  - First, set the switches to a value for A.  
  - Press `KEY1` to store this value into the register.  
  - The stored value is displayed on HEX7–HEX4.  
  - Current switch values (B) are shown on HEX3–HEX0.  
  - Pressing `KEY0` resets the stored register.  

-----------------------------------------------------------------------------------
# Results  

- The RS latch, gated D latch, and master–slave flip-flop worked as expected.  
- The gated latch, pos-edge flip-flop, and neg-edge flip-flop all showed their distinct behaviors.  
- The register correctly stored switch inputs and displayed them separately from the current values.  

-----------------------------------------------------------------------------------
# Conclusion  

This lab reinforced the differences between **latches** (level-sensitive) and **flip-flops** (edge-sensitive). Gate-level implementations showed how storage can be built from simple logic, while behavioral Verilog mapped directly to FPGA flip-flop resources. The final register and 7-segment system demonstrated a practical use of storage elements in a digital circuit.  
