# DC notes and questions
1. Why should we learn Computer Architecture? Is it important when we are designing digital circuit?
> Not neccessary. Since it is just a basic knowledge you need to have, cpu is easiest architecture you can think of when implementing asic or vlsi architecture. So if you can handle the DSP VLSI architecture course well, you can actually bypass computer architecture courses, since its main focus is on cpu design. Architectural conecpt like systolic arrays, Advanced pipeline architecture and Filter architecture matters more.
2. Does industry seperate Controlpath and datapath completely?
> Yes. Moreover, basic design even further partition the controlpath into two parts, one for functional operations generation, another one for low-power control. So we will definitely seperate these two out.
3. Will you use multi-level FSM when implementing Controlpath ckt?
> Usualy not. Because speed matters, so we would want the decoding to be as fast as possible, i.e. making the logic as simple as possible.
4. Should we seperate Sequential logic and Combinational logic out when coding verilog?
> Not neccesary. However, seperating them out can make debug life easier. Thus we only do so when we want to trace a certain value propogating to another register. As a result, we do it only when we want to debug the circuit.
5. Is ternary operator faster than if~else?
> No. They are the same thing and will be synthesized into the same circuit.
6. Is Clock gating the FSM a viable option?
> Yes. If the state transition is stuck within a certain state for a long time, you can actually clock gated the FSM during that period.
7. Use Gray Counter and Gray Encoding for power reduction?
> Yes. It is possible, however the power reduction might not be significant, since clock gating and power gating dominates the power reduction steps.
8. How do you think about C++ HLS?
> I dont think it is going to replace verilog so soon, since writing HLS requires knowledge of vlsi architecture and whole algorithm view, all the way from C -> vlsi architecture. I dont think it is possible lately.
9. How is fpga used in industry? What do you mean by the design is verified on an FPGA? Should I consider efficiency on an FPGA?
> FPGA is used for FUNCTIONAL Verification only. We actually dont care about the efficieny running on fpga, we cares only about functional correctness. Main optimization part lies in ASIC-flow. FPGA is only used for verification.
10. How do you think about CHISEL HDL?
> I think for student, you had better look for the requirement posed by companies, first finish learning them then consider researching such topics. Unless you want to do research in academic, i dont suggest you learning it.


# PVT(Process,Voltage,temperature) MODELS
- In mature process technology, $T \propto 1/\epsilon$, $\epsilon$ means efficiency, however, for advanced process technology, $T \propto \epsilon$. Completely reversed.
- We define mature process technology as cell manufactured in the size that is larger than 40(nm). If the cell is smaller than 40(nm) we call it advanced process technology. FINFET
- The PVT model has 6 constriants to set
1. set_operating_conditions(SS,FF Worst and best environment)
2. set_driving_cell(The driving power of last stage beyond the design)
3. set_input_delay(The input delay of the last stage beyond the design)
4. set_load(The output load capacitance of the design)
5. set_output_delay(The output delay of another stage of the design)
6. set_wire_load_model(Inner wire load model)

# Output load model
- From the experience of the lecturer, the output load driving factor is usually set to the following value.

|Testing machine| PCB | from top to I/O pad |Other's design|
|       ---     |  ---|       ---           |           ---|
|  40           | 10  |     0.05            |  0.002       |

# FF(Best) TT(Typical) SS(Worst) conditions for cell
1. We usually choose two constraint environments for the cell because it covers a wider range than a single constriant. Usually pick ff and ss. I.e. the slow.db and fast.db.
2. Using two min/max case statement in operating condition allows a wider working range.
3. Slow.db can be used as a setup time standard because we want setup time to be as fast as possible, using slow.db is a pessimistic estimation.
4. Fast.db used as a hold time standard with the same reason as above.
- If hold time cannot be met, use inverter chains, fast.db.

# Tape out tips about CHIP.v
1. If you want to tape out, you should synthesize your design with I/O pad around your top. Synthesize your design from the CHIP.v allows for easier synthesis constraint setting also it allows tape out. Synthesizing from top.v is also find however may cause some nusances.
2. When using ICC2 or Innovus, knowing what the numerical value the tool gives you is the most important tasks.
3. PDIDGZ is the TSMC Input port and PDO16CDG is the Output port for pad.
4. Below table shows the delay number of output ports, corresponding to different allowable frequency. To run at higher speed. Higher number port should be selected.

|2,4,8          | 12,16,24 |
|       ---     |  ---|
|  10 Mhz           | 100 Mhz  |
# WLM(Wire load model)
1. To search for wire-load model document. See the slow.lib, lots of info can actuualy be found within this file. Simply search wire load in it to find the wire load for different gate or design.

