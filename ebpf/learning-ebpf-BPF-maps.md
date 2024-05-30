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
# BPF_HASH is a macro that is used to create Hash.
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
