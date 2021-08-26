## Walkthrough of Zephyr 1.9.2's Initialization Code

### Source Code Commentary

#### \_\_start (arch/x86/core/crt0.S:51)

```txt
59-61: Stores boot time values in %esi and %edi.

82-84: Loads the read-only GDT (crt0.S:428) and IDT (crt0.S:357)

91-96: Sets the stack segment and data segments to DATA_SEG = 0x10.

108-109: Assigns boot time values to __start_time_stamp

176-183: Fills the range
         (_interrupt_stack+4096, _interrupt_stack+4096+CONFIG_ISR_STACK_SIZE)
         with 0xAAAAAAAA (0xA = 1010b).

186-191: Points %esp to _interrupt_stack + CONFIG_ISR_STACK_SIZE + 4096,
         which is the top of the init stack.

219-230: Copies the kernel data section from ROM to RAM for eXecute-In-Place
         (XIP) systems. https://en.wikipedia.org/wiki/Execute_in_place

241-248: Clears the BSS for default and XIP systems.

258-270: Loads page tables and enables paging for CONFIG_X86_MMU. A script
         called scripts/gen_mmu.py is used to create an object file called
         mmu_tables.o which contains the page tables.

280: Jumps to the C code portion of kernel initialization.
```

#### \_Cstart (kernel/init.c:349)

```txt

```

#### prepare\_multithreading (kernel/init.c:249)

```txt
_Cstart
    prepare_multithreading <-- Here

262-265: Assigns _current to the dummy_thread argument and sets it as
         an essential thread.

279: No-op for IA-32 kernels.

283-285: Initializes the doubly linked lists for each ready queue, where
         there exists a ready queue for each priority level (32 in total
         by default).

296: Assigns the main thread to the ready_q cache since it must run first.

298-302: Creates the main thread _main, marks it as ready, and adds it to
         the priority level 0 ready queue.

305-309: Creates the idle thread _idle_thread , marks it as ready, and adds
         it to the priority level -1 ready queue.

312:  Initializes the doubly  linked list for the timeout queue _timeout_q.
```

#### \_new\_thread (arch/x86/core/thread.c:199)

```txt
_Cstart
    prepare_multithreading
        _new_thread <-- Here

212-213: Sets the 4KiB guard page in the thread's stack to
         MMU_ENTRY_NOT_PRESENT.

216: Initializes the thread's stack, state, priority, and timeout
     queue.

230-233: Assigns the thread's arguments, entry point, and EFLAGS to
         the thread stack.
```

#### \_new\_thread\_init (kernel/include/kernel\_structs.h:149)

```txt
_Cstart
    prepare_multithreading
        _new_thread
            _new_thread_init <-- Here

159: Initializes the stack values with 0xAA for CONFIG_INIT_STACKS.

166: Assigns the stack sentinel STACK_SENTINEL to the bottom four bytes
     of the stack.

172-173: Initializes the thread's init_data nd fn_abort fields to NULL.

181-182: Assigns the thread's stack_info fields.
```

#### \_init\_thread\_base (kernel/thread.c:452)

```
_Cstart
    prepare_multithreading
        _new_thread
            _new_thread_init
                _init_thread_base <-- Here

458: Initializes the thread's state to _THREAD_PRESTART.

460: Initializes the thread's priority.

462: Initializes sched_locked to 0.
```

#### \_init\_timeout (kernel/include/timeout\_q.h:26)

```txt
_Cstart
    prepare_multithreading
        _new_thread
            _new_thread_init
                _init_thread_base
                    _init_thread_timeout
                        _init_timeout <-- Here

33: Initializes the timeout queue's delta_ticks_from_prev to _INACTIVE.

39-45: Initializes the timeout queue's thread and wait_q to NULL.

50: Initialies the timeout queue's handler function.
```

#### \_new\_thread\_internal (arch/x86/core/thread.c:51)

```txt
_Cstart
    prepare_multithreading
        _new_thread
            _new_thread_init
            _new_thread_internal <-- Here

I will do this later.
```

#### kernel\_arch\_init (arch/x86/include/kernel\_arch\_func.h:35)

```txt
_Cstart
    prepare_multithreading
        _new_thread
        kernel_arch_init <-- Here

37: Initializes _kernel.nested to zero.

38-39: Assigns _kernel.irq_stack to _interrupt_stack + CONFIG_ISR_STACK_SIZE.

41-42: Sets the 4KiB guard page in the interrupt stack to MMU_ENTRY_NOT_PRESENT.
```

#### \_sys\_device\_do\_config\_level (kernel/device.c:46)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level <-- Here


54: Calls the device's init() method for the given system initialization
    level.
```

#### sys\_rand32\_get (drivers/random/rand32\_timer.c:43)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get <-- Here

I will do this later.
```

#### switch\_to\_main\_thread (kernel/init.c:319)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread <-- Here


322: No-op for x86 machines.
```

#### \_Swap (kernel/include/nano\_internal.h:64)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap <-- Here

I will do this later.
```

#### \_\_swap (arch/x86/core/swap.S:89)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            __swap <-- Here

I will do this later.
```

#### \_main (kernel/init.c:189)

```txt
I will do this later.
````
