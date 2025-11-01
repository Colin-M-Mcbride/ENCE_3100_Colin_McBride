# Laboratory Exercise 1: Switches, Lights, and Multiplexers

## Objective
The purpose of this laboratory exercise was to learn how to connect simple input and output devices to an FPGA chip and implement circuits that use these devices. The DE2 board's 8 switches (SW₇₋₀) were used as inputs, and 8 LEDs (LEDR₇₋₀) and 6 seven-segment displays (HEX5-HEX0) were used as outputs.

---

## Part I: Simple Switch to LED Connection

### Description
A simple Verilog module was created to connect the 18 toggle switches directly to the 18 red LEDs on the DE2 board. This demonstrates basic input/output interfacing with the FPGA.

### Implementation
```verilog
module part1 (SW, LEDR);
    input [17:0] SW;    // toggle switches
    output [17:0] LEDR; // red LEDs
    assign LEDR = SW;
endmodule
```

### Results
The circuit was successfully compiled and downloaded to the DE2 board. Toggling each switch immediately reflected its state on the corresponding LED, confirming proper pin assignments and basic I/O functionality.

---

## Part II: 8-bit Wide 2-to-1 Multiplexer

### Description
An 8-bit wide 2-to-1 multiplexer was implemented using eight individual 2-to-1 multiplexers. The circuit selects between two 8-bit inputs (X and Y) based on a select signal (s).

### Truth Table
| s | Output M |
|---|----------|
| 0 | X        |
| 1 | Y        |

### Implementation
The multiplexer uses the Boolean expression: `m = (~s & x) | (s & y)` for each bit.

Switch assignments:
- SW₁₇: Select input (s)
- SW₇₋₀: Input X
- SW₁₅₋₈: Input Y
- LEDR₁₇₋₀: Display switch states
- LEDG₇₋₀: Output M

### Results
The 8-bit multiplexer functioned correctly. When SW₁₇ = 0, the output displayed the value of SW₇₋₀. When SW₁₇ = 1, the output displayed the value of SW₁₅₋₈.

---

## Part III: 3-bit Wide 5-to-1 Multiplexer

### Description
A 3-bit wide 5-to-1 multiplexer was constructed using multiple 2-to-1 multiplexers. The circuit selects one of five 3-bit inputs (U, V, W, X, Y) based on a 3-bit select signal.

### Truth Table
| s₂ | s₁ | s₀ | Output M |
|----|----|----|----------|
| 0  | 0  | 0  | U        |
| 0  | 0  | 1  | V        |
| 0  | 1  | 0  | W        |
| 0  | 1  | 1  | X        |
| 1  | 0  | 0  | Y        |
| 1  | 0  | 1  | Y        |
| 1  | 1  | 0  | Y        |
| 1  | 1  | 1  | Y        |

### Implementation
Switch assignments:
- SW₁₇₋₁₅: Select inputs (s₂s₁s₀)
- SW₁₄₋₁₂: Input U
- SW₁₁₋₉: Input V
- SW₈₋₆: Input W
- SW₅₋₃: Input X
- SW₂₋₀: Input Y
- LEDG₂₋₀: Output M

### Results
Each of the five inputs could be properly selected and displayed on the green LEDs by setting the appropriate select code on SW₁₇₋₁₅.

---

## Part IV: 7-Segment Decoder

### Description
A 7-segment decoder was implemented to display characters H, E, L, O, and blank on a 7-segment display based on a 3-bit input code.

### Character Code Table
| c₂ | c₁ | c₀ | Character |
|----|----|----|-----------|
| 0  | 0  | 0  | H         |
| 0  | 0  | 1  | E         |
| 0  | 1  | 0  | L         |
| 0  | 1  | 1  | O         |
| 1  | 0  | 0  | blank     |
| 1  | 0  | 1  | blank     |
| 1  | 1  | 0  | blank     |
| 1  | 1  | 1  | blank     |

### 7-Segment Encoding
The 7-segment display uses active-low logic (0 = on, 1 = off):
- H: 0001001
- E: 0000110
- L: 1000111
- O: 1000000
- blank: 1111111

