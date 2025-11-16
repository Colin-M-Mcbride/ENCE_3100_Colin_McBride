#### ENCE 3100 Lab 5
Colin McBride
  
## Part 1: 8-bit Accumulator with BCD Display
**What it does:** Takes an 8-bit input from switches and continuously adds it to a running sum. Displays the result on 7-segment displays in decimal (BCD format).

Main of part 1
```verilog
// Part I
//******************

wire [7:0] w_sum;

// ALU - Accumulator that adds SW input to current sum
accumulator_8bit Acc_8bit(
    .i_A(SW[7:0]),        // 8-bit input from switches
    .i_clk(w_myClk),      // Clock signal
    .i_rst(SW[9]),        // Reset switch clears accumulator
    .o_overflow(LEDR[8]), // Overflow indicator LED
    .o_S(w_sum)           // 8-bit sum output
);

assign LEDR[7:0] = w_sum; // Display sum on LEDs

// Convert binary sum to BCD for 7-segment display
wire [3:0] w_O, w_T, w_H; // Ones, Tens, Hundreds digits

bin8_to_bcd u_b2b (
    .bin(w_sum),
    .bcd_hundreds(w_H),
    .bcd_tens(w_T),
    .bcd_ones(w_O)
);

// Drive 7-segment displays with BCD values
seg7Decoder SEG_O (w_O, HEX0); // Ones place
seg7Decoder SEG_T (w_T, HEX1); // Tens place
seg7Decoder SEG_H (w_H, HEX2); // Hundreds place
```

**Accumulator Module:**
```verilog
module accumulator_8bit(
    input   [7:0] i_A,
    input         i_clk,
    input         i_rst,
    output        o_overflow,
    output  [7:0] o_S
);

    wire [7:0] w_A;
    wire w_overflow;
    wire [7:0] w_S;

    // Register to hold input value
    reg_nbit #(.N(8)) REG0
    (
        i_A,
        i_clk,
        i_rst,
        w_A
    );
    
    // Add registered input to current sum (accumulate)
    assign {w_overflow, w_S} = w_A + o_S;
    
    // Register to hold overflow bit
    reg_nbit #(.N(1)) REG1
    (
        w_overflow,
        i_clk,
        i_rst,
        o_overflow
    );
    
    // Register to hold sum (feedback for accumulation)
    reg_nbit #(.N(8)) REG2
    (
        w_S,
        i_clk,
        i_rst,
        o_S
    );
endmodule
```

**How it works:** Input gets registered, added to the current sum, then that new sum is registered and fed back for the next addition. This creates an accumulator that continuously adds new inputs to the running total.

*[INSERT PICTURE/GIF OF WORKING ACCUMULATOR HERE]*

---

## Part 2: Accumulator with Add/Subtract
**What it does:** Same as Part 1 but SW[8] lets you choose between adding (0) or subtracting (1).

Main of part 2:
```verilog
// Part II - Add/Subtract Accumulator
//******************

wire [7:0] w_sum;

// ALU with add/subtract capability
accumulator_sub_8bit Acc__sub_8bit(
    .i_A(SW[7:0]),        // 8-bit input from switches
    .i_addsub(SW[8]),     // 1 = subtract, 0 = add
    .i_clk(w_myClk),
    .i_rst(SW[9]),        // Reset accumulator
    .o_overflow(LEDR[8]), // Overflow/underflow indicator
    .o_S(w_sum)           // 8-bit result
);

assign LEDR[7:0] = w_sum; // Show result on LEDs

// Display result in decimal on 7-segment displays
wire [3:0] w_O, w_T, w_H;

bin8_to_bcd u_b2b (
    .bin(w_sum),
    .bcd_hundreds(w_H),
    .bcd_tens(w_T),
    .bcd_ones(w_O)
);

// BCD to HEX decoders
seg7Decoder SEG_O (w_O, HEX0);
seg7Decoder SEG_T (w_T, HEX1);
seg7Decoder SEG_H (w_H, HEX2);
```

*[INSERT PICTURE/GIF SHOWING BOTH ADD AND SUBTRACT WORKING]*

---

