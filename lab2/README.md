how do I bold text  
**Lab 2 Report**  
**Written by Colin McBride**  

Logism Diagram for Part 1:  
![IMG](img/pt1Sim.png)

Main.v of Part 1:  
![IMG](img/pt1Code.png)  

7 Segment Decoder code:  
![IMG](img/pt1Code2.png)  

- **Purpose:**  
  Converts a 4-bit binary input into the 7-segment display pattern needed to light up digits `0–9`.
- **Inputs:**  
  - `i_m3, i_m2, i_m1, i_m0` → 4-bit binary input (most significant bit first).

- **Outputs:**  
  - `o_seg[7:0]` → The signals that control each LED segment (a–g, and sometimes DP).

- **Operation:**  
  Implements Boolean equations that map binary numbers `0000–1001` (decimal 0–9) into the correct LED on/off pattern.  
  For values `1010–1111` (decimal 10–15), the decoder treats them as **don’t-cares**, since only decimal digits are shown.
  
-----------------------------------------------------------------------------------
**Part 2**


-----------------------------------------------------------------------------------
**Part 3**


-----------------------------------------------------------------------------------
**Part 4**


-----------------------------------------------------------------------------------
**Part 5**

## Design Overview

### 1. Single-Digit BCD Adder (Part IV Module)
- **`adder_4bit`**: Adds two 4-bit BCD numbers, outputs a 4-bit sum and a carry-out.
- **`bin_to_dec_v2`**: Converts a 4-bit BCD sum (plus carry) to 7-segment display signals.
- **`checkBCD`**: Verifies valid BCD inputs and signals errors via LEDs.

### 2. Two-Digit BCD Adder (Top-Level Design)
- Two instances of the single-digit adder are instantiated hierarchically:
  - **Least Significant Digit (A0 + B0):** First adder takes lower 4 bits of `A` and `B` from switches, with carry-in = 0. Produces `sum0` and `carry0`.
  - **Most Significant Digit (A1 + B1):** Second adder takes upper 4 bits of `A` and a fixed `B1 = 4'b1111` (board limitation). Adds `carry0` to produce `sum1` and `carry1`.
- Three-digit sum `S2S1S0`:
  - `S0` = least significant sum (`sum0`)
  - `S1` = most significant sum (`sum1`)
  - `S2` = carry-out from second adder (`carry1`) representing hundreds digit
 
  
