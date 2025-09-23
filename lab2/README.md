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
