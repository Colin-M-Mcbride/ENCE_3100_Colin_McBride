
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
## Part 3:
<p align="center">**An array multiplier circuit:**</p>

![IMG](img/multiply.PNG)


## Part 4: 


What went wrong. 
pt 1: In correct output types 

pt 2: Incorrect output types again

pt 3: my wire naming scheme was so bad I actually got lost trying to build it all leading to errors with certian number values. I tried using GPT to help but it made everything so much worse by insiting on doing it it's own way 

pt 4: 
