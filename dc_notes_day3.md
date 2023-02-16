# DC note and question
1. When should I use memory compiler? When should I use RAM? Register File? D_ff
> If you want to read multiple datas out at a same time, use d_ff arrays. Whenever you have more than 512*8 d_ff content, use RAM. If you insist on using d_ffs in this case, terrible error can occur during APR phase, since there are simply no resourses and wire space available for routing due to the immense amount of d_ffs,there is going to be lots of red wires appeared on your APR result. For reg_file is is a type of memory which support posedege read and negedge write.Just another type of memory. So if the data you want to store is too large, USE MEMORY! If you are sure that the design is only going to read 2 ports or write less port at once, use MEMORY!

2. Can Fusion Compiler be used on EDA cloud?
> Yes. If you cannot find it, just tell the tsri to upload it onto the cloud. Also the documentation can be found on SYNOPSYS solveNet.

# Compile
1. First perform low-level Compile and make sure everything works to make the design progress fast.
2. Compile high may sometimes unable to converges.
3. Before compiling, false path problem must be resolved, otherwise compilation process cannot converges.
4. compile_ultra may sometimes leads to functional errors.
## compile_ultra
1. compile_ultra can only works in GTECH level, it cannot work at gate-level. So your synthesized .ddc file might not be able to benefit from it.
2. These command should not be used together.
```
    compile_ultra -t (timing best) 20% timing improvement
    compile_ultra -a (area best)  40% area improvement
```
3. Suggest dont do compile -gate_clock, gate level sim will have a high chance of failing.

4. When using compile_ultra, dont use -no_autoungroup command, suggest reread the design, then use compile ultra.

5. We can retime the circuit using d_ff and compile_ultra
- This helps resolve timing violation auto performing Retiming techniques to your circuit.
```
    compile_ultra -retime
```
- or you can use the following command
```
    optimize_registers
```
6. We only would like to retime on a smaller bounded cell, not the whole design. If you try to retime your WHOLE circuit, the VLSI architecture of your circuit might get destroyed. So we usually do retiming locally.

## Boundary Optimization
1. Removing unused or unconnected ports
2. When calling designWare IPs it may ask you to plug 1'b0,1'b1 to select for functionality, this would actually waste circuit area, so you have to tell compiler to not synthesize the unused part.
```
    compile -boundary_optimization
```
compile increment can be used if you want to further improve your design but dont want to completely repeat the synthesis process again.
```
    compile -inc
```

# Optimization report
## WNS(Worst negative Slack)
Note in layout tools, +WNS means overdesign, -WNS means lack of slack
## TNS(Total steup cost)
${TNS}_i = \Sigma_i {WNS}_i$ for the input D port of the worst D_ff cell.

# Power Optimization step
- For the synthesis tool to perform power optimization, dc must know the switching factor $\alpha$ by analyzing the waveform i.e. the vcd file. The steps are as follows.
1. compile your .v file
2. generate simulation file .vcd
3. convert your .vcd into .saif file
4. report power
5. compile -inc
6. report power again to see the difference
- Dramatic power reduction can be made using this method.

```
    read_saif -input design.saif -instance_name test/CHIP
```
This step must be added for the dc the analyze power of your circuit.

Note the power analysis given by dc is already pretty close to reality. The power report of the primetime is not trustworthy, by lecturer experience.

# Hold time MDC(Minimum delay cost)
- If the minimum delay cost of your circuit is something like -332, such a huge negative number, your constraint file is definitely wrong.

# Hold time fix in dc
1. hold time should be fixed @ synthesis instead of APR, otherwise in some cases APR tool might not be able to fix hold time violation, due to the insufficient area available for tool to insert buffers onto the clk tree. Leads to hold time failure.
2. Fixing hold time earlier enable us to spot errors earlier.
3. Gate level sim will fail if hold time problem isnt fixed.
```
    set_fix_hold [get_clock CLK]
```

4. If you want to perform DFT(Design for test) on your circuit, run this
```
    compile -inc -only_hold_time
```

# Memory Compiler
1. the number of words in memory must be a multiple of 2
2. The frequency of memory is usually set to 1, it is used for power analysis, s.t. we can further know the ring width for our memory.
3. Ring Width(The powerRail of your memory). Since $P=CV^2f$ , we already hold our $f=1$,average power of your circuit is usually around  18mW,typical current flow of your circuit is around 10(mA) yet Metal 1 of TSMC can only withstand $20(mA)$ of instant current. Remember that is ON AVERAGE. In peak condition, so we usually consider the ability of Metal 1 to widstand current to be 10 time smaller than the original value, i.e. estimate it to withstand 2(mA) only. Thus our Ring Width, the power rail of our memory should be $10/2=5$ thus choose it to be 5. If the ring width is wrong, your memory might just broke down after you tape out.
4. The max size of your memory is 16384. Anything larger is not buildable.
5. When using ROM, the data writing rule must be followed, i.e. if you call a 128x16 rom, the data your prepared must also be 128x16. Even if you only have 100 datas, you still need to zeropad the remaining 28 spaces, otherwise you cannot write data into rom.
6. To search for metal Info, find phy_lib, find M1 to serach for the maximum allowable current of the metal layer.
7. Memory is a LEAF CELL! Synthesis dc can ACTUALLY report the cell area of memory, however, you have to add Hierarchy to the memory cell model s.t. dc can find it. By adding hierachy to mem cell, percentage of area can be found.
8. ROM data gets written into ROM@GDS layout, if you want to reboot ROM or switch data, simply regenerate the ROM GDSs, no need for other modifications.

