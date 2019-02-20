
## Dependence Graph

<p align="center">
  <img height="85%" width="85%" src="../images/graph_example.png">
</p>


A dependence graph model is an abstraction model of a processor,
which describes the execution behavior of a workload on the processor as a graph.
The above figure shows an example that models a 2-way superscalar out-of-order processor executing five instructions.
A column of nodes represents the pipeline stages of the processor where a single instruction goes.
The first column of this example models the first instruction, _i1_.
The edge connecting two nodes describes the dependency of pipeline stages
(e.g., the sequence of pipeline stages, inorder-fetch, data dependencies),
and its weight indicates when the next stage can proceed.
As a result, the longest path from the first node (Fetch node of the first instruction) to the last node (Commit node of the last instruction),
or critical path, determines the execution time of the workload running on the modeled processor.

### Dependence Graph for Out-of-order Processor
We currently provide a graph model for an out-of-order processor.
As listed in the following figures, our graph model has a 13-stage (logical) pipeline.

<p align="center">
  <img height="80%" width="80%" src="../images/graph_model.png">
</p>

Three stages in red color (Memory Ready, Address Generation, DTLB) are for memory-type instructions.
We leave the description of nodes and edges in the below.
For more information, check our paper [MICRO2014] and the source files "graph.[cc,hh]."

#### Node descriptions
* F, start of instruction fetch
* ITLB, ITLB access done
* I$, I-Cache access done
* N, register renaming and re-order buffer entry allocation
* D, issue queue entry allocation
* AR1, all data operands for address calculation ready, except address calculation unit
* AR2, address calculation
* DTLB, DTLB access done
* R, all data operands ready, except functional unit
* E, execution
* P, execution complete
* RC, ready to commit
* C, commit.

#### Edge descriptions

| Constraint | Edge | Description |
| ---------------------- |---------- | ------------------|
| In-order fetch | I$<sub>i-1</sub>&rarr;F<sub>i</sub> | |
| Finite fetch bandwidth | I$<sub>i-fbw</sub>&rarr;F<sub>i</sub> | where _fbw_ is the maximum number of instructions that can be fetched in a cycle |
| Finite fetch buffer size | N<sub>i-fbs</sub>&rarr;F<sub>i</sub> | where _fbs_ is the fetch buffer size |
| Control dependency | I<sub>i-1</sub>&rarr;F<sub>i</sub> | inserted if instruction _i_ - 1 is mispredicted branch |
| ITLB access latency | F<sub>i</sub>&rarr;ITLB<sub>i</sub> | is 0 in case of ITLB hit |
| I$ access latency | ITLB<sub>i</sub>&rarr;I$<sub>i</sub> | is 0 in case of I$ hit 
| Rename after I$ access | I$<sub>i</sub>&rarr;N<sub>i</sub> | 
| In-order rename | N<sub>i-1</sub>&rarr;N<sub>i</sub> |
| Finite reorder buffer| C<sub>i-rbs</sub>&rarr;N<sub>i</sub> | where _rbs_ is the re-order buffer size 
| Finite rename bandwidth | N<sub>i-nbw</sub>&rarr;N<sub>i</sub> | where _nbw_ is the maximum number of instruction that can be processed at the rename stage in a cycle
| Dispatch after rename | N<sub>i</sub>&rarr;D<sub>i</sub> | 
| In-order dispatch | D<sub>i-1</sub>&rarr;D<sub>i</sub> |
| Issue dependency | E<sub>j</sub>&rarr;D<sub>i</sub> | prefer to select instruction _j_ waiting for the result of an optimizable long-latency instruction (modeling the issue dynamics) |
| Finite dispatch width| D<sub>i-dbw</sub>&rarr;D<sub>i</sub> | where _dbw_ is the maximum number of instructions which can be dispatched in a cycle 
| Ready after dispatch | D<sub>i</sub>&rarr;AR1<sub>i</sub> | 
| Data dependency | P<sub>j</sub>&rarr;AR1<sub>i</sub> | inserted if a ld/st instruction _i_ depends on previous instruction  _j_'s result for address calculation
| Address calculation | AR1<sub>i</sub>&rarr;AR2<sub>i</sub> |
| DTLB access latency | AR2<sub>i</sub>&rarr;DTLB<sub>i</sub> |
| Ready after dispatch | D<sub>i</sub>&rarr;R<sub>i</sub> | 
| Finite physical registers | C<sub>j</sub>&rarr;R<sub>i</sub> | inserted if instruction _j_ releases the physical register instruction _i_ will use
| Data dependency | P<sub>j</sub>&rarr;<sub>R</sub><sub>i</sub>| inserted if instruction _i_ depends on previous instruction _j_'s result
| Ready after DTLB access | DTLB<sub>i</sub>&rarr;R<sub>i</sub> | inserted if the instruction _i_ is load or store
| Execute after ready | R<sub>i</sub>&rarr;E<sub>i</sub> | 
| Address dependency | E<sub>j</sub>&rarr;E<sub>i</sub>| every load _i_ can be executed after all stores _j_  $<$  _i_ be executed or at the same time 
| Completion after execute | E<sub>i</sub>&rarr;P<sub>i</sub> |
| Cache line sharing | P<sub>j</sub>&rarr;P<sub>i</sub> | inserted if load _i_ and load _j_ has the same address 
| In-order commit | C<sub>i-1</sub>&rarr;RC<sub>i</sub>|
| Finite commit width | C<sub>i-cbw</sub>&rarr;RC<sub>i</sub> |
| uop dependecy | P<sub>j</sub>&rarr;RC<sub>som</sub>| inserted for all instruction _j_ $>=$ p _som_ where _som_ is start of macro op instruction
| Commit latency | RC<sub>i</sub>&rarr;<sub>C</sub><sub>i</sub>	   | |


