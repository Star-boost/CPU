# CPU

Look ma, I made a CPU! Here's what I did:

Part A:

Task 1: The ALU. We were given inputs A and B, and 4 ALUSel bits. The key to this
task was performing ALL of the potential ALU operations on A and B and having those
values to manipulate. Knowing that each possible ALUSel value maps to either "Unused" or one of the 
instructions, I used a Multiplexer (mux) with ALUSel as the selector to select from all of the
operations performed on A and B, ultimately spitting out one result. The reason behind this design
choice was that using the mux was a very clean way to filter out the result we needed while keeping
things simple.

Task 2: The RegFile. This task involved four parts on a macro level. The first was to figure out 
which register would actually get written to. This was achieved with a demultiplexor (demux) that 
used the write register input as the selector that would either output a 1 or 0 given the input to
the demux which was a single write enable bit. The next two parts involved what data we actually 
read, which is based off of read_reg1 and read_reg2. Two muxes were used and the corresponding 
read_reg bit was used, with the output being the output from all of the 32 registers (implemented
next). The last part was actually placing all 32 registers, with write_data, the register's corresponding
write_en bit, and the clock (clk) as inputs. (x0 was special in that its reset was 1 to ensure it always
spits out zero). The output from each register was used as inputs in the two read_reg muxes, and for 
some of the special registers, the outputs were routed to the debug/test outputs.

Task 3: The Immediate Generator. To generate the immediate from the instruction passed in, I used the 
risc-v green card to determine what bits made up the immediate in the addi instruction (20-31) and 
used a splitter to only get just those bits. After sign-extending, we got our immediate. The immediate 
selector (ImmSel) was ignored for addi.

Task 4: Datapath (addi only). This task involved following through on the 5 step process of executing 
a risc-v instruction (Instruction Fetch (IF), Instruction Decode (ID), Execute (EX), Memory (MEM),Write Back (WB))/.
IF required no additional wiring because the program counter (PC) was already provided. For instruction decode, 
I used the risc-v green card to determine what bits corresponded to what part of the instruction and used a splitter
accordingly. rs1, rd, and clk were connected to the regfile, and the immediate generator was used to get the immediate.
For EX, I used read_data_1 from the regfile as input A and the immediate we got for input B. ALUSel was hard-coded to 0
for addi. For MEM, since addi doesn't use memory, we didn't have to implement anything here. Lastly, for WB I put the 
alu_result into write_data on the regfile, and hard-coded RegWEn to 1 for the addi instruction. And that's all!

Task 5: Readme. That's a little meta.


Part B:

Control logic: I opted to use the ROM method, because having this hardcoded mapping made sense to me early on. The
primary means of getting the ROM output involved breaking down the instruction first by opcode, then by funct3 and 
funct7 (if a given function has them) and using constants to compare the funct3 and funct7 of a given instruction
to give a boolean (e.g. "is_addi") that I can use to identify what instruction is passed in. Then, within a given
Instruction type, I used a priority encoder and used the ROM mappings to choose between the booleans. I then used 
additional logic and bit extension to get the corresponding 6-bit ROM output for each instruction. However, since
only one output could be used for the ROM, I used an OR gate for each potential ROM output, using the fact that one
will be non-zero and the rest will be zero because of the logic I used, to get the right ROM input.


Advantages/Disadvantages of your design: A big advantage from my design definitely has to do with the simplicity of 
it. I felt that my isolation by instruction type and actual instruction really allowed me to filter instructions 
quickly and execute them. I definitely believe there were places where I could simplify things even further, but overall
things look good. One disadvantage of my design might be that using the ROM might not be as robust as hardcoding as
was done in previous semesters. However, from a designer perspective, the ROM definitely saved a lot of headaches
because you know exactly (thanks to the spreadsheet) what values in your control logic contribute to the output,
and simplifying using the sheet rather than gates definitely was worth the tradeoff of accessing the ROM each 
time.


Best/Worst bug or design challenge you encountered, and your solution to it: The biggest design challenge I was met
with was in the control logic, when getting the ROM output for each Instruction's priority encoder. I was struggling
to find a way to make all other ROM outputs zero and the ROM output for the passed in instruction to be the correct
mapping. In order to fix this, I took advantage of the "input extend" feature of the bit extender, where I passed in
the boolean (e.g. "is_I_type") as what I would extend the output of the enable bit for the priority encoder with, so if
the component was disabled (we're not dealing with this instruction type) then we take 0 from the enable output and then 
extend it with the boolean mentioned previously. Then we can AND that with the ROM output, so we'll either get 0b000000
or the corresponding ROM output. Then we're able to get a single output from the OR gate as our ROM input. In hindsight,
there are simpler ways to achieve the same effect, but I was glad I was able to use the components of Logisim and explore
their limits to get a working solution.
