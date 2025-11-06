# RISC-V Machine  
--- 
# Total TODO:  
*RTL for modules*   
adding RTL's for every module is just absurd  
---
![IMG](img/RISC-V.PNG)

---
Let's start with expanding parts from our old VSM.  
## ALU
```verilog
module Arithmetic_Unit (
    input [31:0] A,
    input [31:0] B,
    input [3:0] ALU_Control,
    output reg [31:0] ALU_Result,
    output Zero
);
    localparam ADD  = 4'b0000;
    localparam SUB  = 4'b0001;
    localparam AND  = 4'b0010;
    localparam OR   = 4'b0011;
    localparam XOR  = 4'b0100;
    localparam SLT  = 4'b0101;  // Set less than
    localparam SLL  = 4'b0110;  // Shift left logical
    localparam SRL  = 4'b0111;  // Shift right logical
    localparam SRA  = 4'b1000;  // Shift right arithmetic
    localparam MUL  = 4'b1001;  // Multiply (M extension)
    
    always @(*) begin
        case(ALU_Control)
            ADD:  ALU_Result = A + B;
            SUB:  ALU_Result = A - B;
            AND:  ALU_Result = A & B;
            OR:   ALU_Result = A | B;
            XOR:  ALU_Result = A ^ B;
            SLT:  ALU_Result = ($signed(A) < $signed(B)) ? 32'd1 : 32'd0;
            SLL:  ALU_Result = A << B[4:0];
            SRL:  ALU_Result = A >> B[4:0];
            SRA:  ALU_Result = $signed(A) >>> B[4:0];
            MUL:  ALU_Result = (A * B);  // Uses DSP blocks
            default: ALU_Result = 32'd0;
        endcase
    end
    
    assign Zero = (ALU_Result == 32'd0);
endmodule
```
```verilog
/* 
--------------------------------
ALU TEST
--------------------------------
*/
Arithmetic_Unit (
    .A (SW[9:6]), // Variable values of A for testing opperations
    .B (5'b00101), // Loading a fixed value of 5 just cause
    .ALU_Control (SW[3:0]), 
    .ALU_Result (LEDR [6:0]), // Display binary value
    .Zero (LEDR[9]) // If ay result is 0
);
endmodule
```
## Registers
### RegisterFile.v
```verilog
module RegisterFile (
    input clk,
	 input reset,
    input [4:0] rs1_addr,      // Read port 1 (5 bits = 32 regs)
    input [4:0] rs2_addr,      // Read port 2
    input [4:0] rd_addr,       // Write port
    input [31:0] rd_data,      // Data to write
    input reg_write,           // Write enable
    output [31:0] rs1_data,    // Read data 1
    output [31:0] rs2_data     // Read data 2
);

    // Storage for 32 registers
    reg [31:0] registers [0:31]; // A register of registers
    
    // Read ports: x0 always returns zero, others return stored value
    assign rs1_data = (rs1_addr == 5'd0) ? 32'd0 : registers[rs1_addr];
    assign rs2_data = (rs2_addr == 5'd0) ? 32'd0 : registers[rs2_addr];
    
    // Write port: block writes to x0
    always @(posedge clk) begin
        if (reset) begin
            integer i;
            for (i = 0; i < 32; i = i + 1)
                registers[i] <= 32'd0;
        end
        else if (reg_write && rd_addr != 5'd0)
            registers[rd_addr] <= rd_data;
    end
    
endmodule
```
### Register test driver
```verilog
	//-------------------------------------
	// Registers test
	//-------------------------------------
    //-------------------------------------
    // Clock and Control Signals
    //-------------------------------------
    wire clk = MAX10_CLK1_50;
    wire reset = ~KEY[0];         // Press KEY[0] to reset
    wire reg_write = ~KEY[1];     // Press KEY[1] to write
    
    //-------------------------------------
    // Register File Connections
    //-------------------------------------
    wire [4:0] rs1_addr = SW[4:0];      // Lower 5 switches = read address 1
    wire [4:0] rs2_addr = 5'd1;         // Always read x1 on port 2
    wire [4:0] rd_addr = SW[9:5];       // Upper 5 switches = write address
    
    // Test data - increment on each write
    reg [31:0] write_counter;
    
    always @(posedge clk) begin
        if (reset)
            write_counter <= 32'd0;
        else if (reg_write)
            write_counter <= write_counter + 1;
    end
    
    // Register file outputs
    wire [31:0] rs1_data;
    wire [31:0] rs2_data;
    
    //-------------------------------------
    // Instantiate Register File
    //-------------------------------------
    RegisterFile regfile(
        .clk(clk),
        .reset(reset),
        .rs1_addr(rs1_addr),
        .rs2_addr(rs2_addr),
        .rd_addr(rd_addr),
        .rd_data(write_counter),
        .reg_write(reg_write),
        .rs1_data(rs1_data),
        .rs2_data(rs2_data)
    );
    
    //-------------------------------------
    // Display on LEDs
    //-------------------------------------
    assign LEDR[9:5] = rd_addr;   // Show write address
    assign LEDR[4:0] = rs1_addr;  // Show read address
    
    //-------------------------------------
    // Display on 7-Segment (lower 24 bits of rs1_data)
    //-------------------------------------
    seg7Decoder hex0(.i_bin(rs1_data[3:0]),   .o_HEX(HEX0));
    seg7Decoder hex1(.i_bin(rs1_data[7:4]),   .o_HEX(HEX1));
    seg7Decoder hex2(.i_bin(rs1_data[11:8]),  .o_HEX(HEX2));
    seg7Decoder hex4(.i_bin(rs1_data[15:12]), .o_HEX(HEX4));
    seg7Decoder hex5(.i_bin(rs1_data[19:16]), .o_HEX(HEX5));

```
### Further Explanations of RegisterFile  
![IMG](img/REG.PNG)  
Truly absurd   
![IMG](img/REGZOOM.PNG)
The line of code: 
```verilog
reg [31:0] registers [0:31]; // A register of registers
```  
Creates 32 D flip flops per register address creating a sort of hash table of registers where each registers address is the key. 
## Controller and ROM  
*TODO:*  
Updated FSM Diagram (mermaid)  
Updated ROM table  
**after this point we have essenially built what we need.**