## Part 3: 4x4 Array Multiplier

**What it does:** Multiplies two 4-bit numbers (A and B) using combinational logic to produce an 8-bit product.

main of part 3:  
```verilog
//******************
// Part III - 4x4 Multiplier
//******************    

// Display input values on 7-segment displays
assign HEX3 = 8'b11111111;          // Blank display
seg7Decoder SEG_A (SW[3:0], HEX4);  // Display A value
seg7Decoder SEG_B (SW[7:4], HEX5);  // Display B value

wire [7:0] w_Product;

// 4x4 multiplier: A[3:0] × B[7:4] = Product[7:0]
multiplier_4x4 MT_4by4 (
    .i_A(SW[3:0]),        // 4-bit input A
    .i_B(SW[7:4]),        // 4-bit input B
    .o_P(w_Product),      // 8-bit product
    .o_Overflow(LEDR[8])  // Not used (always 0)
);

assign LEDR[7:0] = w_Product; // Show product in binary on LEDs

// Convert product to decimal for display
wire [3:0] w_O, w_T, w_H;

bin8_to_bcd u_b2b (
    .bin(w_Product),
    .bcd_hundreds(w_H),
    .bcd_tens(w_T),
    .bcd_ones(w_O)
);

// Display decimal result on 7-segment displays
seg7Decoder SEG_O (w_O, HEX0);
seg7Decoder SEG_T (w_T, HEX1);
seg7Decoder SEG_H (w_H, HEX2);
```

<p align="center">**An array multiplier circuit:**</p>

![IMG](img/multiply.PNG)

**Multiplier Implementation:**
```verilog
`default_nettype none

