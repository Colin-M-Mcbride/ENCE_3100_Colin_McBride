# Lab 7- UART Word Detector

In this lab, the DE10 Lite implements a UART TX/RX loop that displays received characters on seven-segment displays. When the word "HELLO" is detected, the system displays "HELLo" for three seconds before reverting to normal operation.

## Part 1- Predefined Circuitry

This portion of the code was provided as part of the lab framework. The `async_receiver` and `async_transmitter` modules handle the serial communication protocol at 115200 baud over GPIO pins 33 and 35 respectively. These modules are intentionally abstracted away as they implement the bit-level timing and framing required for UART communication.

### UART Communication Flow

```verilog
wire w_clk = MAX10_CLK1_50;

wire RxD_data_ready;
wire [7:0] RxD_data;
reg [7:0] GPout;
wire [7:0] seg_data;

async_receiver RX(
    .clk(w_clk),
    .RxD(GPIO[35]),
    .RxD_data_ready(RxD_data_ready),
    .RxD_data(RxD_data)
);

always @(posedge w_clk)
    if(RxD_data_ready)
        GPout <= RxD_data;

async_transmitter TX(
    .clk(w_clk),
    .TxD(GPIO[33]),
    .TxD_start(RxD_data_ready),
    .TxD_data(RxD_data)
);

char2seg Display(
    .char(GPout),
    .HEX0(seg_data)
);
```

The 50MHz clock (`MAX10_CLK1_50`) is distributed to all synchronous modules via `w_clk`, ensuring all state machines and counters operate on the same time base. The `async_receiver` monitors GPIO pin 35 for incoming serial data. When a complete byte is received (start bit, 8 data bits, stop bit), `RxD_data_ready` pulses high for exactly one clock cycle. This single-cycle pulse serves three purposes: it latches the received byte into the `GPout` register, triggers the transmitter to echo the data back on GPIO pin 33, and signals the FSM that new data is available. The `char2seg` module continuously converts the ASCII value in `GPout` to seven-segment display encoding in `seg_data`. This conversion is purely combinational, so the display updates immediately when `GPout` changes.

## Part 2- New Circuitry (FSM and Counter)

The custom logic consists of two primary components: a finite state machine for pattern matching and a counter for timing control.

### Hello Detection (FSM)

The word detection problem requires sequential pattern matching - we must recognize five specific characters in order. A finite state machine is the natural solution as it can track progress through the word while handling invalid input gracefully by resetting to the start state.

#### Module Interface and State Encoding

```verilog
module FSM_Word_Detector(
    input rst,
    input clk,
    input [7:0] RxD_data,
    input Rx_detect,
    input [7:0] seg_data,
    input counter_done,
    output reg counter_start,
    output reg [7:0] HEX0,
    output reg [7:0] HEX1,
    output reg [7:0] HEX2,
    output reg [7:0] HEX3,
    output reg [7:0] HEX4,
    output reg [7:0] HEX5
);

reg [2:0] state, next_state, prev_state;

localparam [2:0] A = 3'd0, H = 3'd1, E = 3'd2, L = 3'd3, L2 = 3'd4, O = 3'd5;

always @(posedge clk) begin
    if(rst)
        state <= A;
    else begin
        prev_state <= state;
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
//next state logic
always @(negedge Rx_detect, negedge counter_done) begin
    case(state)
        A: next_state <= (seg_data == 8'b01001001/*H*/) ? H : A;
        H: next_state <= (seg_data == 8'b01000110/*E*/) ? E : (seg_data == 8'b01001001/*H*/) ? H : A;
        E: next_state <= (seg_data == 8'b01100111/*L*/) ? L : (seg_data == 8'b01001001/*H*/) ? H : A;
        L: next_state <= (seg_data == 8'b01100111/*L*/) && prev_state == L) ? L2 :
                         (seg_data == 8'b01100111/*L*/) ? L : A;
        L2: next_state <= (seg_data == 8'b01010011/*O*/) ? O : (seg_data == 8'b01100111/*L*/) ? L2 : A;
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
always @(*) begin
    case(state)
        A: begin
            HEX0 = seg_data;
            HEX1 = 8'hFF;  // Blank remaining displays
            HEX2 = 8'hFF;
            HEX3 = 8'hFF;
            HEX4 = 8'hFF;
            HEX5 = 8'hFF;
            counter_start = 1'b0;
        end
        // States H, E, L, L2 identical to state A
        O: begin
            HEX0 = 8'h77;              // 'H'
            HEX5 = 8'b01000001;        // 'o' (lowercase)
            HEX4 = 8'b01000110;        // 'E'
            HEX3 = 8'b01000011;        // 'L'
            HEX2 = 8'b01000011;        // 'L'
            HEX1 = 8'b01000011;        // 'L' (error: should be 'o')
            counter_start = 1'b1;
            if(counter_done)
                counter_start = 1'b0;
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
    parameter CLK_SPEED = 50_000_000,
    parameter WIDTH = 26
)
(
    input clk,
    input rst,
    input en,
    output reg [1:0] count,
    output reg counter_done
);

reg [WIDTH-1:0] clk_count;

always @(posedge clk) begin
    if(rst) begin
        clk_count <= {WIDTH{1'b0}};
        count <= 2'd0;
        counter_done <= 1'b0;
    end
    else if(en) begin
        clk_count <= clk_count + 1;
        if(clk_count == CLK_SPEED - 1) begin
            clk_count <= {WIDTH{1'b0}};
            count <= count + 1;
        end
        if(count == 2'd3) begin
            counter_done <= 1'b1;
            count <= 2'd0;
            clk_count <= {WIDTH{1'b0}};
        end
        else
            counter_done <= 1'b0;
    end
    else
        counter_done <= 1'b0;
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
