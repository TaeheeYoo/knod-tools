# knod-tools

Host-side tools for the knod GPU network offload.

## knod-disasm

Disassembles a shader dump from any of knod's debugfs `insn` files.

    knod-disasm /sys/kernel/debug/dri/128/knod/bpf/insn
    knod-disasm /sys/kernel/debug/dri/128/knod/ipsec/insn
    cat insn | knod-disasm --hex

The kernel emits the shader as hex dwords and names the target it was built
for; mnemonics, operands and offsets come from LLVM here.  That keeps a
per-generation opcode table out of the kernel, and means a new GPU generation
needs no kernel change to be readable.

Two layouts turn up, and a dump says which one it is:

- **block** — nothing but dwords, as the ipsec shader is dumped.  Offsets are
  counted from the start of the shader.
- **annotated** — one instruction per line among text saying which BPF
  instruction it came from, as the BPF kernel is dumped.  That text is about
  the program rather than the ISA, so it is left alone; only the dwords are
  replaced, and the `; bpf#N` tags are lined up afterwards.

Branches show where they land as well as the distance they encode, which
llvm-objdump works out and which is worth having - a backward branch's operand
prints as a large unsigned number.

Requires `llvm-mc` and `llvm-objdump` on PATH.
