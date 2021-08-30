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

50: Initialies the timeout queue's handler function to NULL.
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

#### thread\_monitor\_init (kernel/include/kernel\_structs.h:190)

```txt
_Cstart
    prepare_multithreading
        _new_thread
            _new_thread_init
            _new_thread_internal
                thread_monitor_init <-- Here

195-196: Initializes the kernel's active thread list by inserting
         the _main thread.
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

331: Disables interrupts via irq_lock and calls _Swap.
```

#### \_arch\_irq\_lock (include/arch/x86/arch.h:408)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        irq\_lock
            _arch_irq_lock <-- Here

410: Disables interrupts and assigns the current thread's eflags to
     the key local variable.

414: Returns key.
```

#### \_do\_irq\_lock (include/arch/x86/asm\_inline\_gcc.h:42)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        irq\_lock
            _arch_irq_lock
                _do_irq_lock <-- Here

46-53: Pushes eflags, clears the interrupt bit, and pops the eflags into
       the key variable.

55: Returns key.
```

#### \_int\_latency\_start (kernel/int\_latency\_bench.c:53)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        irq\_lock
            _arch_irq_lock
            _int_latency_start <-- Here

56-59: Retrieves the system time and assigns it to int_locked_timestamp.

60: Increments int_locked_unlocked_nest to 1.
```

#### \_Swap (kernel/include/nano\_internal.h:64)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap <-- Here

74: Calls __swap by passing key as an argument, where key is the
    current thread's eflags.
```

#### \_check\_stack\_sentinel (kernel/thread.c:153)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            _check_stack_sentinel <-- Here

157-162: Returns early if the stack's thread is not allowed to run
         (hence no reason to check its sentinel).

165-169: Restores the stack's sentinel if it was corrupted and generates
         the _NANO_ERR_STACK_CHK_FAIL exception.
```

#### \_update\_time\_slice\_before\_swap (kernel/sched.c:423)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            _check_stack_sentinel
            _update_time_slice_before_swap <-- Here

426-428: Returns early if the next thread is NOT time slicing.

430: Obtains the thread's remaining time slice.

432-439: Sets the time slice to MIN(remaining, _time_slice_duration).

443: Sets _time_slice_elapsed to 0 for the new thread.
```

#### \_get\_remaining\_program\_time ()

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            _check_stack_sentinel
            _update_time_slice_before_swap
                _get_remaining_program_time <-- Here

I will do later since this is hardware dependent.
```

#### \_set\_time ()

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            _check_stack_sentinel
            _update_time_slice_before_swap
                _get_remaining_program_time
                _set_time <-- Here

I will do later since this is hardware dependent.
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

94-100: Stores swap timestamp values in __start_swap_time.

118: Assigns the base of _kernel to %edi.

120-122: Pushes %esi, %ebx, and %ebp.

131: Pushes -EAGAIN.

141: Logs the context switch into the logger ring buffer.

143: Assigns the ready queue's cached process to %eax.

308: Assigns the cached process, _main, to _kernel.current.

312: Sets %esp to _main's stack pointer.

317-324: Recovers the _main's %ebp, %ebx, %esi, and %edi
         registers.

337: Pushes __swap's key argument to the stack.

343: Stops counting the time spent with interrupts disabled.

351: Restores previous thread's eflags.

371: Returns to the _main's entry point.
```

#### \_int\_latency\_stop (kernel/int\_latency\_bench.c:72)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            __swap
                _int_latency_stop <-- Here

76: Obtains the current system time.

86: Calculates the elapsed time since calling _int_latency_start and
    assigns it to the delta variable.

95-99: Decrement benchmark overhead delay from delta if necessary.

102-107: Updates int_locked_latency_max and int_locked_latency_min if
         necessary.

111: Sets int_locked_timestamp to zero.
```

#### \_sys\_k\_event\_logger\_context\_switch (subsys/logging/kernel\_event\_logger.c:78)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            __swap
                _sys_k_event_logger_context_switch <-- Here

89: Sets the event_id to KERNEL_EVENT_LOGGER_CONTEXT_SWITCH_EVENT_ID,
    which is equal to 0x001.

91-92: Returns early if the event_id doesn't require logging.

96-97: Returns early if the logger ring buffer is NULL.

100-101: Returns early if the current thread is the _collector_coop_thread.

104-105: Obtains the timestamp data for the context switch.

124-125: Adds the event message to the ring buffer and signals the
         sync semaphore to alert the kernel that there are event
         messages to read.
```

####  event\_logger\_put (subsys/logging/event\_logger.c:23)

```txt
_Cstart
    prepare_multithreading
    _sys_device_do_config_level
    sys_rand32_get
    switch_to_main_thread
        _Swap
            __swap
                _sys_k_event_logger_context_switch
                    _sys_event_logger_put_non_preemptible
                        event_logger_put <-- Here

30: Locks interrupts.

32: Writes the event_data argument to the logger ring buffer logger->ring_buf.

40: Unlocks interrupts.
```

#### \_main (kernel/init.c:189)

```txt
_Cstart
    ...
    switch_to_main_thread
_main <-- Here

195-198: Calls the device initialization functions for system configuration
         levels POST_KERNEL and APPLICATION.

213: Prints the boot banner.

226: Calls the application's main function to conclude system initialization.
```