2. Synthesis tool does not have axis and position info of gates. So in dc, we cannot estimate the load of the wire easily. Thus dc usually sets it into a random value.
3. Usally, in mature process technology
<br /> $delay = Delay_{cell} + Delay_{net}$
<br /> Usually $Delay_{cell}$ dominates the whole delay, however, in advanced technology(smaller than 40nm), $Delay_{net}$ actually contributes to a large portion of delay. So the estimation becomes even harder and more inaccurate. How to solve it?

1. Layout principle. the main IP blocks should be places in the edges or corners to allow other cells to connect with each other without blocking them.

# SPG and Fusion Compiler
1. SPG, Perform layout and synthesis at the same time. To solve the inaccurate wire load model and placement problem. Makes design accurate cell model easier.
2. Fusion Compiler(2022/05) yields even better synthesis result, and is available for student, we have licenses. It perform (APR+DC) using Hier-Flow allowing a design originally requires a whole team, now only needs one single person to handle the whole flow.

# Constraints
## Timing
1. In timing we have to set 7 principal components for design. All of it must be set.
2. When specifying clk. Use dont touch network(Prevent the tool from adding buffers onto the clk path)
- Fix hold(Configures the hold time violation for you)

## CLK naming
1. In dc, you must always reconfigure the clk name to the name you want, since in design, you usually have multiple clks like CLK1,CLK2, and so on.
After renaming, the CLK1 is no longer pins, but a name of your clk.
2. ```created_generated_clock``` is important for you to generate different derived clks.
3. There is no way dc can know the other clk we derived from our main clk, e.g. a clock divider or multipler. Thus we have to tell the dc we have this component by ourself.
4. Constraint sdc must be complete since it would be used over and over again in dc and APR flow.

## CTS
1. Clock tree synthesis, fan out trees are usually a multiple of 2 in sizing for best driving power. The clock tree grows in a parrallel way to handle the whole clock tree network of the circuit.
2. Usually from lecturer experience, the time of clock skew is set as follows
. We can set clock skew using ```set_clock_uncertainty 0.1 [get_clocks CLK]```

|Tech|Your lab HW     | Thesis design  | Industrial level design |
|--- |       ---      |  ---           |       ---               |
|90nm|  0.1           | 0.2            |     0.3~0.5             |
|16nm|  0.03          | 0.067          |     0.1~0.17            |


3. When synthesizing, first use ideal_clk network. No delay exists during synthesis! Set it only during the APR phase.
> Skew can actually be good sometime, there exists good skew and bad skew. Good skew can be used to extend slack.

4. CCD mode(Concurrent clock and data optimization), combine the analysis of clock tree nad synthesis together which allows even faster design.

## Model source latency
1. When teamWorking, we have to ask project leader about the clock delays we cannot know in the modules that is worked by other people, or needs to ask for the clock delays after the clk rate is being double or decremented.

2. The clock buffer delays is usually the following

|Tech|Your lab HW     | Thesis design  | Industrial level design |
|--- |       ---      |  ---           |       ---               |
|90nm|  1             |  2             |     3                   |
|16nm|  0.3           | 0.6            |     1                   |

3. clock latency of source and unknown buffer delays must both be considered to ensure the clock working.

## Special circuit constraint
1. Multicycle path command can increase the clock cycle seen by dc. Also it can be used to eliminate false path.
2. In industry, since negedge trigger d_ff is not common. They usually inverted the clock and connect it into the negedge trigger d_ff you want.

## Multiple clock designs
1. If there are multiple clocks, we have to set 7 principles on all of them.
2. If you are trying to cross domains, one way is to find the gcd of the two clk domain you are tring to cross, then tell the dc that is the clock frequency you are trying to create.
3. Use Synchronizer, if only asynchronous circuit is permitted.

## Inter-Clock delay balance
1. Remember to -balance_group otherwise you will get horrendous clk delays out of no where.

## Elminating False Path
1. In industry standard, they usually write the following, where N is and arbitrarily selected clocks. It depends for your design.

```
    set_multicycle_path -setup N   -from {A} -through {C} to {OUT}
    set_multicycle_path -hold  N-1 -from {A} -through {C} to {OUT}
```

## Multiple clks per registers
1. DC cannot know only clk1 exists. It would think the clk just coexeists, thus giving a wrong analysis
``` -logically_exclusive```
2. Frequency selection for a certain block is allowed, but require certain commands to be set for DC.

# Power Reduction technique
1. Clock Gating can actually be handled automatically by dc if you give it the constraint. You dont have to generate the latch yourself at all.
2.
