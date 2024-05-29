# Hello world program in eBPF

when using BCC's Python library, a sample hello world program looks like this.

```Python
from bcc import BPF

program = r"""
int hello (void *ctx){
  // this is a helper function  that is used to print "Hello world"
  bpf_trace_printk("Hello world")
  return 0
}
"""

# this step compiles the C program from plain text
b = BPF(text = program)
# Get the system call you want to attach your function to.
sys_call = b.get_syscall_fnname("execve")
# here we are attaching the function named hello to the system call "execve" using kprobe.
b.attach_kprobe(event=syscall, fn_name="hello")
# read the tracing output by the kernel and print it on the screen
b.trace_print()
```

## Flow of the program above

The Python program compiles the C code, loads it into the kernel, and attaches it to the execve syscall kprobe. Whenever any applica‐ tion on this (virtual) machine calls execve(), it triggers the eBPF hello() program, which writes a line of trace into a specific pseudofile.

The Python program reads the trace message from the pseu‐ dofile and displays it to the user.
