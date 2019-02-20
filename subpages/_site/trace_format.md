## Detailed Timing Trace

RpStacks requires a detailed timing trace to build a processor dependence graph model.
The detailed timing trace records the exact pipeline timing of executed instructions (e.g., when an instruction is fetched)
and lots of information required to track the dependencies of pipeline events
(i.e., the register indices and a memory address each instruction reads and writes). 

Generating the detailed timing trace requires cycle-level simulators.
We disclose the trace format RpStacks uses.
The format design is primarily after a superscalar out-of-order core modeled in PTLsim,
but it is generic as much as possible to cover various out-of-order core models in other simulators with slight modifications. 


### Trace Format
