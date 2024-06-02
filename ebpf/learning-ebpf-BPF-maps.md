# BPF Maps

A Map is a data structure that can be accessed by an eBPF program as well as a user space program. It has many use cases because it allows many eBPF programs to talk to each other, also allowing user-space applications to talk to the eBPF programs.

- User space applications can write configuration files to be read by eBPF programs.
- An eBPF program can store state, which can be picked up by some other program at some point in future.
- An eBPF program can write metrics in to the Map, which can be picked up by user space programs and displayed to the users.

## Hash Table Map

```python
from bcc import BPF
from time import sleep

program = r"""
# BPF_HASH is a BCC  macro that is used to create Hash.
BPF_HASH(counter_table);

# here the sample map looks like this
# {
#   uid: counter
# }

# ctx provides data about the event triggering the hello program.
int hello(void *ctx) {
   u64 uid;
   u64 counter = 0;
   u64 *p;

   # bpf_get_current_uid_get() returns the UID (User ID) as well as the PID (process ID)
   # of the current process. It is a 64 bit value which is masked using the Bitwise & Operator.
   # since only the first 32 bits are information related to the process.
   uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;
   # lookup the counter associated with the current UID, if it exists, it returns a pointer associated
   # to the counter, except it returns 0.
   p = counter_table.lookup(&uid);
   if (p != 0) {
      counter = *p;
   }
   # get the counter and update it.
   counter++;
   # update the counter table with the update count value.
   counter_table.update(&uid, &counter);
   return 0;
}
"""

b = BPF(text=program)
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")

# Attach to a tracepoint that gets hit for all syscalls
# b.attach_raw_tracepoint(tp="sys_enter", fn_name="hello")

while True:
    sleep(2)
    s = ""
    for k,v in b["counter_table"].items():
        s += f"ID {k.value}: {v.value}\t"
    print(s)

```
> [!NOTE]
> We are using pointers for Hash table operations instead of directly passing key variables because it gives flexibility to the program. We are not constrained to a specific type of key, since a pointer can be used to pass any type of key and the size of the pointer will also be always fixed.
> Pointers have a fixed size (e.g. 8 bytes) this makes the operations on the map (lookup, delete etc.) consistent.
> Also, the code above is not proper C, it is a modified version of C that BCC rewrites before it sends the code to the compiler.

## Perf and Ring Buffer Maps

In the above example, we used a Hash Map to send data from the Kernel space to the User space, which is convenient if the data we are dealing with is in that format.

But the downside is, the user space application has to constantly poll the table on regular basis, this issue is tackled with using the `perf subsystem` for sending data from the kernel space to user space using perf buffers and ring buffers.

### Prerequisites

**Perf Subsystem** in linux is a performance analysis tool that provides a framework for collecting and analyzing performance data.

Some of the features include but not limited to:

- 1: Event Monitoring: `perf` can monitor a wide range of events, including CPU cycles, cache misses, context switches and more, these events can be hardware generated (by the CPU), or Software generated (Kernel generated).
- 2: Performance Counters: It uses performance counters available in modern CPUs to count specific types of events.
- Dynamic Probing: Using `kprobes` and `uprobes`, `perf` can dynamically insert probes into the kernel and user-space code to collect data at specific point of interest.

#### Perf Buffers

These are circular memory buffers that are used to collect and store performance data before it is read and processed by user-space tools.

**Usage**

- Writing: When an event is triggered, the data is written into the buffer.
- Reading: User space application can read from these buffers to retrieve the collected performance data.
> [!NOTE]
> The data is read in chunks, which helps in minimizing the overhead and maximizing the performance of monitoring.

[!TODO] compare the `BPF ring buffer` and `BPF perf buffers`.

[!TODO] Give this file a proper name

