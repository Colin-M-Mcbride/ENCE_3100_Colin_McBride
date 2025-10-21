## Lab 6: Chess clocks and Finite State Machines
### Writen by Colin McBride

*Explaination of ideas*
     
*ascii art of block diagram*  
# State Machine Diagram
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
                            │    ┌─────────┴─────────┐    │
                            │    │                   │    │
                      Reset │    │                   │    │ Reset
                            │    │                   │    │
                            │    v                   v    │
                      ┌─────┴────────┐         ┌─────────┴────┐
                      │  COUNTER1    │         │  COUNTER2    │
                      │   ACTIVE     │         │   ACTIVE     │
                      └─────┬────────┘         └─────┬────────┘
                            │                        │
                            │ Start Button           │ Start Button
                            │ Pressed                │ Pressed
                            │                        │
                            v                        v
                      ┌──────────┐             ┌──────────┐
                      │  PAUSE1  │             │  PAUSE2  │
                      └─────┬────┘             └─────┬────┘
                            │                        │
                            │ Start Button           │ Start Button
                            │ Pressed                │ Pressed
                            │                        │
                            └────────┬───────────────┘
                                     │
                                     v
                              ┌─────────────┐
                              │   Resume    │
                              │  Counter    │
                              └─────┬───────┘
                                    │
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
                    v                               v
          Back to COUNTER1                 Back to COUNTER2
```

## State Descriptions

- **IDLE**: Initial/idle state waiting for start button
- **COUNTER1 ACTIVE**: Counter1 is running (when SW1=0)
- **COUNTER2 ACTIVE**: Counter2 is running (when SW1=1)
- **PAUSE1/PAUSE2**: Paused states for each counter
- Pressing start button while in COUNTER1/2 → enters PAUSE
- Pressing start button while in PAUSE → resumes the respective counter
- Reset signal returns system to IDLE from any state

## Transitions

| From State | Trigger | To State |
|------------|---------|----------|
| IDLE | Start Button + SW1=0 | COUNTER1 ACTIVE |
| IDLE | Start Button + SW1=1 | COUNTER2 ACTIVE |
| COUNTER1 ACTIVE | Start Button | PAUSE1 |
| COUNTER2 ACTIVE | Start Button | PAUSE2 |
| PAUSE1 | Start Button | COUNTER1 ACTIVE |
| PAUSE2 | Start Button | COUNTER2 ACTIVE |
| Any State | Reset | IDLE |
*gif of functional clock*  
**^^^MANDATORY^^^**  
**Below is for the love of the game**  
*pt2 upgrades*  
*ascii art of new block diagram*  
*gif of pt clock*
