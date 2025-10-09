# Lab 4 – Counters

**Name:** Colin McBride  
**Course:** ENCE 3100  
**Board:** Altera DE2-Series FPGA  
**Date:** (Insert Date)

---

## Overview

This lab focused on creating several types of counters in Verilog and comparing how they’re implemented and synthesized.  
It moved from structural flip-flop designs to behavioral and parameterized designs, ending with display control on the DE2 board.  
Also I got lazy and learned I can just format the code in here

---

## Part I – 8-bit Counter Using T Flip-Flops

The first design used individual T flip-flops connected in a synchronous chain.  
Each flip-flop toggles based on the enable signal and the state of the lower bits.

### Code Example

```verilog
module counter_8bit(
    input clk,
    input ena,
    input clear,
    output [7:0] q
); 

wire [7:0] out;

// calculate T inputs
assign out[0] = ena & q[0];
assign out[1] = q[1] & out[0];
assign out[2] = q[2] & out[1];
assign out[3] = q[3] & out[2];
assign out[4] = q[4] & out[3];
assign out[5] = q[5] & out[4];
assign out[6] = q[6] & out[5];
assign out[7] = q[7];

// instantiate T flip-flops
t_ff t0 (.clk(clk), .reset(~clear), .t(ena), .q(q[0]));
t_ff t1 (.clk(clk), .reset(~clear), .t(out[0]), .q(q[1]));
t_ff t2 (.clk(clk), .reset(~clear), .t(out[1]), .q(q[2]));
t_ff t3 (.clk(clk), .reset(~clear), .t(out[2]), .q(q[3]));
t_ff t4 (.clk(clk), .reset(~clear), .t(out[3]), .q(q[4]));
t_ff t5 (.clk(clk), .reset(~clear), .t(out[4]), .q(q[5]));
t_ff t6 (.clk(clk), .reset(~clear), .t(out[5]), .q(q[6]));
t_ff t7 (.clk(clk), .reset(~clear), .t(out[6]), .q(q[7]));

endmodule
```

### Notes
- Structural design using individual flip-flops.  
- Works correctly in simulation and on hardware.  
- RTL viewer shows flip-flops with AND gates forming the enable logic.  

---

## Part II – 16-bit Register Counter

A much simpler implementation that uses a register incremented each clock cycle.  
This version replaces the structural logic with a behavioral statement.

### Code Example

```verilog
module counter_16bit(
    input clk,
    input ena,
    input clear,
    output reg [15:0] q
);

always @(posedge clk) begin
    if (!clear)
        q <= 16'b0;
    else if (ena)
        q <= q + 1'b1;
end

endmodule
```

### Notes
- Uses fewer logic elements and compiles faster.  
- RTL viewer shows one register with an adder feeding back.  
- Easier to scale to larger widths.  

---

## Part III – LPM Counter

This version uses the Quartus **Library of Parameterized Modules (LPM)**.  
Quartus generates a counter block automatically.

### Code Example

```verilog
module counter_16bit_LPM(
    input clk,
    input ena,
    input clear,
    output [15:0] q
);

lpm_counter LPM_COUNTER_component (
    .clock (clk),
    .cnt_en (ena),
    .sclr (~clear),
    .q (q)
);
defparam
    LPM_COUNTER_component.lpm_direction = "UP",
    LPM_COUNTER_component.lpm_width = 16;

endmodule
```

### Notes
- Functionally identical to Part II.  
- LPM block shows as a single module in the RTL viewer.  
- Simplest and most resource-efficient version.  

---

## Part IV – Flashing Digits (HEX0)

This circuit uses a large counter to divide the 50 MHz clock down to one second.  
Each second, the displayed digit increments from 0 to 9.

### Code Example

```verilog
module flash_digits(
    input clk,
    input clear,
    output [6:0] HEX0
);

reg [25:0] counter = 0;
reg [3:0] digit = 0;

always @(posedge clk) begin
    if (!clear) begin
        counter <= 0;
        digit <= 0;
    end
    else if (counter == 26'd49_999_999) begin
        counter <= 0;
        digit <= (digit == 9) ? 0 : digit + 1'b1;
    end
    else
        counter <= counter + 1'b1;
end

hex7seg h0 (.in(digit), .out(HEX0));

endmodule
```

### Notes
- Demonstrates clock division using counters.  
- Operates directly on the 50 MHz system clock.  
- Each number displays for approximately one second.  

---

## Part V – “HELLO” Ticker Display

Displays the word “HELLO” scrolling across HEX7–HEX0.  
The display shifts left once per second, looping continuously.

### Code Example

```verilog
module hello_ticker(
    input clk,
    input clear,
    output [6:0] HEX0, HEX1, HEX2, HEX3, HEX4, HEX5, HEX6, HEX7
);

reg [25:0] counter = 0;
reg [3:0] index = 0;

always @(posedge clk) begin
    if (!clear) begin
        counter <= 0;
        index <= 0;
    end
    else if (counter == 26'd49_999_999) begin
        counter <= 0;
        index <= (index == 8) ? 0 : index + 1'b1;
    end
    else
        counter <= counter + 1'b1;
end

letter7seg L7 (.in(3'd0), .out(HEX7)); // H
letter7seg L6 (.in(3'd1), .out(HEX6)); // E
letter7seg L5 (.in(3'd2), .out(HEX5)); // L
letter7seg L4 (.in(3'd2), .out(HEX4)); // L
letter7seg L3 (.in(3'd3), .out(HEX3)); // O
assign HEX2 = 7'b1111111;
assign HEX1 = 7'b1111111;
assign HEX0 = 7'b1111111;

endmodule
```

### Notes
- Uses the same one-second timer logic.  
- Letters stored as codes and displayed using a 7-segment pattern module.  
- Demonstrates practical use of counters for visual output.

---

## Summary

| Part | Counter Type | Width | Implementation | Key Notes |
|------|---------------|--------|----------------|------------|
| I | T flip-flop chain | 8-bit | Structural | Manual flip-flop logic |
| II | Register + 1 | 16-bit | Behavioral | Compact and efficient |
| III | LPM counter | 16-bit | Quartus block | Simplest implementation |
| IV | Flash digits | 26-bit timer | Behavioral | 1-second interval display |
| V | HELLO ticker | Multi-display | Behavioral | Scrolling word display |

---

## Conclusion

This lab demonstrated multiple ways to implement counters in Verilog.  
The structural T flip-flop design provides a good understanding of internal logic, while the behavioral and LPM approaches are faster and more efficient for real projects.  
The final parts applied timing control to create visible, time-based outputs using the DE2’s 7-segment displays.

---

## Files Created

- `t_ff.v`  
- `counter_8bit.v`  
- `counter_16bit.v`  
- `counter_16bit_LPM.v`  
- `flash_digits.v`  
- `hello_ticker.v`  
- (optional) `main.v` for testing multiple parts