module multiplier_4x4(
    input  [3:0] i_A,
    input  [3:0] i_B,
    output [7:0] o_P,
    output       o_Overflow
);

    // Generate all partial products (AND gates)
    wire a0b0 = i_A[0] & i_B[0];
    wire a1b0 = i_A[1] & i_B[0];
    wire a2b0 = i_A[2] & i_B[0];
    wire a3b0 = i_A[3] & i_B[0];

    wire a0b1 = i_A[0] & i_B[1];
    wire a1b1 = i_A[1] & i_B[1];
    wire a2b1 = i_A[2] & i_B[1];
    wire a3b1 = i_A[3] & i_B[1];

    wire a0b2 = i_A[0] & i_B[2];
    wire a1b2 = i_A[1] & i_B[2];
    wire a2b2 = i_A[2] & i_B[2];
    wire a3b2 = i_A[3] & i_B[2];

    wire a0b3 = i_A[0] & i_B[3];
    wire a1b3 = i_A[1] & i_B[3];
    wire a2b3 = i_A[2] & i_B[3];
    wire a3b3 = i_A[3] & i_B[3];

    // Wires for sum and carry propagation through the array
    wire s12, c12, s13, c13, s14, c14;
    wire s22, c22, s23, c23, s24, c24;
    wire s32, c32, s33, c33, s34, c34;
    wire c11, c21, c31; 
    
    // Row 1: First level of addition
    assign o_P[0] = a0b0; // LSB is just the first partial product
    FullAdder fa11(.i_a(a1b0), .i_b(a0b1), .i_cin(1'b0), .o_sum(o_P[1]), .o_cout(c12));
    FullAdder fa12(.i_a(a2b0), .i_b(a1b1), .i_cin(c12), .o_sum(s12), .o_cout(c13));
    FullAdder fa13(.i_a(a3b0), .i_b(a2b1), .i_cin(c13), .o_sum(s13), .o_cout(c14));
    FullAdder fa14(.i_a(1'b0), .i_b(a3b1), .i_cin(c14), .o_sum(s14), .o_cout(c11));

    // Row 2: Second level
    FullAdder fa21(.i_a(s12), .i_b(a0b2), .i_cin(1'b0), .o_sum(o_P[2]), .o_cout(c22));
    FullAdder fa22(.i_a(s13), .i_b(a1b2), .i_cin(c22), .o_sum(s22), .o_cout(c23));
    FullAdder fa23(.i_a(s14), .i_b(a2b2), .i_cin(c23), .o_sum(s23), .o_cout(c24));
    FullAdder fa24(.i_a(c11), .i_b(a3b2), .i_cin(c24), .o_sum(s24), .o_cout(c21));

    // Row 3: Final level
    FullAdder fa31(.i_a(s22), .i_b(a0b3), .i_cin(1'b0), .o_sum(o_P[3]), .o_cout(c32));
    FullAdder fa32(.i_a(s23), .i_b(a1b3), .i_cin(c32), .o_sum(s32), .o_cout(c33));
    FullAdder fa33(.i_a(s24), .i_b(a2b3), .i_cin(c33), .o_sum(s33), .o_cout(c34));
    FullAdder fa34(.i_a(c21), .i_b(a3b3), .i_cin(c34), .o_sum(s34), .o_cout(o_P[7]));

    // Assign final product bits
    assign o_P[4] = s32;
    assign o_P[5] = s33;
    assign o_P[6] = s34;
    // o_P[7] assigned in fa34
    assign o_Overflow = 1'b0; // No overflow possible with 4x4 mult

endmodule

`default_nettype wire
```

Nightmare to make - the wire naming got really confusing tracking all the carries and sums through the array!

*[INSERT PICTURE/GIF SHOWING MULTIPLIER WORKING - TRY 3×5=15, 7×7=49, ETC.]*

---

## Part 4: 8×8 Registered Multiplier

**What it does:** Implements an 8×8 multiplier using n-bit adders arranged hierarchically (instead of a 2D array of full adders like Part 3). This approach is more scalable and easier to extend to larger bit widths. Both inputs and outputs are registered.

**Key difference from Part 3:** Part 3 used a 2D array of individual full adders. Part 4 abstracts rows of full adders into n-bit adders, creating a cleaner structure that accumulates partial products row-by-row.

Main of part 4:
```verilog
//******************
// Part IV - 8x8 Registered Multiplier
//******************

// Display inputs in hexadecimal
seg7Decoder SEG_A1 (SW[15:12], HEX7); // A[7:4]
seg7Decoder SEG_A0 (SW[11:8], HEX6);  // A[3:0]
seg7Decoder SEG_B1 (SW[7:4], HEX5);   // B[7:4]
seg7Decoder SEG_B0 (SW[3:0], HEX4);   // B[3:0]

wire [15:0] w_Product;

// 8x8 multiplier with registered inputs and outputs
multiplier_8x8_reg MT_8by8_reg (
    .i_A(SW[15:8]),       // 8-bit A from switches
    .i_B(SW[7:0]),        // 8-bit B from switches
    .i_clk(w_myClk),      // Clock for registers
    .o_P(w_Product)       // 16-bit product output
);

// Display product in hexadecimal (4 digits)
seg7Decoder SEG_P0 (w_Product[3:0], HEX0);
seg7Decoder SEG_P1 (w_Product[7:4], HEX1);
seg7Decoder SEG_P2 (w_Product[11:8], HEX2);
seg7Decoder SEG_P3 (w_Product[15:12], HEX3);
```

**How it works:** 
- Registers capture input values A and B on clock edge
- Generate 8 partial products (A × each bit of B)
- Use cascaded n-bit adders to accumulate shifted partial products
- Final result is registered before output

**Testing:** Try multiplying values like:
- 15 × 15 = 225 (0xE1)
- 255 × 2 = 510 (0x1FE)
- 16 × 16 = 256 (0x100)

*[INSERT PICTURE/GIF SHOWING 8×8 MULTIPLIER WORKING]*

**Performance:**
- Logic Elements Used: [Check Compilation Report]
- Maximum Frequency (f_max): [Check Timing Analyzer]

---

## Challenges & Lessons Learned:

**Part 1:** Incorrect output types initially - had to fix port declarations

**Part 2:** Same output type issues

**Part 3:** My wire naming scheme was terrible and I got completely lost building the array. Tried using GPT for help but it made everything worse by insisting on its own structure instead of following the array multiplier pattern. Eventually debugged it manually by carefully tracing through the circuit diagram.

**Part 4:** [To be completed]