```python
program = r"""
// using BCC macro `BPF_PERF_OUTPUT` to create a map named `output`, that will be used to pass messages from kernel
// to user space.
BPF_PERF_OUTPUT(output);

struct data_t {
    int pid;
    int uid;
    char command[16];
    char message[12];
};

int hello (void *ctx){
    struct data_t data = {};
    char message[12] = "Hello World";

    // bpf_get_current_pid_tgid() helper function that gets the ID of the process that triggered this eBPF program to run.
    // PID is top 32 bits.
    data.pid = bpf_get_current_pid_tgid() >> 32;
    // bpf_get_current_uid_gid returns the user ID, which are the lower 32 bits.
    data.uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;

    // bpf_get_current_comm is a helper function for getting the name of the executable (or command) that ran the
    // process which made the `execve` syscall.
    bpf_get_current_comm(&data.command, sizeof(data.command));
    bpf_probe_read_kernel(&data.message, sizeof(data.message), message);

    // output.perf_submit puts the data into the map.
    output.perf_submit(ctx, &data, sizeof(data));

    return 0;
}
"""

b = BPF(text=program)
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")

def print_event(cpu, data, size):
    # b["output"] gives us access to the map, b["output"].event() is used to grab data from that map.
    data = b["output"].event(data)
    print(f"{data.pid} {data.uid} {data.command.decode()}" + \
        f"{data.message.decode()}")

# open_perf_buffer() opens the perf ring buffer. It takes print_event function as a callback, which it runs every time data is to be read from the buffer.
b["output"].open_perf_buffer(print_event)
while True:
    b.perf_buffer_poll()

```

[!TODO] verify the statement "It's also highly performant, since all this information can be gathered within the kernel, without the need for any synchronous context switching to user space", on page 28.

## Function Calls

### Always Inline

Arguments in a function are executed one after the other, when a function calls another function, it causes a "jump" to the set of instructions of the called function. After the called function finishes execution (returns), the calling function arguments start executing just from below the statement which called the other function.

But, in case of "__always_inlint", there is no jump instruction, instead of a copy of the function's instructions is emitted directly within the calling function.

```c
static __always_inline void my_function(void *ctx, int val)
```

Starting from Linux Kernel version 4.16, this limitation has been lifted. But, BCC doesn't support the "BPF subprograms" features. We will be using "tail calls" instead.

[!TODO] Checkout how things work when using Go tooling instead of BCC.

## Tail Calls

Tail calls are a concept in programming and execution where a function calls another function as it's final action. This allows optimization techniques such as Tail Call Optimization (TCO).

Tail call occurs when a function calls another function as it's last statement before returning. There should be nothing more left to do in the function except returning the result.

### Tail Call Optimisation

Some Compilers and Interpreters have an optimisation technique where if a function call is a tail call, it removes the current function's stack frames and replaces it with the new function's stack hence preventing the growth of the call stack and leads to significant performance benifits.

Example, a factorial program written using recursion 

```c
int factorial(int n){
    return n * factorial(n-1);
}
```

can be re-written into:

```c
int factorial_helper(int n, acc){
    if (n == 0) return acc;
    return factorial_helper(n-1, n * acc);
}

int factorial(int n){
    return factorial_helper(n, 1)
}
```

In the first implementation, when the function call returns to the calling function, there is still a computation task of multiplying the result with `n` therefore the call stack needs to be maintained, but in the second implementation, there is no more computation left in the `factorial_helper` function and hence each call to `factorial_helper` replaces the previous frames from the stack.

> [!NOTE]
> This is very useful in eBPF because stack is limited to **512 bytes** only.


> [!IMPORTANT] `long bpf_tail_call(void *ctx, struct bpf_map *prog_array_map, u32 index)`.
> **ctx**: allows passing the context from the calling function to the called function.
> **prog_array_map**: is an eBPF map of type `BPF_MAP_TYPE_PROG_ARRAY`, which holds a set of file descriptors that identify eBPF programs.
> **index**: indicates which of the set of eBPF programs should be invoked.
> The helper doesn't return on successful execution, but returns to the calling eBPF program if it fails.

[!TODO]: Attach link to "File descriptor" file which is writer-reader-interface-go.md

#### Understing Tracepoint

Tracepoints are instrumentation points embedded in the Linux Kernel Code. It allows developers and system administrators to monitor and trace the execution of the kernel and understand it's behaviour under different conditions.

