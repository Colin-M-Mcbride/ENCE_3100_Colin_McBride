# Lab 7- UART Word Detector

In this lab, the DE10 Lite implements a UART TX/RX loop that displays received characters on seven-segment displays. When the word "HELLO" is detected, the system displays "HELLo" for three seconds before reverting to normal operation.

## UART Communication Infrastructure (Predefined)

This portion of the code was provided as part of the lab framework. The `async_receiver` and `async_transmitter` modules handle the serial communication protocol at 115200 baud over GPIO pins 33 and 35 respectively. These modules are intentionally abstracted away as they implement the bit-level timing and framing required for UART communication.

### Data Flow and Module Interconnections

```verilog
// 50MHz system clock distribution
wire w_clk = MAX10_CLK1_50;

// UART receiver signals
wire RxD_data_ready;  // Single-cycle pulse when byte received
wire [7:0] RxD_data;  // Received ASCII byte
reg [7:0] GPout;      // Latched received data
wire [7:0] seg_data;  // Seven-segment encoded output

// UART receiver on GPIO[35]
async_receiver RX(
    .clk(w_clk),
    .RxD(GPIO[35]),
    .RxD_data_ready(RxD_data_ready),
    .RxD_data(RxD_data)
);

// Latch received data on ready pulse
always @(posedge w_clk)
    if(RxD_data_ready)
        GPout <= RxD_data;

// Echo transmitter on GPIO[33]
// Transmit starts immediately when data ready
async_transmitter TX(
    .clk(w_clk),
    .TxD(GPIO[33]),
    .TxD_start(RxD_data_ready),
    .TxD_data(RxD_data)
);

// ASCII to seven-segment converter
char2seg Display(
    .char(GPout),
    .HEX0(seg_data)
);
```

The 50MHz clock (`MAX10_CLK1_50`) is distributed to all synchronous modules via `w_clk`, ensuring all state machines and counters operate on the same time base. The `async_receiver` monitors GPIO pin 35 for incoming serial data. When a complete byte is received (start bit, 8 data bits, stop bit), `RxD_data_ready` pulses high for exactly one clock cycle. This single-cycle pulse serves three purposes: it latches the received byte into the `GPout` register, triggers the transmitter to echo the data back on GPIO pin 33, and signals the FSM that new data is available. The `char2seg` module continuously converts the ASCII value in `GPout` to seven-segment display encoding in `seg_data`. This conversion is purely combinational, so the display updates immediately when `GPout` changes.

## Word Detection System (Custom Logic)

The custom logic consists of two primary components: a finite state machine for pattern matching and a counter for timing control.

### Finite State Machine - Sequential Pattern Matcher

The word detection problem requires sequential pattern matching - we must recognize five specific characters in order. A finite state machine is the natural solution as it can track progress through the word while handling invalid input gracefully by resetting to the start state.

#### State Definition and Synchronous Update Logic

```verilog
module FSM_Word_Detector(
    input rst,
    input clk,
    input [7:0] RxD_data,      // Raw ASCII byte from UART
    input Rx_detect,            // Pulse when new data available
    input [7:0] seg_data,       // Seven-segment encoded character
    input counter_done,         // Feedback from 3s counter
    output reg counter_start,   // Enable signal for counter
    output reg [7:0] HEX0,     // Seven-segment display outputs
    output reg [7:0] HEX1,
    output reg [7:0] HEX2,
    output reg [7:0] HEX3,
    output reg [7:0] HEX4,
    output reg [7:0] HEX5
);

// State storage registers
reg [2:0] state, next_state, prev_state;

// State definitions for "HELLO" detection
localparam [2:0] A = 3'd0,   // Initial/Reset state
                 H = 3'd1,   // Detected 'H'
                 E = 3'd2,   // Detected "HE"
                 L = 3'd3,   // Detected "HEL" (first L)
                 L2 = 3'd4,  // Detected "HELL" (second L)
                 O = 3'd5;   // Complete "HELLO"

// Synchronous state update
// prev_state tracks where we came from (needed for L vs L2 logic)
always @(posedge clk) begin
    if(rst)
        state <= A;
    else begin
        prev_state <= state;  // Store current before updating
        state <= next_state;
    end
end
```

The FSM declares six states corresponding to detection progress:

