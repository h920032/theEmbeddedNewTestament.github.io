# ARM64 核心例外模式的運作方式
### ARM aarch32 的例外向量表
![Exception_Vectors.jpg](http://kcchao.wdfiles.com/local--files/arm/Exception_Vectors.jpg)


### ARM aarch64 的例外向量表
![Exception_Handling_Guide_6.jpg?revision=6b7bc331-414b-40d4-b2c4-bb73cb2a48cf&hash=48187D388A2C7794506C4012C5FF73D089A8C331&hash=48187D388A2C7794506C4012C5FF73D089A8C331&la=en](https://developer.arm.com/-/media/Arm%20Developer%20Community/Images/Tutorial%20Guide%20Diagrams%20and%20Screenshots/Learn%20the%20Architecture/Exception%20model/Exception_Handling_Guide_6.jpg?revision=6b7bc331-414b-40d4-b2c4-bb73cb2a48cf&hash=48187D388A2C7794506C4012C5FF73D089A8C331&hash=48187D388A2C7794506C4012C5FF73D089A8C331&la=en)


### 以下原始碼來自 Linux ARM64 3.18.102。

##### 當硬體產生例外時，核心會嘗試在向量表中尋找對應的進入點。

```C
ENTRY(vectors)
	ventry	el1_sync_invalid		// 同步例外 EL1t
	ventry	el1_irq_invalid			// IRQ EL1t
	ventry	el1_fiq_invalid			// FIQ EL1t
	ventry	el1_error_invalid		// 錯誤 EL1t

	ventry	el1_sync			// 同步例外 EL1h
	ventry	el1_irq				// IRQ EL1h
	ventry	el1_fiq_invalid			// FIQ EL1h
	ventry	el1_error_invalid		// 錯誤 EL1h

	ventry	el0_sync			// 同步例外 64 位元 EL0
	ventry	el0_irq				// IRQ 64 位元 EL0
	ventry	el0_fiq_invalid			// FIQ 64 位元 EL0
	ventry	el0_error_invalid		// 錯誤 64 位元 EL0

#ifdef CONFIG_COMPAT
	ventry	el0_sync_compat			// 同步例外 32 位元 EL0
	ventry	el0_irq_compat			// IRQ 32 位元 EL0
	ventry	el0_fiq_invalid_compat		// FIQ 32 位元 EL0
	ventry	el0_error_invalid_compat	// 錯誤 32 位元 EL0
#else
	ventry	el0_sync_invalid		// 同步例外 32 位元 EL0
	ventry	el0_irq_invalid			// IRQ 32 位元 EL0
	ventry	el0_fiq_invalid			// FIQ 32 位元 EL0
	ventry	el0_error_invalid		// 錯誤 32 位元 EL0
#endif
END(vectors)
```

##### 例如，當 EL0 同步中止發生時，核心會搜尋 el0_sync 進入點並找到匹配的處理常式。

```C
/*
 * EL0 模式處理常式。
 */
	.align	6
el0_sync:
	kernel_entry 0
	mrs	x25, esr_el1			// 讀取症候群暫存器
	lsr	x24, x25, #ESR_EL1_EC_SHIFT	// 例外類別
	cmp	x24, #ESR_EL1_EC_SVC64		// 64 位元狀態下的 SVC
	b.eq	el0_svc
	cmp	x24, #ESR_EL1_EC_DABT_EL0	// EL0 中的資料中止
	b.eq	el0_da
	cmp	x24, #ESR_EL1_EC_IABT_EL0	// EL0 中的指令中止
	b.eq	el0_ia
	cmp	x24, #ESR_EL1_EC_FP_ASIMD	// FP/ASIMD 存取
	b.eq	el0_fpsimd_acc
	cmp	x24, #ESR_EL1_EC_FP_EXC64	// FP/ASIMD 例外
	b.eq	el0_fpsimd_exc
	cmp	x24, #ESR_EL1_EC_SYS64		// 可配置陷阱
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_SP_ALIGN	// 堆疊對齊例外
	b.eq	el0_sp_pc
	cmp	x24, #ESR_EL1_EC_PC_ALIGN	// PC 對齊例外
	b.eq	el0_sp_pc
	cmp	x24, #ESR_EL1_EC_UNKNOWN	// EL0 中的未知例外
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_BREAKPT_EL0	// EL0 中的除錯例外
	b.ge	el0_dbg
	b	el0_inv

#ifdef CONFIG_COMPAT
	.align	6
el0_sync_compat:
	kernel_entry 0, 32
	mrs	x25, esr_el1			// 讀取症候群暫存器
	lsr	x24, x25, #ESR_EL1_EC_SHIFT	// 例外類別
	cmp	x24, #ESR_EL1_EC_SVC32		// 32 位元狀態下的 SVC
	b.eq	el0_svc_compat
	cmp	x24, #ESR_EL1_EC_DABT_EL0	// EL0 中的資料中止
	b.eq	el0_da
	cmp	x24, #ESR_EL1_EC_IABT_EL0	// EL0 中的指令中止
	b.eq	el0_ia
	cmp	x24, #ESR_EL1_EC_FP_ASIMD	// FP/ASIMD 存取
	b.eq	el0_fpsimd_acc
	cmp	x24, #ESR_EL1_EC_FP_EXC32	// FP/ASIMD 例外
	b.eq	el0_fpsimd_exc
	cmp	x24, #ESR_EL1_EC_UNKNOWN	// EL0 中的未知例外
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_CP15_32	// CP15 MRC/MCR 陷阱
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_CP15_64	// CP15 MRRC/MCRR 陷阱
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_CP14_MR	// CP14 MRC/MCR 陷阱
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_CP14_LS	// CP14 LDC/STC 陷阱
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_CP14_64	// CP14 MRRC/MCRR 陷阱
	b.eq	el0_undef
	cmp	x24, #ESR_EL1_EC_BREAKPT_EL0	// EL0 中的除錯例外
	b.ge	el0_dbg
	b	el0_inv
el0_svc_compat:
	/*
	 * AArch32 系統呼叫處理
	 */
	adr	stbl, compat_sys_call_table	// 載入相容系統呼叫表指標
	uxtw	scno, w7			// 系統呼叫編號在 w7 (r7) 中
	mov     sc_nr, #__NR_compat_syscalls
	b	el0_svc_naked

	.align	6
el0_irq_compat:
	kernel_entry 0, 32
	b	el0_irq_naked
#endif

el0_da:
	/*
	 * 資料中止處理
	 */
	mrs	x26, far_el1
	// 呼叫主處理常式前啟用中斷
	enable_dbg_and_irq
	ct_user_exit
	clear_address_tag x0, x26
	mov	x1, x25
	mov	x2, sp
	adr	lr, ret_to_user
	b	do_mem_abort
el0_ia:
	/*
	 * 指令中止處理
	 */
	mrs	x26, far_el1
	// 呼叫主處理常式前啟用中斷
	enable_dbg_and_irq
	ct_user_exit
	mov	x0, x26
	orr	x1, x25, #1 << 24		// 使用保留的 ISS 位元用於指令中止
	mov	x2, sp
	adr	lr, ret_to_user
	b	do_mem_abort
el0_fpsimd_acc:
	/*
	 * 浮點運算或進階 SIMD 存取
	 */
	enable_dbg
	ct_user_exit
	mov	x0, x25
	mov	x1, sp
	adr	lr, ret_to_user
	b	do_fpsimd_acc
el0_fpsimd_exc:
	/*
	 * 浮點運算或進階 SIMD 例外
	 */
	enable_dbg
	ct_user_exit
	mov	x0, x25
	mov	x1, sp
	adr	lr, ret_to_user
	b	do_fpsimd_exc
el0_sp_pc:
	/*
	 * 堆疊或 PC 對齊例外處理
	 */
	mrs	x26, far_el1
	// 呼叫主處理常式前啟用中斷
	enable_dbg_and_irq
	ct_user_exit
	mov	x0, x26
	mov	x1, x25
	mov	x2, sp
	adr	lr, ret_to_user
	b	do_sp_pc_abort
el0_undef:
	/*
	 * 未定義指令
	 */
	// 呼叫主處理常式前啟用中斷
	enable_dbg_and_irq
	ct_user_exit
	mov	x0, sp
	adr	lr, ret_to_user
	b	do_undefinstr
el0_dbg:
	/*
	 * 除錯例外處理
	 */
	tbnz	x24, #0, el0_inv		// 僅限 EL0
	mrs	x0, far_el1
	mov	x1, x25
	mov	x2, sp
	bl	do_debug_exception
	enable_dbg
	ct_user_exit
	b	ret_to_user
el0_inv:
	enable_dbg
	ct_user_exit
	mov	x0, sp
	mov	x1, #BAD_SYNC
	mrs	x2, esr_el1
	bl	bad_el0_sync
	b	ret_to_user
ENDPROC(el0_sync)
```

##### 例如，假設資料中止被觸發。核心找到資料中止處理常式並呼叫 do_mem_abort 函式。LR 暫存器被寫入 ret_to_user 函式的位址。因此當 do_mem_abort 函式完成後，ret_to_user 函式會被呼叫。
```C
el0_da:
	/*
	 * 資料中止處理
	 */
	mrs	x26, far_el1
	// 呼叫主處理常式前啟用中斷
	enable_dbg_and_irq
	ct_user_exit
	clear_address_tag x0, x26
	mov	x1, x25
	mov	x2, sp
	adr	lr, ret_to_user
	b	do_mem_abort
```


##### 對於資料中止，有各種錯誤處理函式定義在 struct fault_info 中。核心可以根據 struct fault_info 的位址加上 ESR 暫存器的偏移量來呼叫對應的函式。


```C
/*
 * 將資料中止分派到相關的處理常式。
 */
asmlinkage void __exception do_mem_abort(unsigned long addr, unsigned int esr,
					 struct pt_regs *regs)
{
	const struct fault_info *inf = fault_info + (esr & 63);
	struct siginfo info;

	if (!inf->fn(addr, esr, regs))
		return;

	pr_alert("Unhandled fault: %s (0x%08x) at 0x%016lx/n",
		 inf->name, esr, addr);

	info.si_signo = inf->sig;
	info.si_errno = 0;
	info.si_code  = inf->code;
	info.si_addr  = (void __user *)addr;
	arm64_notify_die("", regs, &info, esr);
}
```

```C
static struct fault_info {
	int	(*fn)(unsigned long addr, unsigned int esr, struct pt_regs *regs);
	int	sig;
	int	code;
	const char *name;
} fault_info[] = {
	{ do_bad,		SIGBUS,  0,		"ttbr address size fault"	},
	{ do_bad,		SIGBUS,  0,		"level 1 address size fault"	},
	{ do_bad,		SIGBUS,  0,		"level 2 address size fault"	},
	{ do_bad,		SIGBUS,  0,		"level 3 address size fault"	},
	{ do_translation_fault,	SIGSEGV, SEGV_MAPERR,	"input address range fault"	},
	{ do_translation_fault,	SIGSEGV, SEGV_MAPERR,	"level 1 translation fault"	},
	{ do_translation_fault,	SIGSEGV, SEGV_MAPERR,	"level 2 translation fault"	},
	{ do_page_fault,	SIGSEGV, SEGV_MAPERR,	"level 3 translation fault"	},
	{ do_bad,		SIGBUS,  0,		"reserved access flag fault"	},
	{ do_page_fault,	SIGSEGV, SEGV_ACCERR,	"level 1 access flag fault"	},
	{ do_page_fault,	SIGSEGV, SEGV_ACCERR,	"level 2 access flag fault"	},
	{ do_page_fault,	SIGSEGV, SEGV_ACCERR,	"level 3 access flag fault"	},
	{ do_bad,		SIGBUS,  0,		"reserved permission fault"	},
	{ do_page_fault,	SIGSEGV, SEGV_ACCERR,	"level 1 permission fault"	},
	{ do_page_fault,	SIGSEGV, SEGV_ACCERR,	"level 2 permission fault"	},
	{ do_page_fault,	SIGSEGV, SEGV_ACCERR,	"level 3 permission fault"	},
	{ do_bad,		SIGBUS,  0,		"synchronous external abort"	},
	{ do_bad,		SIGBUS,  0,		"asynchronous external abort"	},
	{ do_bad,		SIGBUS,  0,		"unknown 18"			},
	{ do_bad,		SIGBUS,  0,		"unknown 19"			},
	{ do_bad,		SIGBUS,  0,		"synchronous abort (translation table walk)" },
	{ do_bad,		SIGBUS,  0,		"synchronous abort (translation table walk)" },
	{ do_bad,		SIGBUS,  0,		"synchronous abort (translation table walk)" },
	{ do_bad,		SIGBUS,  0,		"synchronous abort (translation table walk)" },
	{ do_bad,		SIGBUS,  0,		"synchronous parity error"	},
	{ do_bad,		SIGBUS,  0,		"asynchronous parity error"	},
	{ do_bad,		SIGBUS,  0,		"unknown 26"			},
	{ do_bad,		SIGBUS,  0,		"unknown 27"			},
	{ do_bad,		SIGBUS,  0,		"synchronous parity error (translation table walk" },
	{ do_bad,		SIGBUS,  0,		"synchronous parity error (translation table walk" },
	{ do_bad,		SIGBUS,  0,		"synchronous parity error (translation table walk" },
	{ do_bad,		SIGBUS,  0,		"synchronous parity error (translation table walk" },
	{ do_bad,		SIGBUS,  0,		"unknown 32"			},
	{ do_bad,		SIGBUS,  BUS_ADRALN,	"alignment fault"		},
	{ do_bad,		SIGBUS,  0,		"debug event"			},
	{ do_bad,		SIGBUS,  0,		"unknown 35"			},
	{ do_bad,		SIGBUS,  0,		"unknown 36"			},
	{ do_bad,		SIGBUS,  0,		"unknown 37"			},
	{ do_bad,		SIGBUS,  0,		"unknown 38"			},
	{ do_bad,		SIGBUS,  0,		"unknown 39"			},
	{ do_bad,		SIGBUS,  0,		"unknown 40"			},
	{ do_bad,		SIGBUS,  0,		"unknown 41"			},
	{ do_bad,		SIGBUS,  0,		"unknown 42"			},
	{ do_bad,		SIGBUS,  0,		"unknown 43"			},
	{ do_bad,		SIGBUS,  0,		"unknown 44"			},
	{ do_bad,		SIGBUS,  0,		"unknown 45"			},
	{ do_bad,		SIGBUS,  0,		"unknown 46"			},
	{ do_bad,		SIGBUS,  0,		"unknown 47"			},
	{ do_bad,		SIGBUS,  0,		"unknown 48"			},
	{ do_bad,		SIGBUS,  0,		"unknown 49"			},
	{ do_bad,		SIGBUS,  0,		"unknown 50"			},
	{ do_bad,		SIGBUS,  0,		"unknown 51"			},
	{ do_bad,		SIGBUS,  0,		"implementation fault (lockdown abort)" },
	{ do_bad,		SIGBUS,  0,		"unknown 53"			},
	{ do_bad,		SIGBUS,  0,		"unknown 54"			},
	{ do_bad,		SIGBUS,  0,		"unknown 55"			},
	{ do_bad,		SIGBUS,  0,		"unknown 56"			},
	{ do_bad,		SIGBUS,  0,		"unknown 57"			},
	{ do_bad,		SIGBUS,  0,		"implementation fault (coprocessor abort)" },
	{ do_bad,		SIGBUS,  0,		"unknown 59"			},
	{ do_bad,		SIGBUS,  0,		"unknown 60"			},
	{ do_bad,		SIGBUS,  0,		"unknown 61"			},
	{ do_bad,		SIGBUS,  0,		"unknown 62"			},
	{ do_bad,		SIGBUS,  0,		"unknown 63"			},
};

```
