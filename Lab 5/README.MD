
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
## Part 2: 
## Part 3:
<p align="center">**An array multiplier circuit:**</p>

![IMG](img/multiply.PNG)


## Part 4: 


What went wrong. 
pt 1: In correct output types 

pt 2: Incorrect output types again

pt 3: my wire naming scheme was so bad I actually got lost trying to build it all leading to errors with certian number values. I tried using GPT to help but it made everything so much worse by insiting on doing it it's own way 

pt 4: 
