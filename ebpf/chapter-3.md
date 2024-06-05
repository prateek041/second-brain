## eBPF Virtual Machine

It takes eBPF bytecode insturctions and converts it into Machine code, that is understandable by Computers (CPU).

In the early days, eBPF programs were interpreted in the kernel, where the Kernel first converted eBPF bytecode into Machine code and then finally runs it, but it is replaced by JIT (Just in Time) Compilation, because of which conversion from bytecode to machine code happens once whien the program is loaded into the kernel.

> [!TODO]
> Revise stuff about CPU architectures related to registers and instruction sets.

## eBPF Registers

There are 10 general-purpose registers in eBPF virtual Machines which are numbered from 0-9. 10th register is used as a stack frame pointer.

eBPF has 11 general-purpose registers `(r0 - r10)` which have the following usecase.

- **r0-r5** are caller-saved
- **r6-r9** are callee-saved
- **r10** is frame pointer.

> [!IMPORTANT]
> These registers implemented in Software.

> [!NOTE]
> Dive deeper into how these registers work later on.

## eBPF Instructions

```c
struct bpf_insn {
  __u8 code; /*opcode*/
  __u8 dst_reg:4; /*dest register*/
  __u8 src_reg:4; /*source register*/
  __s16 off; /* signed offset */
  __s32 imm; /* signed immediate constant */
}
```

When eBPF bytecode is loaded into the kernel, it is represented as `bpf_insn` struct. The verifies checks them to ensure that the code is safe to run.

> [!NOTE]
> For more information about eBPF architecture, read [BPF and XDP reference Guide](https://docs.cilium.io/en/stable/bpf/) Page 40: learning eBPF.

## eBPF "Hello world" for a Network Interface

```c
// attaching the necessary header files to get support for eBPF development.
#include <linux/bpf.h>
#include <linux/bpf_helpers.h>

// Gloabl Variable.
int counter = 0;

// Defines a section.
SEC("xdp");
int hello(void *ctx){
  // libbpf's wrapper around kernel function `bpf_trace_printk()`.
  bpf_printk("Hello world %d", counter);
  counter ++;
  // Verdict telling kernel to pass network packet normally.
  return XDP_PASS;
}

// defining license so that verifier can make sure that helper functions used are supported.
chat LICENSE[] SEC("license") = "Dual BSD/GPL";
```

when the above C code is compiled to eBPF bytecode (so that eBPF VM can understand it) using a C compiler (run make all), it is converted into Object Code, which we can inspect using `file hello.bpf.o`. The output will look like this.

`hello.bpf.o: ELF 64-bit LSB relocatable, eBPF, version 1 (SYSV), with debug_info,
    not stripped` It gives the following information.

- **ELF**: Executable and Linked Format file.
- **eBPF**: Containing eBPF code.
- **64-bit LSB**: 64-bit platform, with LSB (least Significant bit) architecture.

> [!NOTE]
> This can be inspected further with `llvm-objdump` to see the eBPF instructions. e.g: llvm-objdump -S hello.ebpf.o

It will look like this

```object
hello.bpf.o:    file format elf64-bpf

Disassembly of section xdp:

0000000000000000 <hello>:
;     bpf_printk("Hello World %d", counter);
       0:       18 06 00 00 00 00 00 00 00 00 00 00 00 00 00 00 r6 = 0 ll
       2:       61 63 00 00 00 00 00 00 r3 = *(u32 *)(r6 + 0)
       3:       18 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 r1 = 0 ll
       5:       b7 02 00 00 0f 00 00 00 r2 = 15
       6:       85 00 00 00 06 00 00 00 call 6
;     counter++;
       7:       61 61 00 00 00 00 00 00 r1 = *(u32 *)(r6 + 0)
       8:       07 01 00 00 01 00 00 00 r1 += 1
       9:       63 16 00 00 00 00 00 00 *(u32 *)(r6 + 0) = r1
;     return XDP_PASS;
      10:       b7 00 00 00 02 00 00 00 r0 = 2
      11:       95 00 00 00 00 00 00 00 exit

```

> [!IMPORTANT]
> eBPF instructions are generally 8 bytes long, so on a 64 bit system, each memory location can hold 8 bytes, therefore to get next instruction, generally offset is increased by 1.

In the above Bytecode, there are a few things to understand.

- first instruction, is a "wide" instruction, which means it needs 16 bytes instead of 8 to be represented in memory (register) therefore for the second instruction, the offset becomes 2.

- This pattern can be seen in the entire bytecode, if the instruction is wide, the offset of the next instruction is increased by `2` instead of `1`.

> [!NOTE] > [eBPF opcode spec](https://github.com/iovisor/bpf-docs/blob/master/eBPF.md)

Let's try to understand one instruction.

`5:       b7 02 00 00 0f 00 00 00 r2 = 15`
