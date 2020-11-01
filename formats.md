# RISC-V Instruction Formats
All instructions consist of 32 bits.


## R-Format Instructions
R-format instructions denote register-to-register instructions. They allocate bits in the 
following manner, from MSB to LSB:
- 7 - `funct7`, specifies instruction to perform
- 5 - `rs2`, the second register
- 5 - `rs1`, the first register
- 3 - `funct3`, specifies instruction to perform
- 5 - `rd`, the address of the destination register
- 7 - `opcode`, specifies type of instruction (`0110011` for R)

The second bit of `funct7` also specifies whether or not to sign-extend the operands.

## I-Format Instructions
I-format instructions denote instructions between registers and immediates. They replace the funct7 and
rs2 fields with a 12-bit field for signed immediates, as follows:

- 12 - `imm[11:0]`
- 5 - `rs1`, the first register
- 3 - `funct3`, specifies instruction to perform
- 5 - `rd`, the address of the destination register
- 7 - `opcode`, specifies type of instruction (`0010011` for I)

There are 8 I-type instructions in RISC-V, but the three bits in `funct3` can only store 7. Cleverly, the
following 12 bit `imm` field is implicitly reformatted for left and right shifts as follows:

- The five LSBs are allocated to specify the shift amount, for a maximum value of 31
- The second MSB is distinguishes between a right logical shift (0) and a right arithmetic shift (1).

### Loads
There is only one type of load in RISC-V, which takes a memory address stored in
`r1`, offsets it by `i`, and stores it in `rd`. With a destination register, a source register, and
an immediate offset, these instructions can be specified by the I-type format.
- The `opcode` for the load instruction is `0000011`.
- The immediate value specified in `imm` is added to the value stored in `r1`.
- `funct3` specifies the size of the data and whether or not it is signed.
    - The first bit (MSB) is `0` for signed, and `1` for unsigned.
    - The last two bits are `00` for a byte, `01` for a half-word, and `10` for a word. 

### Jump and Link Register
The instruction `jalr` is actually an I-type instruction, since it requires a destination register (typically `ra`), a source register
(containing the address to jump to), and an immediate to offset the jump.

**Unlike** branching and `jal`, `jalr` does not multiply the offset by 2.

`jalr` has various applications in both pseudoinstructions and jumping.
- `ret` = `jr ra` = `jalr x0, ra, 0`
- ```
  lui   reg1, uimm      # saves the upper 12 bits of an immediate
  jalr  ra, reg1, limm  # calls a function at any 32-bit absolute address
  ```
- ```
  auipc  reg1, uimm      # saves the value of the PC with an upper offset
  jalr   ra, reg1, limm  # calls a function with a PC-relative address
  ```

## S-Format Instructions
There is no destination register for stores, since we write directly to memory. Hence, the
`funct7` and `rd` fields are replaced by `imm` fields to specify 12-bit memory offsets.

- 7 - `imm[11:5]`, stores the most significant 7 bits of the immediate
- 5 - `rs2`, the source register holding the data to be stored
- 5 - `rs1`, the register holding the desired memory address
- 3 - `funct3`, the size of the data being stored; `000` for a byte, `001` for a halfword, and `010` for a word
- 5 - `imm[4:0]`, the least significant 5 bits of the immediate
- 7 - `opcode`, specifies type of instruction (`0100011` for S)

## B-Format Instructions
These instructions handle branching operations, which take two source registers and a label as operands.
Like store instructions, branch instructions do not have a destination register.

These instructions use **PC-Relative Addressing**; the `immediate` field is used to add a two's-complement
offset to the program counter.

RISC-V can support compressed instruction sets, in which each instruction is only 16 bits long (2 bytes, or a halfword).
In order to accomodate both 32-bit and 16-bit instruction sets, **the branch offset is only scaled by 2 bytes**.

Hence, branch instructions can only reach 2<sup>11</sup> 32-bit instructions.

Two 1-bit and 6-bit `imm` fields replace `funct7` in this instruction format. Similarly to S-Format instructions, the `rd`
field is replaced by two more `imm` fields for a total of 12 bits allocated to the immediate. The ordering of the bits is
shuffled for hardward efficiency.

- 1 - `imm[12]`, stores the most significant bit of the immediate; allows immediate to be sign-extended
- 6 - `imm[10:5]`, another immediate field
- 5 - `rs2`, the address of a register
- 5 - `rs1`, the address of a register
- 3 - `funct3`, specifies the type of branch
- 4 - `imm[4:1]`, bits 1-4 of the immediate; lacks the LSB
- 1 - `imm[11]`, the *second* most significant bit of the immediate field
- 7 - `opcode`, specifies type of instruction (`1100011` for B)

Because all byte offsets will be even (since we multiply the immediate by 2), a 12-bit immediate allows us to represent a 13-bit byte offset.
Hence, the least significant bit of the immediate does not need to be stored.

## U-Format Instructions
If we need to represent immediate values greater than 12-bits, we can use U-format instruction to represent long immediates.

- 20 - `imm[31:12]`, the upper 20 bits of a 32-bit immediate
- 5 - `rd`, the destination register
- 7 - `opcode`, specifying the type of instruction ( )

### Load Upper immediate
The `lui` instruction loads the upper 20 bits of the immediate specified by the `imm` field to a destination register, then clears the bottom 12 bits.
The `addi` instruction can then be used to fill in the full 32-bit value.

However, we must note that `addi` sign extends; hence, if the 12-bit value immediate has a 1 in its most significant bit, it will sign-extend and subtract
1 from the upper 20 bits. To counteract this, we can increment the `lui` immediate by 1.

Luckily, the `li` pseudoinstruction handles the usage of `lui`. Calling it with `li rd, imm` will automatically set up the two instructions needed.

### Add Upper Immediate Program Counter
The `auipc` instruction adds the upper immediate value to the program counter and stores it in the return address using the following syntax: `auipc rd, imm`.

This can be used for PC-relative addressing, as `auipc rd, 0` will store the current program counter value unchanged.

## J-Format Instructions
Jump instructions follow the J format. It has space to store a longer immediate, since there is only one type of jump (`jal`) and no source registers to consider.
Similarly to B-formats instructions, the ordering of the bits are shuffled.

- 1 - `imm[20]`, the most significant bit of the immediate
- 10 - `imm[10:1]`, lacks the least significant bit
- 1 - `imm[11]`, 11th bit of the immediate
- 8 - `imm[19:12]`, middle 8 bits of the immediate
- 5 - `rd`, the address of the destination register
- 7 - `opcode`, specifies the type of instruction (`1101111` for J)

As with branch instructions, `jal` offsets the PC by the value specified by the immediate in even amounts. It also saves `PC + 4` into `rd`, if specified.