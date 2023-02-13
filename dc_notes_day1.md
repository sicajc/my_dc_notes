# DC notes and questions
1. Is synthesis result 1-1?
> Yes. If the constraint you gave are fully specified and complete, the design will be 1-1 and not changing much.
2. How to ensure resource sharing is used?
> Instantiate the arithmetic unit out alone, and then send the data in using a selector mux.
3. Do instantiating multiple wires increase Area?
> No. Wires does not contain any cell area, wires are flying above the main design cell, so it doesn't matter.
4. Does modularizing ckt minimize area and optimize ckt better>
> No. You have to ungroup the modules to give dc more degree of freedom to do the optimization.
5. Where can I find larger functional blocks?
> Synopsys designWare has lots of free IPs for you to use. e.g. AHB or AXI.
6. Where can I learn the right naming convention and check whether they are right or not?
> Use spyglass to ensure that your naming convention is at least similiar to industrial standard.
7. Should an input register be placed in the input of the module?
> It depends.
8. Are there negedge triggered design memory?
> No. IC contest is bullshit. Most memory design sends its output in posedege and cost a certain delay. Just like d_ff.
9. When are latches used?
> Barrael shifter kind of design. It can be used to save lots of area. However, two-phase clock has to be used to handle this kind of design.
10. When should I add reset? When shouldnt?
> In datapath components, reset can be omitted. However, for controller where initial value is important, reset is a must.
11. Why should we add register delays in main design?
> The wave form simulation in RTL would be similiar if you add delays to them.
> When adding memory IPs, memory has their own delays, if you dont specify the delays in design, dc might not simulate the design for you.
If you dont want delays, but want to use memory IP, add the following command
```bash
    +notimingcheck
```
# Other notes and methedology.
1. GTECH v.s. Gate-level design. Gtech is the virtual technology gate used to display how your design would briefy look like in gate level, not yet compiled.
2. ```* added to specify all the files in the directory.```
3. $search_path command is needed and kept when specifying target library in synopsys.dcdetup
4. Usually I/O has ESD,Level shifter and Buffer made by TSMC to prevent suddent change in voltage destroy the ckt.
5. Better look into dcsetup file to ensure you use the right technology for your design.

# Synopsys solveNet
1. To use the resource provided by synopsys, call Anna Hsu, email her for account, then you can view the document or serach for problems in solvenet.
2. Simply keying the error messages into the serach bar can sometimes solve your problem.
3. Training section includes all the new tech or updates toward dc.
4. Methodology section has GUI,SCRIPTS and can help you auto generate the scripts you want. Simply download the script and modify to the script you want.

# DC NEXT v.s. DC
1. This calls dc next, version 2 of dc.
> dcnxt_shell -gui
2. This calls current old version dc.
> dc_shell -gui
If the licensed is not enough in old mode dc, switching into newer version might be a better idea, since not much people knows about it. Can prevent traffic when lots of people are connecting into the work station.

# Analyze mode in DC
1. When you use analyze, MODELs are created, such that parameterized circuit can be created through such design.
2. In normal compile flow, parameterized ckt is not allowed.
3. -autoread command is useful.
4. When viewing Hierarchy, viewing the design you want and simply elaborate it is a better idea. Since you are going to coorporate with others in the late future.

# Saving the design
1. You can save the steps and design you just made in "DESIGN_NAME.ddc"file s.t. you can simply just reopen the design you just made.
2. When saving, Unchecking "SAVE ALL DESIGN IN HIERARCHY" enables the design of 3D design space. Checking uses 2D only.


# Basic TCLs
1. set a {$b}. The value in {} is varying one, this is usually used.
2. $[expr $b + 2], within the bracket, it is treated as a command line argument.
3. Remember to add  ;# to tcl if you need to add COMMENTS.
4. get_cells, get_ports, get_pins, get_clk.
5. ```*U*``` this is usualy used to serach for all U in the design.
6. */U means which cell, / all hierachy's U, -h can be added here.
7. ```*``` meaning all.
8. source the 1.tcshrc , to prerun the procedure you want for the design.
9. ```link``` Check if the design are all read in.
10. A bug exists in dc, you must reOpen your design view, everytime you finish synthesis. Thus one must NewHierarchy/ reopen the design window. Delete the old one.
11. GUI of dc would generate scripts in GREEN line. You can simply copy it and removes the number for later use.
12. history commands of dc_shell contains all the scripts you just executed on dc. Can be reUsed.
13. myscript.tcl can be generated, simply by copy the scripts generated in the dc history. Slightly modifying it to make it work.
# DC shell
1. Size of in,out ports.
> sizeof_collection {all_inputs} or {all_outputs}
2. Size of ports.
> sizeof_collection [get_ports]
- Port number must be known because in order to tape out, port numbers should be less than 204 for packaging.
3. ```sizeof_collection [get_cells *reg* -hier]```

# Latch in DC
1. DC simply gives up on STA toward latches.

# Synthesis
1. When you enter a new company, you had better first scan through the databook.pdf which tells you about the cell-library this company supports.
2. If the cell does not exists. Like asynchronous reset flip flop. 90% does not have this. Your ckt would definitely be not synthesizable. So you must first check whether your cell-library supports this call.

# Compiler directives
This kind of design method is used for one-hot coding. And makes decoding a lot faster.
Otherwise without the directives, priority encoder is generated. Longer delays.
```verilog
always@(*)
begin // synopsys full_case parrallel_case
    case(2'b11)
    u: out = 10'b001;
    v: out = 10'b010;
    w: out = 10'b100;
    default: out = 10'b000;
    endcase
end
```
- full_case prevents latches.

This removes the delay info during synthesis.
```verilog
`define del 2
always@(posedge clk)
begin
    //synopsys translate_off
    #`del;
    //synopsys translate_on
end
```

## Partial synthesis
```verilog
`define A

`ifdef A
statement
`elsif B
statement
`else
C
`endif

```
- This is very helpful

# DesignWare
1. To see manual for student. the directory is usually
2. usr-cad-CIC-dw-doc-manuals-userguide

# DC for loop and while loop
1. DC does not support decrement arithmetic in for loop
> i=i-1 operation is not supported.
- However,
> i=i+1 is supported.
