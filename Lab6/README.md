## Lab 6: Chess clocks and Finite State Machines
### Writen by Colin McBride

*Explaination of ideas*
     
*ascii art of block diagram*  
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
          Back to COUNTER1                 Back to COUNTER2*pt 1 code (start/pause, player switch, counters)*  
*gif of functional clock*  
**^^^MANDATORY^^^**  
**Below is for the love of the game**  
*pt2 upgrades*  
*ascii art of new block diagram*  
*gif of pt clock*
