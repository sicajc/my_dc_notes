# DC notes and questions
1. Why should we learn Computer Architecture? Is it important when we are designing digital circuit?
> Not neccessary. Since it is just a basic knowledge you need to have, cpu is easiest architecture you can think of when implementing asic or vlsi architecture. So if you can handle the DSP VLSI architecture course well, you can actually bypass computer architecture courses, since its main focus is on cpu design. Architectural conecpt like systolic arrays, Advanced pipeline architecture and Filter architecture matters more.
2. Does industry seperate Controlpath and datapath completely?
> Yes. Moreover, basic design even further partition the controlpath into two parts, one for functional operations generation, another one for low-power control. So we will definitely seperate these two out.
> Usualy not. Because speed matters, so we would want the decoding to be as fast as possible, i.e. making the logic as simple as possible.
3. Will you use multi-level FSM when implementing Controlpath ckt?
4. Should we seperate Sequential logic and Combinational logic out when coding verilog?
> Not neccesary. However, seperating them out can make debug life easier. Thus we only do so when we want to trace a certain value propogating to another register. As a result, we do it only when we want to debug the circuit.
5. Is ternary operator faster than if~else?
> No. They are the same thing and will be synthesized into the same circuit.