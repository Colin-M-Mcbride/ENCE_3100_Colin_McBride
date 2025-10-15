
#### ENCE 3100 Lab 5
Colin McBride
  
## Part 1:  
Main of part 1
```verilog
	// Part I
	//******************
	
	wire [7:0] w_sum;
	
	// ALU - add
	accumulator_8bit Acc_8bit(
		.i_A(SW[7:0]),  //8bit
		.i_clk(w_myClk),
		.i_rst(SW[9]),
		.o_overflow(LEDR[8]),
		.o_S(w_sum) //8bit
	);
	
	assign LEDR[7:0] = w_sum;
	
	// Display result in BCD
	
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
*text blurb*  
```verilog
module accumulator_8bit(
	input 	[7:0] i_A,
	input 			i_clk,
	input				i_rst,
	output 			o_overflow,
	output 	[7:0] o_S
);


	wire [7:0] w_A;
	wire w_overflow;
	wire [7:0] w_S;

	reg_nbit #(.N(8)) REG0
	(
	i_A,
	i_clk,
	i_rst,
	w_A
	);
	

	// 8bit ALU
	// Now we accumulate
	assign {w_overflow, w_S} = w_A + o_S;
	
	// 1bit Register
	reg_nbit #(.N(1)) REG1
	(
	w_overflow,
	i_clk,
	i_rst,
	o_overflow
	);
	// 8bit Register
	reg_nbit #(.N(8)) REG2
	(
	w_S,
	i_clk,
	i_rst,
	o_S
	);
endmodule
```
## Part 2: 
Main of part 2:
```verilog
// Part II
	//******************

	
	wire [7:0] w_sum;
	
	// ALU - add and sub
	accumulator_sub_8bit Acc__sub_8bit(
		.i_A(SW[7:0]),  		//8bit
		.i_addsub(SW[8]), 	// 1 - sub / 0 - add
		.i_clk(w_myClk),
		.i_rst(SW[9]),
		.o_overflow(LEDR[8]),
		.o_S(w_sum) 			//8bit
	);
	
	assign LEDR[7:0] = w_sum;
	
	// Display result in BCD
	
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

## Part 3:

main of part 3:
```verilog
//******************
	
	// Part III
	//******************	

	
	assign HEX3 = 8'b11111111;  // B
	seg7Decoder SEG_A (SW[3:0], HEX4);  // A
	seg7Decoder SEG_B (SW[7:4], HEX5);  // B
	
	wire [7:0] w_Product;
	
	multiplier_4x4 MT_4by4 (
		.i_A(SW[3:0]),  // 4bits
		.i_B(SW[7:4]),  // 4bits
		.o_P(w_Product),  // 8bits
		.o_Overflow(LEDR[8])
	);
	
	assign LEDR[7:0] = w_Product;
	
	// Display result in BCD
	
	wire [3:0] w_O, w_T, w_H;
	
	bin8_to_bcd u_b2b (
    .bin(w_Product),
    .bcd_hundreds(w_H),
    .bcd_tens(w_T),
    .bcd_ones(w_O)
	);
	
	// BCD to HEX decoders
	seg7Decoder SEG_O (w_O, HEX0);
	seg7Decoder SEG_T (w_T, HEX1);
	seg7Decoder SEG_H (w_H, HEX2);
	```

<p align="center">**An array multiplier circuit:**</p>

![IMG](img/multiply.PNG)

```verilog
`default_nettype none

module multiplier_4x4(
    input  [3:0] i_A,
    input  [3:0] i_B,
    output [7:0] o_P,
    output       o_Overflow
);

    // Partial products
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

    // Internal wires
    wire s12, c12, s13, c13, s14, c14;
    wire s22, c22, s23, c23, s24, c24;
    wire s32, c32, s33, c33, s34, c34;
	 wire c11, c21, c31; 
    // Row 1
    assign o_P[0] = a0b0;
    FullAdder fa11(.i_a(a1b0), .i_b(a0b1), .i_cin(1'b0), .o_sum(o_P[1]), .o_cout(c12));
    FullAdder fa12(.i_a(a2b0), .i_b(a1b1), .i_cin(c12), .o_sum(s12), .o_cout(c13));
    FullAdder fa13(.i_a(a3b0), .i_b(a2b1), .i_cin(c13), .o_sum(s13), .o_cout(c14));
    FullAdder fa14(.i_a(1'b0), .i_b(a3b1), .i_cin(c14), .o_sum(s14), .o_cout(c11)); // c14 cascades

    // Row 2
    FullAdder fa21(.i_a(s12), .i_b(a0b2), .i_cin(1'b0), .o_sum(o_P[2]), .o_cout(c22)); // fixed cin
    FullAdder fa22(.i_a(s13), .i_b(a1b2), .i_cin(c22), .o_sum(s22), .o_cout(c23));
    FullAdder fa23(.i_a(s14), .i_b(a2b2), .i_cin(c23), .o_sum(s23), .o_cout(c24));
    FullAdder fa24(.i_a(c11), .i_b(a3b2), .i_cin(c24), .o_sum(s24), .o_cout(c21)); // carry cascades

    // Row 3
    FullAdder fa31(.i_a(s22), .i_b(a0b3), .i_cin(1'b0), .o_sum(o_P[3]), .o_cout(c32)); // fixed cin
    FullAdder fa32(.i_a(s23), .i_b(a1b3), .i_cin(c32), .o_sum(s32), .o_cout(c33));
    FullAdder fa33(.i_a(s24), .i_b(a2b3), .i_cin(c33), .o_sum(s33), .o_cout(c34));
    FullAdder fa34(.i_a(c21), .i_b(a3b3), .i_cin(c34), .o_sum(s34), .o_cout(o_P[7]));

    // Output assembly
    assign o_P[4] = s32;
    assign o_P[5] = s33;
    assign o_P[6] = s34;
    //assign o_P[7] = c34; // final carry
    assign o_Overflow = 1'b0;

endmodule

`default_nettype wire
```
Nightmare to make
*gif of working multiplier*  

## Part 4: 


What went wrong. 
pt 1: In correct output types 

pt 2: Incorrect output types again

pt 3: my wire naming scheme was so bad I actually got lost trying to build it all leading to errors with certian number values. I tried using GPT to help but it made everything so much worse by insiting on doing it it's own way 

pt 4: 
