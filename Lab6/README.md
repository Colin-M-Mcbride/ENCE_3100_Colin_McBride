## Lab 6: Chess clocks and Finite State Machines
### Writen by Colin McBride

*Explaination of ideas*
     
*ascii art of block diagram*  
### State Machine Diagram
```
                                    ┌──────────────┐
                            ┌───────│     IDLE     │◄──────┐
                            │       │  (Initial)   │       │
                            │       └──────┬───────┘       │
                            │              │               │
                            │              │ Start Button  │
                            │              │ Pressed       │
                            │              v               │
                            │         ┌────────┐           │
                            │         │ SW1=?  │           │
                            │         └───┬─┬──┘           │
                            │             │ │              │
                            │      SW1=0  │ │  SW1=1       │
                            │             │ │              │
                            │             v v              │
                            │    ┌─────────┴─────────┐     │ 
                            │    │                   │     │
                      Reset │    │                   │     │ Reset
                            │    │                   │     │
                            │    v                   v     │
                      ┌─────┴────────┐         ┌───────────┴──┐
                   ┌──│  COUNTER1    │         │  COUNTER2    │──┐
                   │  │   ACTIVE     │         │   ACTIVE     │  │
                   │  └─────┬────────┘         └─────┬────────┘  │
                   │        │                        │           │
                   │        │ Start Button           │ Start     │
                   │        │ Pressed                │ Button    │
                   │        │                        │ Pressed   │
                   │        v                        v           │
                   │  ┌──────────┐             ┌──────────┐      │
                   │  │  PAUSE1  │             │  PAUSE2  │      │
                   │  └─────┬────┘             └─────┬────┘      │
                   │        │                        │           │
                   │        │ Start Button           │ Start     │
                   │        │ Pressed                │ Button    │
                   │        │                        │ Pressed   │
                   │        │                        │           │
                   │        └────────┬───────────────┘           │
                   │                 │                           │
                   │                 v                           │
                   │          ┌─────────────┐                    │
                   │          │   Check     │                    │
                   │          │    SW1      │                    │
                   │          └─────┬───┬───┘                    │
                   │                │   │                        │
                   │         SW1=0  │   │  SW1=1                 │
                   │                │   │                        │
                   └────────────────┘   └────────────────────────┘
```

### State Descriptions

- **IDLE**: Initial/idle state waiting for start button
- **COUNTER1 ACTIVE**: Counter1 is running (when SW1=0)
- **COUNTER2 ACTIVE**: Counter2 is running (when SW1=1)
- **PAUSE1/PAUSE2**: Paused states for each counter
- **Check SW1**: Decision point that checks SW1 to determine which counter to resume
- Pressing start button while in COUNTER1/2 → enters PAUSE
- Pressing start button while in PAUSE → checks SW1 and loops back to appropriate counter
- Reset signal returns system to IDLE from any state

### Transitions

| From State | Trigger | To State |
|------------|---------|----------|
| IDLE | Start Button + SW1=0 | COUNTER1 ACTIVE |
| IDLE | Start Button + SW1=1 | COUNTER2 ACTIVE |
| COUNTER1 ACTIVE | Start Button | PAUSE1 |
| COUNTER2 ACTIVE | Start Button | PAUSE2 |
| PAUSE1 | Start Button | Check SW1 |
| PAUSE2 | Start Button | Check SW1 |
| Check SW1 | SW1=0 | COUNTER1 ACTIVE |
| Check SW1 | SW1=1 | COUNTER2 ACTIVE |
| Any State | Reset | IDLE |

### Behavior Notes

When the start button is pressed from either PAUSE state, the system checks SW1 and returns to:
- **COUNTER1 ACTIVE** if SW1=0
- **COUNTER2 ACTIVE** if SW1=1

This allows the user to switch between counters by toggling SW1 before pressing the start button to resume.
*gif of functional clock*  
**^^^MANDATORY^^^**  
**Below is for the love of the game**  
*pt2 upgrades*  
*ascii art of new block diagram*  
*gif of pt clock*
