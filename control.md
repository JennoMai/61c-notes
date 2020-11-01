# Control
Control and status registers (CSR) are stored separately from the regfile. They monitor the status and performance of the hardware and manage communication with peripherals.
- Can have up to 4096 registers
- May count the number of cycles or instructions executed

## CSR Instructions
CSR instructions follow the same format as I-Format instructions, except the 12-bit immediate represents the address of the CSR.
- `csrrw` reads the CSR and stores it in `rd`. Then, it writes to contents of `rs1` into the CSR.
    - If the destination register is `x0`, the previous contents of the CSR are not stored.
    - The pseudoinstruction `csrw` is equivalent to the above scenario
- `csrrs` and `csrrc` set and clear the `flag` of a CSR, respectively
- `csrrwi`


## Control Logic Design
Although using read-only memory is popular for prototyping and manually designing control logic, combinatorial logic is faster and more compact.

### ROM-Based control
- Takes in an 11-bit instruction address as input
- Outputs 15-bit data containing the control signals
- Functions like a lookup table
    - First, instruction addresses are decoded to return a control word
    - The control word for the specific instruction is outputted