---
layout: single
categories: 
    - OS
author: Yue Yin

toc: true
toc_sticky: true
---




> Note: this post is largely based on MIT 6.828 Lab3. Thus, we use the term "user environment" instead of the more traditional term "process". The two are conceptually the same, but differs in interfaces and semantics.

## User Environment Basics

The kernel keeps track of user environments with three data structures: `struct Env *envs`, `struct Env *env_free_list`, and `struct Env* curenv`. `envs` is allocated at boot time, and `env_free_list` is initialized based on `envs`. Environment state:

```c
struct Env {
	struct Trapframe env_tf;	// Saved registers when being descheduled
	struct Env *env_link;		// Next free Env
	envid_t env_id;			    // Unique environment identifier of the env using this `struct Env`
	envid_t env_parent_id;		// env_id of this env's parent
	enum EnvType env_type;		// Indicates special system environments
	unsigned env_status;		// Status of the environment
	uint32_t env_runs;			// Number of times environment has run

	// Address space
	pde_t *env_pgdir;			// Kernel virtual address of page dir
};
```

To run a environment, the kernel need logic+data. Logic is primarily defined by `env_tf`, containing %esp, %eip, etc. Data is defined by the page directory and page tables. 

To create and run an evironment, call `env_create` and `env_run`. Pseudo-code:

```
env_alloc() {
    env = find from free list;
    initialize env;
}

env_setup_vm(e) {
    e->env_pgdir = page_alloc(); // allocate page directory
	e->env_pgdir = memcpy(kern_pgdir); // copy from kernel's page directory
}

load_icode(e) {
    switch from kern_pgdir to e->env_pgdir;
    load program data into e's address space using memset, memcpy;
    switch from e->env_pgdir to kern_pgdir;
    allocate user stack for e;
    set up e->env_tf;
}

env_create() {
    e = env_alloc();
    env_setup_vm(e);
    load_icode(e);
}

env_pop_tf(e) {
    pop e->env_tf into registers;
}

env_run(e) {
    switch from kern_pgdir to e->env_pgdir;
    env_pop_tf(e);
}
```



## Protected Control Transfer

Exceptions and interrupts are both *protected control transfers*, casuing CPU to switch from user mode (CPL=3) to kernel mode (CPL=0), without giving the user-mode code any opportunity to interfere the kernel or other environments. 

An *interrupt* is caused by asynchronous event, like I/O activities. An *exception* is caused synchronously by currently running code, like division by zero and invalid memory access. Each interrupt/exception is identified by a number in 0-255 (called *vector*).

To ensure protection, entry-points to the kernel are strictly limited. On x86, two mechanisms work together:

- When interrupt/exception happens, the CPU uses the vector as an index into the *interrupt descriptor table* (IDT), which was set up in the kernel-private memory. From the IDT entry, the CPU loads %eip (instruction pointer) and %cs (code segment), and starts executing in kernel mode.
- The CPU needs to save the *old* processor state before the interrupt/exception occurred, for future resumption. This place must be protected from the user-mode code and the *kernel stack* can be used. When the interrupt/exception cause a privilege change from user the kernel mode, the CPU reads the kernel stack location (segment selector and address) from the *task state segment* (TSS). The CPU pushes %ss, %esp, %eflags, %cs, %esp and an optional error code to the kernel stack. Then it loads %cs and %esp from the IDT entry, and sets %ss and %esp to refer to the kernel stack. When an interrupt/exception happens in kernel mode, no stack-switching happens. The CPU just pushes %eflags, %cs, %esp, and an optional error code. This supports nested exception.

The kernel should set up the IDT at boot time. Let's review the responsibility of software and hardware. When an interrupt/exception happens:

- The hardware reads from TSS to know the position of the kernel stack;
- The hardware saves %ss, %esp, %cs, %eip, and possibly an error code to the kernel stack, and updates %ss and %esp to refer to the kernel stack. This constructs part of the `struct Trapframe`. Hardware finishes all its jobs, and software takes over.
- Control passes to the entry point in `trapentry.S`, pushes other registers onto the kernel stack to complete the `struct Trapframe`. Then calls `trap`, with the trapframe as an argument, in `_alltraps`.
- The `trap` function in `trap.c` executes.
- In JOS, `trap` calls `trap_dispatch` to dispatch to specific handler. After that `trap` calls `env_run` to pop registers and fall back into user mode. 



## Exception Handling

System calls can be implemented with protected control transfer. The application pass the system call number and arguments in registers, then calls `int $0x30`. The kernel passes the return value in %eax. 

When the user-mode code tries to access an invalid address or one for which it has no permissions, the processors traps into kernel with information about the attempted operation. If the fault is fixable, the kernel fix it and the instruction is restarted. A fixable fault can be an extendible user stack, where the kernel allocates on demand. The kernel must carefully verify the address passed by the user correctly. 



## Reference

https://pdos.csail.mit.edu/6.828/2018/labs/lab3/#Handling-Interrupts-and-Exceptions