| State | ASCII | Hex  | Meaning |
|-------|-------|------|---------|
| A     | -     | 3'd0 | Initial/Reset |
| H     | 72    | 3'd1 | Detected 'H' |
| E     | 69    | 3'd2 | Detected "HE" |
| L     | 76    | 3'd3 | Detected "HEL" |
| L2    | 76    | 3'd4 | Detected "HELL" |
| O     | 79    | 3'd5 | Complete "HELLO" |

Three 3-bit registers manage state: `state` holds the current state, `next_state` is computed by combinational logic, and `prev_state` stores the previous state value. The `prev_state` register is essential for distinguishing between the first and second 'L' in "HELLO" - without it, we couldn't verify that L2 was reached from state L.

#### Next State Logic

```verilog
// Next state logic - evaluates on falling edge of control signals
// negedge prevents multiple evaluations during single data reception
always @(negedge Rx_detect, negedge counter_done) begin
    case(state)
        // State A: Wait for 'H' to start sequence
        A: next_state <= (seg_data == 8'b01001001/*H*/) ? H : A;
        
        // State H: Got 'H', expect 'E' next
        // If 'H' again, stay in H (handles "HHHELLO")
        H: next_state <= (seg_data == 8'b01000110/*E*/) ? E : 
                         (seg_data == 8'b01001001/*H*/) ? H : A;
        
        // State E: Got "HE", expect first 'L'
        // If 'H' received, restart from H (handles "HEHELLO")
        E: next_state <= (seg_data == 8'b01100111/*L*/) ? L : 
                         (seg_data == 8'b01001001/*H*/) ? H : A;
        
        // State L: Got "HEL", expect second 'L'
        // Check prev_state to ensure we came from L not elsewhere
        // TODO: Second condition appears redundant
        L: next_state <= (seg_data == 8'b01100111/*L*/) && prev_state == L) ? L2 :
                         (seg_data == 8'b01100111/*L*/) ? L : A;
        
        // State L2: Got "HELL", expect 'O' to complete
        // If 'L' again, stay in L2 (handles "HELLLLO")
        L2: next_state <= (seg_data == 8'b01010011/*O*/) ? O : 
                          (seg_data == 8'b01100111/*L*/) ? L2 : A;
        
        // State O: Complete word detected, wait for counter
        O: next_state <= counter_done ? A : O;
        
        default: next_state <= A;
    endcase
end
```

The next state logic uses a combinational case statement sensitive to the negative edge of both `Rx_detect` and `counter_done`. Using `negedge` prevents the FSM from evaluating multiple times during a single data reception - when `Rx_detect` pulses high for one clock cycle, the FSM evaluates on the falling edge for clean state transitions.

The logic compares `seg_data` (seven-segment encoded characters) rather than raw ASCII. While unconventional, this works because `char2seg` provides deterministic mapping. Each state checks for its expected character and either advances, maintains position, or resets. States H and E include fallback logic that jumps back to state H if 'H' is received, handling cases like "HEHELLO" where the pattern restarts mid-sequence. State L2 includes self-loop logic for repeated 'L' characters. State O waits for `counter_done` before returning to state A.

**TODO:** Verify state L logic - the condition `(seg_data == 8'b01100111/*L*/) ? L : A` appears redundant since we can only reach state L from state E with an 'L' character.

**TODO:** Consider comparing against ASCII values in `RxD_data` instead of seven-segment encodings for clarity: `A: next_state <= (RxD_data == 8'h48) ? H : A;`

#### Output Logic

```verilog
// Output logic - combinational, determines display and control signals
always @(*) begin
    case(state)
        // States A, H, E, L, L2: Normal operation
        // Display current character on HEX0, blank others
        A: begin
            HEX0 = seg_data;     // Show current character
            HEX1 = 8'hFF;        // 8'hFF = all segments off (blank)
            HEX2 = 8'hFF;
            HEX3 = 8'hFF;
            HEX4 = 8'hFF;
            HEX5 = 8'hFF;
            counter_start = 1'b0;  // Counter disabled
        end
        // States H, E, L, L2 identical to state A
        
        // State O: "HELLO" detected, display word for 3 seconds
        O: begin
            HEX0 = 8'h77;              // 'H' = 0111 0111
            HEX5 = 8'b01000001;        // 'o' lowercase
            HEX4 = 8'b01000110;        // 'E' = 0100 0110
            HEX3 = 8'b01000011;        // 'L' = 0100 0011
            HEX2 = 8'b01000011;        // 'L'
            HEX1 = 8'b01000011;        // 'L' (TODO: should be 'o')
            counter_start = 1'b1;      // Start 3-second counter
            if(counter_done)
                counter_start = 1'b0;  // Clear when done
        end
    endcase
end
```