## Simulating memory
1. Memory actually has its own verilog model, but you have to check it in the generate Menu.
2. Synopsys model of memory has APT(Area Power Timing) infos of memory.
3. PostScript Datasheet is suggested s.t. you can know how your memory works.
4. This memory simulation model is only used for simulation, not synthesizable.

## APR of memory
1. LEF false layout file must be generated for the layout tool to reserve some space for the memory. False layout would be placed on the APR cell.
2. The dataSheet file of memory is in .ps form, convert it into pdf file using
```
    ps2pdf design.ps memory_data.pdf
```
3. Memory timing diagram must be studied. If the path for the data to be send is more than 2 cycles.
```
    multicycle_path 2
```
Can be set on memory.

## Before APR
1. prepare verilog file and sdc
2. Dont use ddc, since dont touch network and ideal network => CTS dead
3. Save ddc after synthesis no matter what!

## Add ROM cell into Synopsys
1. Use lc_shell to convert memory model into .db and .v file.
```
    read_lib rom_128x16_t18_slow_syn.lib
    write_lib rom128x16_t18 -output rom128x16_t18_slow_syn.db
```
Then design the rom128x16_t18_slow & also fast version of memory cell into your target library. Otherwise you top design cannot find your memory cell.

# Reports Analysis
## Timing
1. The following command catch the timing path from the specified nets.
```
report_timing -from X[1] //#start points

report_timing -to   Y[1] //#End points

report_timing -through mult_11_2s/* //search timing path for mid point circuit
```
2. To test how fast your circuit can run, Just give your ckt an impossible task, and see the negative slack/
3. When performing hold time analysis, the clk delay will always be 0.

4. In the timing report, select ```end``` and look at the top 10 delays to quickly scan through the top 10 critical timing path's slack of your design. Very useful!

5. If you add the I/O port, longer delay might occurs in ```out```, Due to the fault of TSRI machine having an output_load of 40. Remember to change the load value once you switch environment.


# High fan out problem
- 3 signals have high fan outs, clk,rst and se(scan enable) used in DFT
1. To solve high fan out problem simply
```
    set_ideal_network
        or
    set high_fanout_net_threshould 0
```
2. To find and debug high fanout nets, simply type
```
    report_net_fanout -high_fanout
```
Then find the cell and click the corresponding schemetic output in the schemetic to know what the net is trying to drive.

3. After you set clk_gating SET IDEAL_NETWORK! Otherwise, lots of unused buffer might be added onto the clk path after synthesizing.

# assign problem
Since APR might not be able layout for your assign signal in the final synthesis Result, so to prevent assign from appearing in the final synthesis result.
```
    set_fix_multiple_port_nets -all -buffer -constraints [get_delays *]
```
```-all``` means multiple assign on same wire, feedthrough path in circuit and constants assignment problem in circuit.

# return usage in script.tcl
1. return can be used as a breakpoint in tcl file for better debug.

# DDC file usage
- We actually only need ddc file to regenerate all other files. Actually only ddc is needed to generate layout of a cell design.
1. From ddc file we can generate .v file(generate .v)
2. from ddc file, (generate sdc)
```
    write_sdc -version 2.1 yoursdc.sdc
```
- Thus we can run simulation.
3. To run gate level-sim, must save the sdf file and making sure the delay constraints met you syn.v design, to do that
```
    $self_annotate("chip_syn.sdf",chip);
```
4. When writing sdf file, -load_delay net MUST BE ADDED, otherwise wrong sdf file is generated.

# SDF file annotation
1. To ensure sdf met syn.v 100%, must consider CHANGE NAMING RULE PROBLEM
Before saving into .sdf & syn.v
```
    change_names -hiearchy -rule verilog
```
2. One should not use different case name to distinguish signals in design otherwise signals might get mixed together in some tools like Calibre (DRC LVS checker)

3. To check if sdf == syn.v Check selfAnnotateInfo, check if annotate = 100%

# Verdi nWave
1. We use vcs for waveform
2. For the waveform of MEM, the extra command line must be added, otherwise, waveform cannot be dumped.
```
    nWave &
```
3. Read vcd in, use *.vcd to find all the vcd file.

# Synthesis Area report
1. We can actually analyze the final area of our circuit by the following formula
<br /> $A/C = A'$ where A is the cell area of synthesis report, C is the core-ultilization depends on your design and an important parameter for APR. $A'$ is the final cell area.
2. Usually $C$ is given as follows.

|No Black Box Or Mem| Some memory cell exists 2~3 |More than 4 memories|
|       ---         |         ---            |           ---|
|  0.95               |       0.8             |  0.65       |

These extra space would be used to implement the power rail of memories and other IP black boxes.
2. How many buv/inv are used?
> If there are too many buffers, consider using the multicycle command. Since dc is trying to insert buffers to combat the hold time violations on clk tree.
