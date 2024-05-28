# KProbes (Kernel Probes)

They allow us to dynamically insert handlers into the kernel code without needing to modify the kernel source code, or recompiling kernel code.

## How KProbes work

- Probing: You set a probe at a specific point in the kernel code.
- Handler: When the kernel execution reaches that point, it triggers the handler function you defined.
- Handler execution: The handler executes it's logic which can collect information as well as modify the execution flow of kernel.

Example:

```c
# include <linux/kprobes.h>

static struct kprobe kp = {
  // symbol_name specifies the function to probe.
  .symbol_name = "do_sys_open"
};

// Handler function
// handler_pre is the function that is run before do_sys_open is executed.
int handler_pre(struct kprobe *p, struct pr_regs *regs){
  // printk is the kernel equivalent of `printf`
  printk(KERN_INFO "do_sys_open called \n");
  return 0;
}

// Initialization function
// function marked with __init is a macro, which is used to mark an initialization function for memory optimisation. This function is removed from the memory after the module is initialized
static int __init kprobe_init(void) {
  int ret;
  kp.pre_handler = handler_pre;
  ret = register_kprobe(&kp);
  if (ret < 0){
    printk(KERN_INFO "register_kprobe failed, returned %n", ret);
    return ret;
  }

  printk(KERN_INFO "KProbe registered\n");
  return 0;
}

// Cleanup function
// function marked with __exit runs when the module is unloaded.
static void __exit kprobe_exit(void){
  // unregister_kprobe unregisters the kprobe, cleaning up the resources.
  unregister_kprobe(&kp);
  printk(KERN_INFO "Kprobe unregisterd\n");
}

module_init(kprobe_init);
module_exit(kprobe_exit);

```

KProbes are like hooks, that are inserted into the kernel functions, like the `do_sys_open` function which is eventually run when a user space program tries to open a file, through kProbes, we run the logic present in the `pre_handler` before the actual kernel funciton.