### Implementation
Switch assignments:
- SW₂₋₀: Character code input
- HEX0: 7-segment display output

### Results
All five characters (H, E, L, O, blank) displayed correctly when the appropriate codes were set on SW₂₋₀.

---

## Part V: Rotating 5 Characters on 5 Displays

### Description
This circuit combines the 3-bit wide 5-to-1 multiplexer and 7-segment decoder to display and rotate a word across five 7-segment displays. Five instances of each module were used.

### Rotation Pattern for "HELLO"
| SW₁₇ | SW₁₆ | SW₁₅ | HEX4 | HEX3 | HEX2 | HEX1 | HEX0 |
|------|------|------|------|------|------|------|------|
| 0    | 0    | 0    | H    | E    | L    | L    | O    |
| 0    | 0    | 1    | E    | L    | L    | O    | H    |
| 0    | 1    | 0    | L    | L    | O    | H    | E    |
| 0    | 1    | 1    | L    | O    | H    | E    | L    |
| 1    | 0    | 0    | O    | H    | E    | L    | L    |

### Implementation
Each display's multiplexer receives the five character codes in a rotated order:
- HEX4: U, V, W, X, Y
- HEX3: V, W, X, Y, U
- HEX2: W, X, Y, U, V
- HEX1: X, Y, U, V, W
- HEX0: Y, U, V, W, X

Switch assignments:
- SW₁₇₋₁₅: Rotation select
- SW₁₄₋₀: Five 3-bit character codes

### Results
The word "HELLO" was successfully displayed and rotated across the five displays. Each rotation position showed the correct circular shift of the characters.

---

## Part VI: Rotating 5 Characters on 8 Displays

### Description
The design from Part V was extended to use all eight 7-segment displays. This required implementing 8-to-1 multiplexers and adding blank characters to properly position the 5-character word.

### Rotation Pattern for "HELLO"
| SW₁₇ | SW₁₆ | SW₁₅ | HEX7 | HEX6 | HEX5 | HEX4 | HEX3 | HEX2 | HEX1 | HEX0 |
|------|------|------|------|------|------|------|------|------|------|------|
| 0    | 0    | 0    | _    | _    | _    | H    | E    | L    | L    | O    |
| 0    | 0    | 1    | _    | _    | H    | E    | L    | L    | O    | _    |
| 0    | 1    | 0    | _    | H    | E    | L    | L    | O    | _    | _    |
| 0    | 1    | 1    | H    | E    | L    | L    | O    | _    | _    | _    |
| 1    | 0    | 0    | E    | L    | L    | O    | _    | _    | _    | H    |
| 1    | 0    | 1    | L    | L    | O    | _    | _    | _    | H    | E    |
| 1    | 1    | 0    | L    | O    | _    | _    | _    | H    | E    | L    |
| 1    | 1    | 1    | O    | _    | _    | _    | H    | E    | L    | L    |

*Note: _ represents blank character*

### Implementation
Eight 8-to-1 multiplexers were used (one per display). Each multiplexer selects from eight possible inputs that include the five character codes and three blank codes in appropriate positions.

### Results
The extended circuit successfully displayed "HELLO" with leading blanks and rotated the word through all eight positions. The rotation created a smooth circular animation effect as the word moved across all displays.

---

## Conclusion

This laboratory exercise successfully demonstrated:
1. Basic FPGA I/O interfacing with switches and LEDs
2. Implementation of multiplexer circuits in Verilog
3. 7-segment display control and character encoding
4. Hierarchical design using multiple module instances
5. Complex data routing to achieve rotating display effects

All circuits were synthesized, compiled, and tested on the DE2 board with correct functionality. The progressive complexity of each part built upon previous work, illustrating modular design principles in digital systems.

---

## Appendix: Pin Assignments

Pin assignments were imported from the DE2_pin_assignments.csv file provided by Altera. Key assignments include:
- SW[17:0]: Toggle switches
- LEDR[17:0]: Red LEDs
- LEDG[7:0]: Green LEDs
- HEX0-HEX7: 7-segment displays (each with [0:6] segments)

All pin names in the Verilog code matched those in the pin assignment file to ensure proper FPGA configuration.