There are multiple events like system calls, context switches, interrupts etc. in kernel code, which has Tracepoints placed in specific locations, so that data about these events can be collected.

We use tools like BCC to attaches a function to a Tracepoint.

> [!TODO] Figure out how Tracepoint execution works.

>[!NOTE] Linux has almost 300 system calls, so the size of the BPF_PROG_ARRAY is ok to be around 300 entries.

[**List of Linux System call with their Opcodes**](https://filippo.io/linux-syscall-table/).

```python
program = r"""
// BCC macro to create a map of type BPF_MAP_TYPE_PROG_ARRAY, it's name is `syscall` and allows 300 entries
BPF_PROG_ARRAY(syscall, 300);

// hello function will be called anytime `sys_enter` raw tracepoint is hit, the context passed to an eBPF program
// attached to a raw tracepoint takes the form of bpf_raw_tracepoint_args.
int hello (struct bpf_raw_tracepoint_args *ctx){

    // using a shorthand for derefrencing the ctx pointer and accessing it's `args` property, which holds the opcode
    // of system call being made.
    int opcode = ctx -> args[1];
    // below code get re-written to bpf_tail_call(ctx, syscall, opcode)
    syscall.call(ctx, opcode);
    // lines below this will not run if tailcalls succeed.
    bpf_trace_printk("Another syscall: %d", opcode);
    return 0;
}

// Below are the Tail Call functions that we will load in our map and execute using bpf_tail_call() function.

// hello_execve is a program that will be loaded into the syscall program array map.
// this function will be executed as a tail call when opcode indicates it's an execve() syscall.
int hello_execve(void *ctx){
    bpf_trace_printk("Executing a program);
    return 0;
}

// hello_timer is program that will be loaded into the syscall program array map.
// this function will be executed as a tail call when the opcode indicates timer related operations.
int hello_timer(struct bpf_raw_tracepoint_args *ctx){
    if (ctx -> args[1] == 222){
        bpf_trace_printk("Creating a timer");
    } else if (ctx ->args[1] == 226){
        bpf_trace_printk("Deleting a timer");
    } else {
        bpf_trace_printk("Some other timer operation")
    }
    return 0;
}

// ignore_opcode is tail call program that will be executed for syscalls we don't want to trace.
int ignore_opcode(void *ctx){
    return 0
}

"""

b = BPF(text=program)
b.attach_raw_tracepoint(tp="sys_enter", fn_name="hello")

# b.load_func loads the functions into the BPF object, the flag BPF.RAW_TRACEPOINT indicates that these funcitons
# are designed to be attacehed to raw tracepoints.
# This function returns a file-descriptor.
ignore_fn = b.load_func("ignore_opcode", BPF.RAW_TRACEPOINT)
timer_fn= b.load_func("hello_timer", BPF.RAW_TRACEPOINT)
exec_fn= b.load_func("hello_execve", BPF.RAW_TRACEPOINT)

# Get the program array.
prog_array = b.get_table("syscall")
# ct.c_int(59) creates a C integer object with the value 59.
# Below we are filling the array_map with keys pointing to syscall opcodes and values pointing to fd of functions
# to be executed when the matching syscall executes.
prog_array[ct.c_int(59)] = ct.c_int(exec_fn.fd)
prog_array[ct.c_int(222)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(223)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(224)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(225)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(226)] = ct.c_int(timer_fn.fd)

# Ignore some syscalls that come up a lot
prog_array[ct.c_int(21)] = ct.c_int(ignore_fn.fd)
prog_array[ct.c_int(22)] = ct.c_int(ignore_fn.fd)
prog_array[ct.c_int(25)] = ct.c_int(ignore_fn.fd)
prog_array[ct.c_int(29)] = ct.c_int(ignore_fn.fd)
prog_array[ct.c_int(291)] = ct.c_int(ignore_fn.fd)

b.trace_print()
```
> [!NOTE] We can chain up to 33 Tail calls together, with eBPF programs of 1 million instructions, we can write very complex code to run entirely in the kernel.