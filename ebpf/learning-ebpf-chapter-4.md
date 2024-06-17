# BPF System Call

bpf() is used to "perform a command on an extended BPF program or map". bpf()'s signature is as follows:

`int bpf(int cmd, union bpf_attr *attr, unsigned int size);`

- int cmd: specifies which command we want to run.
- union bpf_attr \*attr: holds whatever data is needed to specify the parameters of the command.
- int size: indicates how many bytes of data there are in attr.

Now, let's see how the bpf sys-call works.

```c
// user_msg_t is a structure definition for holding user message
struct user_msg_t {
  chat message[12];
};

// BPF_HASH macro is used to create a hash table map, called config, which stores value of type user_msg_t, and the keys
// are u32, which are correct size for storing user ID.
BPF_HASH(config, u32, struct user_msg_t);

// Macro, defining a map named output, to send message from kernel space to user space.
BPF_PERF_OUTPUT(output);

struct data_t{
  int pid;
  int uid;
  char command[16];
  char message[12];
};

int hello(void *ctx){
  struct data_t data = {};
  struct user_msg_t *p;
  char message[12] = "Hello World";

  data.pid = bpf_get_current_pid_tgid() >> 32;
  data.uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;

  bpf_get_current_comm(&data.command, sizeof( data.command));

  p = config.lookup(&data.uid);
  if (p!=0){
    bpf_probe_read_kernel(&data.message, sizeof(data.message), p->message);
  } else{
    bpf_probe_read_kernel(&data.message, sizeof(data.message), message);
  }

  output.pref_submit(ctx, &data, size(data));
  return 0;

}


```
