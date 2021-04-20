---
layout: single
categories: 
    - OS
author: Yue Yin

toc: true
toc_sticky: true
---

## Multiprocessor support

All details explained [here](https://pdos.csail.mit.edu/6.828/2018/labs/lab4/#Part-A--Multiprocessor-Support-and-Cooperative-Multitasking). Not too important for interview though. 

Per-CPU state:

- Per-CPU kernel stack: As multiple CPUs can trap into the kernel simultaneously, a separate kernel stack is needed for each CPU;
- Per-CPU TSS and TSS descriptor: due to the fact that per-CPU kernel stack exists;
- Per-CPU current environment pointer;
- Per-CPU system registers. All registers are private to a CPU, so instructions to initialize the registers, like `lcr3`, `lgdt`, `lidt` must be excuted once on each CPU. 

Definition in JOS:

```c
struct CpuInfo {
	uint8_t cpu_id;                 // Local APIC ID; index into cpus[] array
	volatile unsigned cpu_status;   // The status of the CPU
	struct Env *cpu_env;            // The currently-running environment.
	struct Taskstate cpu_ts;        // TSS Used by x86 to find stack for interrupt
};
```

Locking is required to avoid race conditions. JOS uses a *big kernel lock*. 



## Cooperative Multitasking

The kernel provides the `sys_yield` system call for user-mode program. `Sys_yield` calls `sched_yeild`, which searches another runnable env in `envs`, and call `env_run`.



## Environment Creation: `dumbfork`

Function `dumbfork` is an inefficient implementation of `fork`: it `env_alloc` an environment (set up the kernel part of the address space), and copies the parent's register set into the child environment; then it calls `duppage` to copy user part of the address space into the child; then it marks child as `ENV_RUNNABLE`.



## Copy-on-Write `fork`

Naive `fork` copies the entire address space, which is inefficient. Later versions of Unix used virtual memory hardware to allow the parent and child to *share* the memory mapped into their own address spaces until one of the processes modifies it. This technique is known as *copy-on-write*. To do this, on `fork()` the kernel would copy the address space *mappings* from the parent to the child instead of the contents of the mapped pages, and mark the now-shared pages read-only. When one of the two processes tries to write to one of these shared pages, the process takes a page fault. At this point, the Unix kernel realizes that the page was really a "copy-on-write" copy, and so it makes a new, private, writable copy of the page for the faulting process. In this way, the contents of individual pages aren't actually copied until they are actually written to. This optimization makes a `fork()` followed by an `exec()` in the child much cheaper: the child will probably only need to copy one page (the current page of its stack) before it calls `exec()`.

Copy-on-write `fork` is only one of many uses for handling user-level page fault. The kernel can decide what actions to take when getting a user-level page fault in different memory region. For example, A fault in the stack region will allocate and map new page of physical memory. A fault in the `.bss` region will typically allocate a new page, fill it with zeroes, and map it. In systems with demand-paged executables, a fault in the text region will read a page of the binary off of disk and then map it.

The exact implementation in JOS is quite complicated (JOS implements user-level `fork`, so page fault handler is implemented in the user code and registered into kernel; the user-level handler needs to consule the page table, so UVPT trick must be used; user stack and user exception stack are needed), but here is the idea for `fork` implementation. 

- Call `env_create` to create a new env. This copies the mapping for the kernel part of the address space;
- Copy the register set of the parent into the child;
- For each writable or copy-on-write page in the user part of the address space, map the page as copy-on-write into the child's address space. Note this only copies the mapping, not the physical content;
- Mark the child runnable

When a page fault happens, the page fault handler does the following:

- Check that the fault is caused by a write AND the page is marked as `COW`. Otherwise, kill the user process.
- Allocate a new page mapped at a temporay location and copy the contents of the faulting page into it. Then map the new page at the appropritate address with read/write permission, in place of theold read-only mapping.



## Preemptive Multitasking

To *preempt* a running environment, the kernel forcelly retakes control of the CPU by external hardware interrupt from the clock hardware. The user environments must run with interrupts enabled. At boot, set up periodic clock interrupts. In this way, clock interrupts periodically preempts the running user environment and allow the kernel to take control. 



## Inter-Process Communication

The "messages" that user environments can send to each other using JOS's IPC mechanism consist of two components: a single 32-bit value, and optionally a single page mapping. Allowing environments to pass page mappings in messages provides an efficient way to transfer more data than will fit into a single 32-bit integer, and also allows environments to set up shared memory arrangements easily. This is implemented by two system calls: `sys_ipc_recv` and `sys_ipc_try_send`.

### Sending and Receiving Messages

To receive a message, an environment calls `sys_ipc_recv`. This system call de-schedules the current environment and does not run it again until a message has been received. When an environment is waiting to receive a message, any other environment can send it a message - not just a particular environment, and not just environments that have a parent/child relationship with the receiving environment. 

To try to send a value, an environment calls `sys_ipc_try_send`. If the named environment is actually receiving (it has called `sys_ipc_recv` and not gotten a value yet), then the send delivers the message and returns 0. Otherwise the send returns `-E_IPC_NOT_RECV` to indicate that the target environment is not currently expecting to receive a value.

A library function `ipc_recv` in user space will take care of calling `sys_ipc_recv` and then looking up the information about the received values in the current environment's `struct Env`. Similarly, a library function `ipc_send` will take care of repeatedly calling `sys_ipc_try_send` until the send succeeds.

### Transferring Pages

When an environment calls `sys_ipc_recv` with a valid `dstva` parameter (below UTOP), the environment is stating that it is willing to receive a page mapping. If the sender sends a page, then that page should be mapped at `dstva` in the receiver's address space. If the receiver already had a page mapped at `dstva`, then that previous page is unmapped.

When an environment calls `sys_ipc_try_send` with a valid `srcva` (below UTOP), it means the sender wants to send the page currently mapped at `srcva` to the receiver, with permissions `perm`. After a successful IPC, the sender keeps its original mapping for the page at `srcva` in its address space, but the receiver also obtains a **mapping** for this same physical page at the `dstva` originally specified by the receiver, in the receiver's address space. As a result this page becomes **shared** between the sender and receiver.

If either the sender or the receiver does not indicate that a page should be transferred, then no page is transferred. After any IPC the kernel sets the new field `env_ipc_perm` in the receiver's Env structure to the permissions of the page received, or zero if no page was received.