# Single-Cycle CPU Datapath
There are five phases in the execution of an instruction:
- Instruction fetch: ~200 ps
    - Begins at the rising edge of the first clock cycle
    - PC+4 is calculated by the adder during this time
    - Completes once `inst[31:0]` is output by the IMEM.
- Instruction decoding: ~100 ps
    - Begins with the control unit recieving the instruction
    - The control unit sets its variables
    - Completes when the instruction is split and sent to the regfile
- Execution: ~200 ps
    - Begins with the register values being output by the regfile
    - Ends once the ALU outputs its result
- Memory access: ~200 ps
- Writeback: ~100 ps
    - The PC counter and regfile are updated simultaneously at the next rising edge of the clock
## R-Format
- The datapath begins with a register for the program counter (PC)
    - The output of this register sent to an adder as well as the instruction memory (IMEM)
    - The adder increments the PC by 4 and sends it back to the input of the PC
- The IMEM takes the address given by the PC and fetches the instruction to be executed
    - The output is sent to the regfile and the control unit
- The regfile retrieves the destination and argument registers
    - The argument registers are inputted into the ALU
    - Note that the `RegWriteEnable` permission is set to `1` by default for R-format instructions
- The ALU performs the operation and returns the output to the destination register
    - The control unit selects between addition and subtraction with `ALUSel`; addition is `0`, while subtraction is `1`.
## I-Format
- An immediate generator is added in order to handle immediate values
    - Bits `[31:20]` outputted by the IMEM are sent to an immediate generator
    - The control unit feeds `ImmSel` to the immediate selector to specify what format the immediate has been given in
    - The result is outputted to a multiplexor
- A multiplexor is added to select between a second register and an immediate
    - It takes in `rs2` and `imm` and outputs either to the ALU depending on `BSel` given by the control unit
    - `rs2` corresponds to `0`, `imm` corresponds to `1`.

### Loads
- The output of the ALU may actually be the address of a load
- Data memory (DMEM) is appended to the output of the ALU
    - Read-write permissions are given by the `MemRW` from the control unit
    - The data is fetched and outputted to a multiplexor
- A multiplexor is added between the ALU and its path back to the destination register
    - `WBSel` from the control unit determines whether data directly from the ALU or DMEM is sent back to the destination register
- In order to handle loads of different sizes, we add more combinational logic/mux units

## S-Format
- `rs2` from the regfile is also sent directly to `DataW` of the DMEM
- The control unit sets `RegWEn` to `0` to disable writing to registers and `MemRW` is set to `W` to allow for writing to DMEM
- Data is only written into memory on the next clock tick

## B-Format
- A multiplexor is added in front of the PC to select between adding 4 to the PC and branching
    - It takes input from the adder and the ALU
    - `PCSel` from the control unit determines whether or not the branch is taken
- Hardware to perform branch comparisons is added to receive output from `rs1` and `rs2`
    - `BrUn` from the control unit specifies whether the branch is signed or unsigned
    - Produces two one-bit outputs; `BrEq` if the values are equal and `BrLT` if `rs1` is less than `rs2`
- Another multiplexor is added between `DataA` and the ALU
    - It takes the current PC as an input and `DataA` as another
    - `ASel` from the control unit determines whether the ALU does addition with `DataA` (as usual) or with the PC (for a branch)

## Jump Instructions
### jalr
- We widen the `WBSel` multiplexor and allow it to take in `PC + 4` as a third value
    - If `WBSel` is `2`, then the address of the next instruction is written into the destination register
- `ImmSel` is set to the I type, since immediates in J-Format instructions are written in the same format as I-Formats
### jal
- `ImmSel` is set to the J type

## U-Format
- `ImmSel` is set to the B type
- For `lui`, `ALUSel` is set to B, for pass-through; no operation is performed on the generated immediate
- For `auipc`, `ALUSel` is set to add in order to add the immediate to the PC