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
> I think for student, you had better look for the requirement posed by companies, first finish learning them then consider researching such topics. Unless you want to do research in academic, i dont suggest you learning it. For now systmeVerilog and Verilog also some C-programming and python should be enough for you.


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
1. If you want to tape out, you should synthesize your design with I/O pad around your top. Synthesize your design from the CHIP.v allows for easier synthesis constraint setting also it allows tape out. Synthesizing from top.v is also fine however may cause some nusances.
2. When using ICC2 or Innovus, knowing what the numerical value the tool gives you is the most important tasks.
3. PDIDGZ is the TSMC Input port and PDO16CDG is the Output port for pad.
4. Below table shows the delay number of output ports, corresponding to different allowable frequency. i.e.
```
        PDO16CDG, the 16 in it
```
- To run at higher speed.Higher number port should be selected.

|2,4,8          |             12,16,24 |
|       ---     |                   ---|
|  10 Mhz           | 100 Mhz or more  |
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

#  CCD mode in DC-NXT(DC 2.0)
- CCD mode(Concurrent clock and data optimization), combine the analysis of clock tree nad synthesis together which allows even faster design.

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
1. Clock Gating can actually be handled automatically by dc if you give it the constraint. You dont have to generate the latch yourself at all. Just give dc the command it will do it for you.
```verilog
    assign gclk = clk & Enable;
    always@(posedge gclk)
    begin
        if(reset)
        begin
            data <= 0;
        end
        else
        begin
            data <= data + 8'b1;
        end
    end
```
Then simply gives command to dc, it would auto serach all clk gating cells.
```
        replace_clock_gates
        compile
```
To serach for number of CG(CLK GATING) cells type
```
    report_clock_gating -gating_elements
```

## Gate level dynamic power optimization
0. In industry, advanced process technology is used, in such implementation, leakage power starts to dominates and becomes a serious problem, thus one of the low power technique is to replace some cells with worse or better cells, instead of using only 1 type of cell.
1. Actually there exists different cell libraries. Except the common standard one, dc is also smart enough to replace your design with faster or slower cells when needed.
- Usualy we have HVT,LVT,RVT cells (High $V_t$ cell) (Low $V_t$ cell) and regular one.
- HVT has lower leakage power but longer delays.
- LVT has larger leakage power but low delay.
2. These are generally not a problem in academic however serious problem in industry, and a great technique to reduce the power and improve the performance of your ckt.
3. It replaces the cell by saving power on Non-critical paths.

```
    compile

    set_leakage_optimization true
    compile -inc
```
- -inc means keep my design but further improve it without resynthesizing again.
- To search for leakage power of every cell, search in the slow.lib and search keyword cell leakage power.
- Leakage power can be reduced by an astonishing 75% with this method!!!!

```
        report_threshold_voltage_group
```
- To call the cells get replaced by HVT or LVT.

## Gate counts anaylsis
1. In industry or design are estimation. We usually use $um^2$ for units. However sometimes we need gate counts for our design as design metrics. How can we get it?
```
    sizeof_collection [get_cells -hier *]
```

# Important notes to prevent your circuit from failing
1. The constraint you give for timing, skew, uncertatinty, WLM, fan_out, input_delay etc.. MUST met the range of table index within the slow.lib. Otherwise, even if synthesis can be performed, the chip you tape out cannot work at all!!! So you had better first check if the constraint value you keyed is within the range. To serach for index, simply type index for the cell you want to serach. Usually they are within a certain value range.

# SDC runtime in DC-NXT
In large design, removing unnecessary constraints can save you LOTS OF TIME, the following command removes those unneeded constraints away and optimizes the timing for you. This works only in DC-NXT. The DC version2.
```
    check_timing -sdc_runtime
    source sdc_runtime.log.tcl
    compile_ultra
```

```
    set sdc_runtime_analysis_enable true
```

# Clock Margin
1. In industry, we usually gives our own circuit a 10% clk margin. i.e. even if I want my circuit to run at 100 Mhz, we would like it to be made at 110 Mhz s.t. things can be easier for the APR and layout tools to work. Used to preserve margin and gives flexibility in design.
<br /> dc tools actually automate this

```
 set_timing_derate -late 1.10 -cell_delay [get_cells -hier *] #Setup time 110%
 set_timing_derate -early 1.00 -cell_delay [get_cells -hier *] #Hold time 100% No need to change
```
Note this command also needed to be given during the APR flow to make clock margin preserved.
Flow as the following
- set_timing_derate -> place -> CTS -> Route -> Report_timing -> reset_timing_derate -> report_timing to see the difference.

# TeamWorking clock margin for outputs and inputs
1. If you are working with someone else, and you dont know how to distribute the delay between your inputs delay and other people's output delays. Usually picking a 50% clock margin for each person is a good idea. i.e. if you want clock delay to be 10 ns. Then 5 ns would be given to person A's output logic, another 5 would be your allowable input delay logic.


# APR constraints file
- Before going into APR flow, although APR and DC shares the same xdc file, the following constraints MUST BE REMOVED!!!
1. WLM constraints
2. DONT_TOUCH_NETWORK
3. FIX_HOLD
4. IDEAL_NETWORK
- If you dont remove these, the APR tool cannot optimize and place cell for you.