During states A through L2, the FSM displays the current character on HEX0 via `seg_data` and blanks remaining displays with `8'hFF` (all segments off). The `counter_start` signal remains low. When state O is reached, the FSM displays "HELLo" across the six displays and asserts `counter_start` high to begin the 3-second countdown. When `counter_done` goes high, it immediately clears `counter_start`, creating a feedback path that allows the FSM to detect timeout and transition back to state A.

**TODO:** Verify HEX1 assignment in state O - code shows 'L' pattern but specification requires lowercase 'o'. Physical display orientation on DE10-Lite should be verified.

### Counter Logic

The `counter_3s` module generates a precise 3-second delay using the FPGA's 50MHz clock through a dual-stage counter: a clock divider stage and a second counter stage.

```verilog
module counter_3s
#(
    parameter CLK_SPEED = 50_000_000,  // 50MHz clock frequency
    parameter WIDTH = 26                // Bits needed: 2^26 = 67M > 50M
)
(
    input clk,
    input rst,
    input en,                           // Enable from FSM counter_start
    output reg [1:0] count,            // Counts seconds: 0, 1, 2, 3
    output reg counter_done            // Pulses high when count reaches 3
);

reg [WIDTH-1:0] clk_count;  // Clock cycle counter for frequency division

always @(posedge clk) begin
    if(rst) begin
        // Reset all counters and flags
        clk_count <= {WIDTH{1'b0}};  // Replication: 26'b0
        count <= 2'd0;
        counter_done <= 1'b0;
    end
    else if(en) begin
        // Increment clock cycle counter
        clk_count <= clk_count + 1;
        
        // One second elapsed: 50M cycles counted
        // Use CLK_SPEED-1 because we count from 0
        if(clk_count == CLK_SPEED - 1) begin
            clk_count <= {WIDTH{1'b0}};  // Reset cycle counter
            count <= count + 1;           // Increment second counter
        end
        
        // Three seconds complete: count went 0->1->2->3
        if(count == 2'd3) begin
            counter_done <= 1'b1;         // Signal completion
            count <= 2'd0;                // Auto-reset for next use
            clk_count <= {WIDTH{1'b0}};
        end
        else
            counter_done <= 1'b0;
    end
    else
        counter_done <= 1'b0;  // Clear done when disabled
end
endmodule
```

#### Timing Calculations

At 50MHz, one clock cycle is 20ns. To generate 1-second intervals: `1 second / 20ns = 50,000,000 cycles`. Therefore `CLK_SPEED = 50_000_000`. The bit width calculation: `2^25 = 33,554,432` (insufficient), `2^26 = 67,108,864` (sufficient). The parameter `WIDTH = 26` provides adequate storage with 34% margin, which is good design practice for parameter flexibility.

#### Operation

The counter uses two registers: `clk_count [25:0]` counts clock cycles for frequency division, and `count [1:0]` counts seconds (0-3). When enabled, `clk_count` increments every cycle. At `CLK_SPEED - 1` (49,999,999), it resets and increments `count`. The comparison uses `CLK_SPEED - 1` because counting 0→49,999,999 gives exactly 50,000,000 cycles. When `count` reaches 3 (after 0→1→2→3, three complete seconds), `counter_done` asserts high and both counters reset. When `en` is low, `counter_done` clears but counter values retain their state.

**TODO:** Consider separating clock counting and second counting into separate always blocks for better timing closure in fast designs.

## Testing and Verification

Connect the DE10-Lite via USB and configure a serial terminal to 115200 baud, 8 data bits, no parity, 1 stop bit. Type characters to verify echo and display. Type "HELLO" to verify 3-second display period. Test partial matches like "HELO" followed by "HELLO" to verify reset behavior.
