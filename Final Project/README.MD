#RISC-V Machine  
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
## mmmm bus
