# RISC-V Machine  
--- 
# Total TODO:  
*RTL for modules*   
adding RTL's for every module is just absurd  
More explanations of certian things for easier readability + explanation of failures.  
Instruction Memory and Data Memory modules  
Branch Unit integration  
Control Unit FSM completion  
---
![IMG](img/RISC-V.PNG)

---
Let's start with expanding parts from our old VSM.  
## ALU
### ALU.v
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
    localparam SLTU = 4'b1001;  // Set Less Than Unsigned
    
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
            SLTU: ALU_Result = (A < B) ? 32'd1 : 32'd0;
            default: ALU_Result = 32'd0;
        endcase
    end
    
    assign Zero = (ALU_Result == 32'd0);
endmodule
```
### ALU Test Driver
```verilog
/* 
--------------------------------
ALU TEST
--------------------------------
*/
Arithmetic_Unit alu_test(
    .A (SW[9:6]), // Variable values of A for testing opperations
    .B (5'b00101), // Loading a fixed value of 5 just cause
    .ALU_Control (SW[3:0]), 
    .ALU_Result (LEDR [6:0]), // Display binary value
    .Zero (LEDR[9]) // If any result is 0
);
```
## R Type Instructions
With our ALU complete we can begin creating and understanding the first instruction set, R Type. R type are our simple functions done by the ALU.  
|Binary|Opperation|Explaination|
|---|---|---|
|0000|ADD|Adding two values|
|0001|SUB|Subtracting two values|
|0010|AND|And operation|
|0011|OR|Or operation|
|0100|XOR|Exclusive or opperation|
|0101|SLT|Set less than (signed)|
|0110|SLL|Shift logical left|
|0111|SRL|Shift logical right|
|1000|SRA|Shift right arithmetic|
|1001|SLTU|Set less than unsigned|  

Upon completion of the M extension the table will be updated with the following: 
|Binary|Opperation|Explaination|
|---|---|---|
|1010|MUL|Multiply two values|  

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

The line of code in RegisterFile.v: 
`reg [31:0] registers [0:31];` - Creates 32 D flip flops per register address creating a sort of hash table of registers where each registers address is the key and each sub register are the buckets. This can be seen in the RTL Viewer built into verilog.  

![IMG](img/REGZOOM.PNG)  
![IMG](img/REG2.PNG)  

## PC and Branch Unit
### ProgramCounter.v
```verilog
`default_nettype none

module ProgramCounter (
    input MainClock,
    input reset,              // Clear counter (renamed from ClearCounter)
    input PCWrite,            // Enable PC update
    input [1:0] PCSrc,        // Source select: 00=PC+4, 01=branch, 10=jump, 11=jalr
    input [31:0] branch_target,   // Branch target address
    input [31:0] jump_target,     // Jump target address (JAL)
    input [31:0] jalr_target,     // JALR target from ALU
    output reg [31:0] PC          // Program Counter (32-bit for RISC-V)
);

    // Internal signal for next PC value
    reg [31:0] next_pc;
    
    // Combinational logic for next PC calculation
    always @(*) begin
        case (PCSrc)
            2'b00:   next_pc = PC + 4;           // Normal: PC = PC + 4
            2'b01:   next_pc = branch_target;    // Branch taken
            2'b10:   next_pc = jump_target;      // JAL
            2'b11:   next_pc = jalr_target;      // JALR
            default: next_pc = PC + 4;
        endcase
    end
    
    // Sequential logic for PC register
    always @(posedge MainClock) begin
        if (reset)
            PC <= 32'b0;              // Reset to address 0
        else if (PCWrite)
            PC <= next_pc;            // Update PC when enabled
        // else: hold current value
    end

endmodule

`default_nettype wire
```

### BranchUnit.v
```verilog
`default_nettype none

module BranchUnit (
    input [31:0] PC,              // Current program counter
    input [31:0] rs1_data,        // Register rs1 value (for JALR)
    input [31:0] immediate,       // Immediate value from instruction
    
    output [31:0] branch_target,  // Branch target: PC + imm
    output [31:0] jump_target,    // JAL target: PC + imm
    output [31:0] jalr_target     // JALR target: (rs1 + imm) & ~1
);

    // Branch and JAL: PC + immediate (PC-relative addressing)
    assign branch_target = PC + immediate;
    assign jump_target   = PC + immediate;
    
    // JALR: (rs1 + immediate) with LSB cleared for alignment
    assign jalr_target = (rs1_data + immediate) & 32'hFFFFFFFE;

endmodule

`default_nettype wire
```

## Immediate Generator
### ImmediateGenerator.v
The immediate generator extracts and sign-extends immediate values from RISC-V instructions. RISC-V has 6 different instruction formats (R, I, S, B, U, J), and each one packs the immediate bits differently. This module handles all of them.

```verilog
`default_nettype none

module ImmediateGenerator (
    input [31:0] instruction,
    output reg [31:0] immediate
);

    wire [6:0] opcode;
    assign opcode = instruction[6:0];
    
    // Immediate extraction is purely combinational
    always @(*) begin
        case (opcode)
            // I-type: ADDI, SLTI, SLTIU, XORI, ORI, ANDI, SLLI, SRLI, SRAI
            // Also: Load instructions (LB, LH, LW, LBU, LHU) and JALR
            7'b0010011, // I-type arithmetic
            7'b0000011, // Load instructions
            7'b1100111: // JALR
                begin
                    // Sign-extend 12-bit immediate
                    immediate = {{20{instruction[31]}}, instruction[31:20]};
                end
            
            // S-type: Store instructions (SB, SH, SW)
            7'b0100011:
                begin
                    // Sign-extend split immediate: [31:25] and [11:7]
                    immediate = {{20{instruction[31]}}, instruction[31:25], instruction[11:7]};
                end
            
            // B-type: Branch instructions (BEQ, BNE, BLT, BGE, BLTU, BGEU)
            7'b1100011:
                begin
                    // Sign-extend and reorder branch immediate
                    immediate = {{19{instruction[31]}}, 
                                instruction[31],      // imm[12]
                                instruction[7],       // imm[11]
                                instruction[30:25],   // imm[10:5]
                                instruction[11:8],    // imm[4:1]
                                1'b0};                // imm[0] = 0 (always even)
                end
            
            // U-type: LUI, AUIPC
            7'b0110111, // LUI
            7'b0010111: // AUIPC
                begin
                    // Upper 20 bits, lower 12 bits = 0
                    immediate = {instruction[31:12], 12'b0};
                end
            
            // J-type: JAL
            7'b1101111:
                begin
                    // Sign-extend and reorder jump immediate
                    immediate = {{11{instruction[31]}},
                                instruction[31],      // imm[20]
                                instruction[19:12],   // imm[19:12]
                                instruction[20],      // imm[11]
                                instruction[30:21],   // imm[10:1]
                                1'b0};                // imm[0] = 0 (always even)
                end
            
            // R-type and unknown: no immediate
            default:
                begin
                    immediate = 32'b0;
                end
        endcase
    end

endmodule

`default_nettype wire
```

### Why Immediates Are Weird in RISC-V
RISC-V packs immediate bits in different positions depending on instruction type to keep the register fields (rs1, rs2, rd) in consistent locations across all formats. This makes decoding hardware simpler even though it makes the immediate extraction look complicated.

For example:
- **I-type**: immediate in bits [31:20] - straightforward
- **S-type**: split between [31:25] and [11:7] - need to concatenate
- **B-type**: scrambled and shifted - bits out of order, LSB is always 0
- **J-type**: even more scrambled - 20-bit offset for bigger jumps

The `{{20{instruction[31]}}, ...}` syntax replicates the sign bit 20 times for sign extension.

## Controller and ROM  
*TODO:*  
Updated FSM Diagram (mermaid)  
Control Unit FSM (currently has R-type working, need to expand for I, S, B, U, J types)  
Updated ROM table  
**after this point we have essentially built what we need.**

## Module Progress Tracker
✅ **Completed:**
- ALU (all base operations + SLTU)
- Register File (32 x 32-bit registers)
- Program Counter (with branch/jump support)
- Branch Unit (target calculation)
- Immediate Generator (all 6 instruction formats)

□ **In Progress:**
- Control Unit FSM (R-type done, expanding to other types)

□ **To Do:**
- Instruction Memory (ROM)
- Data Memory (for loads/stores)
- Top-level integration

## Final test, fibonacci sequence, referencing previous registers and using jumping for a loop.
