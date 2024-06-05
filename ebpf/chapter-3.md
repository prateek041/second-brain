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
> When an eBPF program finishes running, the return value is present in Register 0.

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

- b7 is the instruction set, which means `dst = imm` which means "Set the destination to immediate value".
- immediate value is defined by the next byte, `0x02` which here means "register 2".
- `0x0f` is the value to be loaded, which is 15 in decimal.
- These in combination say "set Register 2 to value 15".

> [!NOTE]
> Also, right after the bytecode instructions, there is human readable version as well, which says "r2=15".

### Load the program

We use bpftool tool to load the program into the kernel, we use command.

`bpftool prog load <file-name.bpf.o> /sys/fs/bpf/<file-name>`
d
a

### Inspecting the loaded pdrogram

- Get list of all the loaded program: `bpftool prog list`. Here each program has an ID, we can use that ID to get more information about it.
- To get more information, use `bpftool prog show id <id number> --pretty`, the `--pretty` tag is to get a prettified JSON output.

We might get something like this:

```json
{
  "id": 76,
  "type": "xdp",
  "name": "hello",
  "tag": "d35b94b4c0c10efb",
  "gpl_compatible": true,
  "loaded_at": 1717606591,
  "uid": 0,
  "orphaned": false,
  "bytes_xlated": 96,
  "jited": true,
  "bytes_jited": 140,
  "bytes_memlock": 4096,
  "map_ids": [3, 4],
  "btf_id": 103
}
```

> [!NOTE]
> To get more informatigon, go to page 46 of learning eBPF.
> Also, when the compiled program is loaded into the kernel, it is first checked to be safe by the verifier.

Let's look at the translated byte-code.

`bpftool prog dump xlated name hello`

```object
int hello(struct xdp_md * ctx):
; bpf_printk("Hello World %d", counter);
   0: (18) r6 = map[id:3][0]+0
   2: (61) r3 = *(u32 *)(r6 +0)
   3: (18) r1 = map[id:4][0]+0
   5: (b7) r2 = 15
   6: (85) call bpf_trace_printk#-74096
; counter++;
   7: (61) r1 = *(u32 *)(r6 +0)
   8: (07) r1 += 1
   9: (63) *(u32 *)(r6 +0) = r1
; return XDP_PASS;
  10: (b7) r0 = 2
  11: (95) exit
```

This is low level code, but not yet machine code. The JIT compiler will convert the eBPF bytecode into machine code.

### JIT Compiled Machine Code

We can see the Just In Time (JIT) compiled machine code using `bpftool` through the command, `bpftool prog dump jited name hello`, the output will be something like this.

```assembly

int hello(struct xdp_md * ctx):
bpf_prog_d35b94b4c0c10efb_hello:
; bpf_printk("Hello World %d", counter);
   0:   stp     x29, x30, [sp, #-16]!
   4:   mov     x29, sp
   8:   stp     x19, x20, [sp, #-16]!
   c:   stp     x21, x22, [sp, #-16]!
  10:   stp     x25, x26, [sp, #-16]!
  14:   mov     x25, sp
  18:   mov     x26, #0
  1c:   sub     sp, sp, #0
  20:   mov     x19, #-281474976710656
  24:   movk    x19, #32768, lsl #32
  28:   movk    x19, #2069, lsl #16
  2c:   mov     x10, #0
  30:   ldr     w2, [x19, x10]
  34:   mov     x0, #-200501958279169
  38:   movk    x0, #715, lsl #16
  3c:   movk    x0, #6928
  40:   mov     x1, #15
  44:   mov     x10, #-12928
  48:   movk    x10, #63531, lsl #16
  4c:   movk    x10, #53584, lsl #32
  50:   blr     x10
  54:   add     x7, x0, #0
; counter++;
  58:   mov     x10, #0
  5c:   ldr     w0, [x19, x10]
  60:   add     x0, x0, #1
  64:   mov     x10, #0
  68:   str     w0, [x19, x10]
; return XDP_PASS;
  6c:   mov     x7, #2
  70:   mov     sp, sp
  74:   ldp     x25, x26, [sp], #16
  78:   ldp     x21, x22, [sp], #16
  7c:   ldp     x19, x20, [sp], #16
  80:   ldp     x29, x30, [sp], #16
  84:   add     x0, x7, #0
  88:   ret
```

> [!WARNING]
> Thie program has been loaded into the kernel, but it is not attached to any event yet.

### Attaching to an Event

To attach an XDP program to an XDP event, we can use bpftool like this.

`bpftool net attach xdp id <program id> dev eth0`

The above command attaches the program `program id` to eth0 interface.
