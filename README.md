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

A dump can use both.  The BPF one does: its body is annotated, but its
prologue is a block, because part of that may have been spliced in from a
prebuilt routine and the kernel cannot say where the instructions inside it
begin.  `# format` and `# base` apply from where they appear until the next
one, so they read per section rather than per file.

Branches show where they land as well as the distance they encode, which
llvm-objdump works out and which is worth having - a backward branch's operand
prints as a large unsigned number.

Requires `llvm-mc` and `llvm-objdump` on PATH.

## knod-blob-check

Checks a prebuilt routine against what the kernel's JIT emits for the same
thing.

    knod-blob-check /sys/kernel/debug/dri/128/knod/bpf/insn \
                    ~/proj/knod-blob/build/knod-bpf-gfx10.bin

The two have to agree instruction for instruction and nothing says so on its
own - editing one is not a build error in the other, and a shader wrong in this
way reads the wrong memory rather than faulting. Run it after touching either.

Only the prologue is covered so far: it is the one routine the kernel emits
somewhere a dump shows whole. What the JIT emits after it varies with the
program, so the comparison stops where the fixed part does and says how much
was left.

Needs `knod-disasm` on PATH.
