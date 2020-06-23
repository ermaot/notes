本文来自：https://blog.51cto.com/john88wang/815867

https://www.vpsee.com/2010/11/what-are-cpu-feature-flags-in-proc/

在平时的工作中，我们需要了解当前CPU的一些特性，以便我们能够确定当前CPU是否满足我们的需求。例如，查看CPU是32位的还是64位的，查看CPU是否支持全虚拟化或物理地址扩展功能。所有CPU的这些特性都可以通过linux下面的/proc/cpuinfo文件获取

## 查看cpu信息

```
# cat /proc/cpuinfo 
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 79
model name	: Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz
stepping	: 1
microcode	: 0x1
cpu MHz		: 2494.224
cache size	: 40960 KB
physical id	: 0
siblings	: 2
core id		: 0
cpu cores	: 1
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs ibpb stibp fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm rdseed adx smap xsaveopt spec_ctrl intel_stibp
bogomips	: 4988.44
clflush size	: 64
cache_alignment	: 64
address sizes	: 46 bits physical, 48 bits virtual
power management:

```

## flags的含义

可以看到一个很重要的部分flags，表示的是cpu能支持的一些特性。

### flag在源代码中的含义

最能说明问题的，永远是源代码。

```
vim /usr/src/kernels/5.2.11-1.el7.elrepo.x86_64/arch/x86/include/asm/cpufeatures.h

/* Intel-defined CPU features, CPUID level 0x00000001 (EDX), word 0 */
#define X86_FEATURE_FPU			( 0*32+ 0) /* Onboard FPU */
#define X86_FEATURE_VME			( 0*32+ 1) /* Virtual Mode Extensions */
#define X86_FEATURE_DE			( 0*32+ 2) /* Debugging Extensions */
#define X86_FEATURE_PSE			( 0*32+ 3) /* Page Size Extensions */
#define X86_FEATURE_TSC			( 0*32+ 4) /* Time Stamp Counter */
#define X86_FEATURE_MSR			( 0*32+ 5) /* Model-Specific Registers */
#define X86_FEATURE_PAE			( 0*32+ 6) /* Physical Address Extensions */
#define X86_FEATURE_MCE			( 0*32+ 7) /* Machine Check Exception */
#define X86_FEATURE_CX8			( 0*32+ 8) /* CMPXCHG8 instruction */
#define X86_FEATURE_APIC		( 0*32+ 9) /* Onboard APIC */
#define X86_FEATURE_SEP			( 0*32+11) /* SYSENTER/SYSEXIT */
#define X86_FEATURE_MTRR		( 0*32+12) /* Memory Type Range Registers */
#define X86_FEATURE_PGE			( 0*32+13) /* Page Global Enable */
#define X86_FEATURE_MCA			( 0*32+14) /* Machine Check Architecture */
#define X86_FEATURE_CMOV		( 0*32+15) /* CMOV instructions (plus FCMOVcc, FCOMI with FPU) */
#define X86_FEATURE_PAT			( 0*32+16) /* Page Attribute Table */
#define X86_FEATURE_PSE36		( 0*32+17) /* 36-bit PSEs */
#define X86_FEATURE_PN			( 0*32+18) /* Processor serial number */
#define X86_FEATURE_CLFLUSH		( 0*32+19) /* CLFLUSH instruction */
#define X86_FEATURE_DS			( 0*32+21) /* "dts" Debug Store */
#define X86_FEATURE_ACPI		( 0*32+22) /* ACPI via MSR */
#define X86_FEATURE_MMX			( 0*32+23) /* Multimedia Extensions */
#define X86_FEATURE_FXSR		( 0*32+24) /* FXSAVE/FXRSTOR, CR4.OSFXSR */
#define X86_FEATURE_XMM			( 0*32+25) /* "sse" */
#define X86_FEATURE_XMM2		( 0*32+26) /* "sse2" */
#define X86_FEATURE_SELFSNOOP		( 0*32+27) /* "ss" CPU self snoop */
#define X86_FEATURE_HT			( 0*32+28) /* Hyper-Threading */
#define X86_FEATURE_ACC			( 0*32+29) /* "tm" Automatic clock control */
#define X86_FEATURE_IA64		( 0*32+30) /* IA-64 processor */
#define X86_FEATURE_PBE			( 0*32+31) /* Pending Break Enable */

/* AMD-defined CPU features, CPUID level 0x80000001, word 1 */
/* Don't duplicate feature flags which are redundant with Intel! */
#define X86_FEATURE_SYSCALL		( 1*32+11) /* SYSCALL/SYSRET */
#define X86_FEATURE_MP			( 1*32+19) /* MP Capable */
#define X86_FEATURE_NX			( 1*32+20) /* Execute Disable */
#define X86_FEATURE_MMXEXT		( 1*32+22) /* AMD MMX extensions */
#define X86_FEATURE_FXSR_OPT		( 1*32+25) /* FXSAVE/FXRSTOR optimizations */
#define X86_FEATURE_GBPAGES		( 1*32+26) /* "pdpe1gb" GB pages */
#define X86_FEATURE_RDTSCP		( 1*32+27) /* RDTSCP */
#define X86_FEATURE_LM			( 1*32+29) /* Long Mode (x86-64, 64-bit support) */
#define X86_FEATURE_3DNOWEXT		( 1*32+30) /* AMD 3DNow extensions */
#define X86_FEATURE_3DNOW		( 1*32+31) /* 3DNow */

/* Transmeta-defined CPU features, CPUID level 0x80860001, word 2 */
#define X86_FEATURE_RECOVERY		( 2*32+ 0) /* CPU in recovery mode */
#define X86_FEATURE_LONGRUN		( 2*32+ 1) /* Longrun power control */
#define X86_FEATURE_LRTI		( 2*32+ 3) /* LongRun table interface */

/* Other features, Linux-defined mapping, word 3 */
/* This range is used for feature bits which conflict or are synthesized */
#define X86_FEATURE_CXMMX		( 3*32+ 0) /* Cyrix MMX extensions */
#define X86_FEATURE_K6_MTRR		( 3*32+ 1) /* AMD K6 nonstandard MTRRs */
#define X86_FEATURE_CYRIX_ARR		( 3*32+ 2) /* Cyrix ARRs (= MTRRs) */
#define X86_FEATURE_CENTAUR_MCR		( 3*32+ 3) /* Centaur MCRs (= MTRRs) */

/* CPU types for specific tunings: */
#define X86_FEATURE_K8			( 3*32+ 4) /* "" Opteron, Athlon64 */
#define X86_FEATURE_K7			( 3*32+ 5) /* "" Athlon */
#define X86_FEATURE_P3			( 3*32+ 6) /* "" P3 */
#define X86_FEATURE_P4			( 3*32+ 7) /* "" P4 */
#define X86_FEATURE_CONSTANT_TSC	( 3*32+ 8) /* TSC ticks at a constant rate */
#define X86_FEATURE_UP			( 3*32+ 9) /* SMP kernel running on UP */
#define X86_FEATURE_ART			( 3*32+10) /* Always running timer (ART) */
#define X86_FEATURE_ARCH_PERFMON	( 3*32+11) /* Intel Architectural PerfMon */
#define X86_FEATURE_PEBS		( 3*32+12) /* Precise-Event Based Sampling */
#define X86_FEATURE_BTS			( 3*32+13) /* Branch Trace Store */
#define X86_FEATURE_SYSCALL32		( 3*32+14) /* "" syscall in IA32 userspace */
#define X86_FEATURE_SYSENTER32		( 3*32+15) /* "" sysenter in IA32 userspace */
#define X86_FEATURE_REP_GOOD		( 3*32+16) /* REP microcode works well */
#define X86_FEATURE_MFENCE_RDTSC	( 3*32+17) /* "" MFENCE synchronizes RDTSC */
#define X86_FEATURE_LFENCE_RDTSC	( 3*32+18) /* "" LFENCE synchronizes RDTSC */
#define X86_FEATURE_ACC_POWER		( 3*32+19) /* AMD Accumulated Power Mechanism */
#define X86_FEATURE_NOPL		( 3*32+20) /* The NOPL (0F 1F) instructions */
#define X86_FEATURE_ALWAYS		( 3*32+21) /* "" Always-present feature */
#define X86_FEATURE_XTOPOLOGY		( 3*32+22) /* CPU topology enum extensions */
#define X86_FEATURE_TSC_RELIABLE	( 3*32+23) /* TSC is known to be reliable */
#define X86_FEATURE_NONSTOP_TSC		( 3*32+24) /* TSC does not stop in C states */
#define X86_FEATURE_CPUID		( 3*32+25) /* CPU has CPUID instruction itself */
#define X86_FEATURE_EXTD_APICID		( 3*32+26) /* Extended APICID (8 bits) */
#define X86_FEATURE_AMD_DCM		( 3*32+27) /* AMD multi-node processor */
#define X86_FEATURE_APERFMPERF		( 3*32+28) /* P-State hardware coordination feedback capability (APERF/MPERF MSRs) */
#define X86_FEATURE_NONSTOP_TSC_S3	( 3*32+30) /* TSC doesn't stop in S3 state */
#define X86_FEATURE_TSC_KNOWN_FREQ	( 3*32+31) /* TSC has known frequency */

/* Intel-defined CPU features, CPUID level 0x00000001 (ECX), word 4 */
#define X86_FEATURE_XMM3		( 4*32+ 0) /* "pni" SSE-3 */
#define X86_FEATURE_PCLMULQDQ		( 4*32+ 1) /* PCLMULQDQ instruction */
#define X86_FEATURE_DTES64		( 4*32+ 2) /* 64-bit Debug Store */
#define X86_FEATURE_MWAIT		( 4*32+ 3) /* "monitor" MONITOR/MWAIT support */
#define X86_FEATURE_DSCPL		( 4*32+ 4) /* "ds_cpl" CPL-qualified (filtered) Debug Store */
#define X86_FEATURE_VMX			( 4*32+ 5) /* Hardware virtualization */
#define X86_FEATURE_SMX			( 4*32+ 6) /* Safer Mode eXtensions */
#define X86_FEATURE_EST			( 4*32+ 7) /* Enhanced SpeedStep */
#define X86_FEATURE_TM2			( 4*32+ 8) /* Thermal Monitor 2 */
#define X86_FEATURE_SSSE3		( 4*32+ 9) /* Supplemental SSE-3 */
#define X86_FEATURE_CID			( 4*32+10) /* Context ID */
#define X86_FEATURE_SDBG		( 4*32+11) /* Silicon Debug */
#define X86_FEATURE_FMA			( 4*32+12) /* Fused multiply-add */
#define X86_FEATURE_CX16		( 4*32+13) /* CMPXCHG16B instruction */
#define X86_FEATURE_XTPR		( 4*32+14) /* Send Task Priority Messages */
#define X86_FEATURE_PDCM		( 4*32+15) /* Perf/Debug Capabilities MSR */
#define X86_FEATURE_PCID		( 4*32+17) /* Process Context Identifiers */
#define X86_FEATURE_DCA			( 4*32+18) /* Direct Cache Access */
#define X86_FEATURE_XMM4_1		( 4*32+19) /* "sse4_1" SSE-4.1 */
#define X86_FEATURE_XMM4_2		( 4*32+20) /* "sse4_2" SSE-4.2 */
#define X86_FEATURE_X2APIC		( 4*32+21) /* X2APIC */
#define X86_FEATURE_MOVBE		( 4*32+22) /* MOVBE instruction */
#define X86_FEATURE_POPCNT		( 4*32+23) /* POPCNT instruction */
#define X86_FEATURE_TSC_DEADLINE_TIMER	( 4*32+24) /* TSC deadline timer */
#define X86_FEATURE_AES			( 4*32+25) /* AES instructions */
#define X86_FEATURE_XSAVE		( 4*32+26) /* XSAVE/XRSTOR/XSETBV/XGETBV instructions */
#define X86_FEATURE_OSXSAVE		( 4*32+27) /* "" XSAVE instruction enabled in the OS */
#define X86_FEATURE_AVX			( 4*32+28) /* Advanced Vector Extensions */
#define X86_FEATURE_F16C		( 4*32+29) /* 16-bit FP conversions */
#define X86_FEATURE_RDRAND		( 4*32+30) /* RDRAND instruction */
#define X86_FEATURE_HYPERVISOR		( 4*32+31) /* Running on a hypervisor */

/* VIA/Cyrix/Centaur-defined CPU features, CPUID level 0xC0000001, word 5 */
#define X86_FEATURE_XSTORE		( 5*32+ 2) /* "rng" RNG present (xstore) */
#define X86_FEATURE_XSTORE_EN		( 5*32+ 3) /* "rng_en" RNG enabled */
#define X86_FEATURE_XCRYPT		( 5*32+ 6) /* "ace" on-CPU crypto (xcrypt) */
#define X86_FEATURE_XCRYPT_EN		( 5*32+ 7) /* "ace_en" on-CPU crypto enabled */
#define X86_FEATURE_ACE2		( 5*32+ 8) /* Advanced Cryptography Engine v2 */
#define X86_FEATURE_ACE2_EN		( 5*32+ 9) /* ACE v2 enabled */
#define X86_FEATURE_PHE			( 5*32+10) /* PadLock Hash Engine */
#define X86_FEATURE_PHE_EN		( 5*32+11) /* PHE enabled */
#define X86_FEATURE_PMM			( 5*32+12) /* PadLock Montgomery Multiplier */
#define X86_FEATURE_PMM_EN		( 5*32+13) /* PMM enabled */

/* More extended AMD flags: CPUID level 0x80000001, ECX, word 6 */
#define X86_FEATURE_LAHF_LM		( 6*32+ 0) /* LAHF/SAHF in long mode */
#define X86_FEATURE_CMP_LEGACY		( 6*32+ 1) /* If yes HyperThreading not valid */
#define X86_FEATURE_SVM			( 6*32+ 2) /* Secure Virtual Machine */
#define X86_FEATURE_EXTAPIC		( 6*32+ 3) /* Extended APIC space */
#define X86_FEATURE_CR8_LEGACY		( 6*32+ 4) /* CR8 in 32-bit mode */
#define X86_FEATURE_ABM			( 6*32+ 5) /* Advanced bit manipulation */
#define X86_FEATURE_SSE4A		( 6*32+ 6) /* SSE-4A */
#define X86_FEATURE_MISALIGNSSE		( 6*32+ 7) /* Misaligned SSE mode */
#define X86_FEATURE_3DNOWPREFETCH	( 6*32+ 8) /* 3DNow prefetch instructions */
#define X86_FEATURE_OSVW		( 6*32+ 9) /* OS Visible Workaround */
#define X86_FEATURE_IBS			( 6*32+10) /* Instruction Based Sampling */
#define X86_FEATURE_XOP			( 6*32+11) /* extended AVX instructions */
#define X86_FEATURE_SKINIT		( 6*32+12) /* SKINIT/STGI instructions */
#define X86_FEATURE_WDT			( 6*32+13) /* Watchdog timer */
#define X86_FEATURE_LWP			( 6*32+15) /* Light Weight Profiling */
#define X86_FEATURE_FMA4		( 6*32+16) /* 4 operands MAC instructions */
#define X86_FEATURE_TCE			( 6*32+17) /* Translation Cache Extension */
#define X86_FEATURE_NODEID_MSR		( 6*32+19) /* NodeId MSR */
#define X86_FEATURE_TBM			( 6*32+21) /* Trailing Bit Manipulations */
#define X86_FEATURE_TOPOEXT		( 6*32+22) /* Topology extensions CPUID leafs */
#define X86_FEATURE_PERFCTR_CORE	( 6*32+23) /* Core performance counter extensions */
#define X86_FEATURE_PERFCTR_NB		( 6*32+24) /* NB performance counter extensions */
#define X86_FEATURE_BPEXT		( 6*32+26) /* Data breakpoint extension */
#define X86_FEATURE_PTSC		( 6*32+27) /* Performance time-stamp counter */
#define X86_FEATURE_PERFCTR_LLC		( 6*32+28) /* Last Level Cache performance counter extensions */
#define X86_FEATURE_MWAITX		( 6*32+29) /* MWAIT extension (MONITORX/MWAITX instructions) */

/*
 * Auxiliary flags: Linux defined - For features scattered in various
 * CPUID levels like 0x6, 0xA etc, word 7.
 *
 * Reuse free bits when adding new feature flags!
 */
#define X86_FEATURE_RING3MWAIT		( 7*32+ 0) /* Ring 3 MONITOR/MWAIT instructions */
#define X86_FEATURE_CPUID_FAULT		( 7*32+ 1) /* Intel CPUID faulting */
#define X86_FEATURE_CPB			( 7*32+ 2) /* AMD Core Performance Boost */
#define X86_FEATURE_EPB			( 7*32+ 3) /* IA32_ENERGY_PERF_BIAS support */
#define X86_FEATURE_CAT_L3		( 7*32+ 4) /* Cache Allocation Technology L3 */
#define X86_FEATURE_CAT_L2		( 7*32+ 5) /* Cache Allocation Technology L2 */
#define X86_FEATURE_CDP_L3		( 7*32+ 6) /* Code and Data Prioritization L3 */
#define X86_FEATURE_INVPCID_SINGLE	( 7*32+ 7) /* Effectively INVPCID && CR4.PCIDE=1 */
#define X86_FEATURE_HW_PSTATE		( 7*32+ 8) /* AMD HW-PState */
#define X86_FEATURE_PROC_FEEDBACK	( 7*32+ 9) /* AMD ProcFeedbackInterface */
#define X86_FEATURE_SME			( 7*32+10) /* AMD Secure Memory Encryption */
#define X86_FEATURE_PTI			( 7*32+11) /* Kernel Page Table Isolation enabled */
#define X86_FEATURE_RETPOLINE		( 7*32+12) /* "" Generic Retpoline mitigation for Spectre variant 2 */
#define X86_FEATURE_RETPOLINE_AMD	( 7*32+13) /* "" AMD Retpoline mitigation for Spectre variant 2 */
#define X86_FEATURE_INTEL_PPIN		( 7*32+14) /* Intel Processor Inventory Number */
#define X86_FEATURE_CDP_L2		( 7*32+15) /* Code and Data Prioritization L2 */
#define X86_FEATURE_MSR_SPEC_CTRL	( 7*32+16) /* "" MSR SPEC_CTRL is implemented */
#define X86_FEATURE_SSBD		( 7*32+17) /* Speculative Store Bypass Disable */
#define X86_FEATURE_MBA			( 7*32+18) /* Memory Bandwidth Allocation */
#define X86_FEATURE_RSB_CTXSW		( 7*32+19) /* "" Fill RSB on context switches */
#define X86_FEATURE_SEV			( 7*32+20) /* AMD Secure Encrypted Virtualization */
#define X86_FEATURE_USE_IBPB		( 7*32+21) /* "" Indirect Branch Prediction Barrier enabled */
#define X86_FEATURE_USE_IBRS_FW		( 7*32+22) /* "" Use IBRS during runtime firmware calls */
#define X86_FEATURE_SPEC_STORE_BYPASS_DISABLE	( 7*32+23) /* "" Disable Speculative Store Bypass. */
#define X86_FEATURE_LS_CFG_SSBD		( 7*32+24)  /* "" AMD SSBD implementation via LS_CFG MSR */
#define X86_FEATURE_IBRS		( 7*32+25) /* Indirect Branch Restricted Speculation */
#define X86_FEATURE_IBPB		( 7*32+26) /* Indirect Branch Prediction Barrier */
#define X86_FEATURE_STIBP		( 7*32+27) /* Single Thread Indirect Branch Predictors */
#define X86_FEATURE_ZEN			( 7*32+28) /* "" CPU is AMD family 0x17 (Zen) */
#define X86_FEATURE_L1TF_PTEINV		( 7*32+29) /* "" L1TF workaround PTE inversion */
#define X86_FEATURE_IBRS_ENHANCED	( 7*32+30) /* Enhanced IBRS */

/* Virtualization flags: Linux defined, word 8 */
#define X86_FEATURE_TPR_SHADOW		( 8*32+ 0) /* Intel TPR Shadow */
#define X86_FEATURE_VNMI		( 8*32+ 1) /* Intel Virtual NMI */
#define X86_FEATURE_FLEXPRIORITY	( 8*32+ 2) /* Intel FlexPriority */
#define X86_FEATURE_EPT			( 8*32+ 3) /* Intel Extended Page Table */
#define X86_FEATURE_VPID		( 8*32+ 4) /* Intel Virtual Processor ID */

#define X86_FEATURE_VMMCALL		( 8*32+15) /* Prefer VMMCALL to VMCALL */
#define X86_FEATURE_XENPV		( 8*32+16) /* "" Xen paravirtual guest */
#define X86_FEATURE_EPT_AD		( 8*32+17) /* Intel Extended Page Table access-dirty bit */

/* Intel-defined CPU features, CPUID level 0x00000007:0 (EBX), word 9 */
#define X86_FEATURE_FSGSBASE		( 9*32+ 0) /* RDFSBASE, WRFSBASE, RDGSBASE, WRGSBASE instructions*/
#define X86_FEATURE_TSC_ADJUST		( 9*32+ 1) /* TSC adjustment MSR 0x3B */
#define X86_FEATURE_BMI1		( 9*32+ 3) /* 1st group bit manipulation extensions */
#define X86_FEATURE_HLE			( 9*32+ 4) /* Hardware Lock Elision */
#define X86_FEATURE_AVX2		( 9*32+ 5) /* AVX2 instructions */
#define X86_FEATURE_FDP_EXCPTN_ONLY	( 9*32+ 6) /* "" FPU data pointer updated only on x87 exceptions */
#define X86_FEATURE_SMEP		( 9*32+ 7) /* Supervisor Mode Execution Protection */
#define X86_FEATURE_BMI2		( 9*32+ 8) /* 2nd group bit manipulation extensions */
#define X86_FEATURE_ERMS		( 9*32+ 9) /* Enhanced REP MOVSB/STOSB instructions */
#define X86_FEATURE_INVPCID		( 9*32+10) /* Invalidate Processor Context ID */
#define X86_FEATURE_RTM			( 9*32+11) /* Restricted Transactional Memory */
#define X86_FEATURE_CQM			( 9*32+12) /* Cache QoS Monitoring */
#define X86_FEATURE_ZERO_FCS_FDS	( 9*32+13) /* "" Zero out FPU CS and FPU DS */
#define X86_FEATURE_MPX			( 9*32+14) /* Memory Protection Extension */
#define X86_FEATURE_RDT_A		( 9*32+15) /* Resource Director Technology Allocation */
#define X86_FEATURE_AVX512F		( 9*32+16) /* AVX-512 Foundation */
#define X86_FEATURE_AVX512DQ		( 9*32+17) /* AVX-512 DQ (Double/Quad granular) Instructions */
#define X86_FEATURE_RDSEED		( 9*32+18) /* RDSEED instruction */
#define X86_FEATURE_ADX			( 9*32+19) /* ADCX and ADOX instructions */
#define X86_FEATURE_SMAP		( 9*32+20) /* Supervisor Mode Access Prevention */
#define X86_FEATURE_AVX512IFMA		( 9*32+21) /* AVX-512 Integer Fused Multiply-Add instructions */
#define X86_FEATURE_CLFLUSHOPT		( 9*32+23) /* CLFLUSHOPT instruction */
#define X86_FEATURE_CLWB		( 9*32+24) /* CLWB instruction */
#define X86_FEATURE_INTEL_PT		( 9*32+25) /* Intel Processor Trace */
#define X86_FEATURE_AVX512PF		( 9*32+26) /* AVX-512 Prefetch */
#define X86_FEATURE_AVX512ER		( 9*32+27) /* AVX-512 Exponential and Reciprocal */
#define X86_FEATURE_AVX512CD		( 9*32+28) /* AVX-512 Conflict Detection */
#define X86_FEATURE_SHA_NI		( 9*32+29) /* SHA1/SHA256 Instruction Extensions */
#define X86_FEATURE_AVX512BW		( 9*32+30) /* AVX-512 BW (Byte/Word granular) Instructions */
#define X86_FEATURE_AVX512VL		( 9*32+31) /* AVX-512 VL (128/256 Vector Length) Extensions */

/* Extended state features, CPUID level 0x0000000d:1 (EAX), word 10 */
#define X86_FEATURE_XSAVEOPT		(10*32+ 0) /* XSAVEOPT instruction */
#define X86_FEATURE_XSAVEC		(10*32+ 1) /* XSAVEC instruction */
#define X86_FEATURE_XGETBV1		(10*32+ 2) /* XGETBV with ECX = 1 instruction */
#define X86_FEATURE_XSAVES		(10*32+ 3) /* XSAVES/XRSTORS instructions */

/*
 * Extended auxiliary flags: Linux defined - for features scattered in various
 * CPUID levels like 0xf, etc.
 *
 * Reuse free bits when adding new feature flags!
 */
#define X86_FEATURE_CQM_LLC		(11*32+ 0) /* LLC QoS if 1 */
#define X86_FEATURE_CQM_OCCUP_LLC	(11*32+ 1) /* LLC occupancy monitoring */
#define X86_FEATURE_CQM_MBM_TOTAL	(11*32+ 2) /* LLC Total MBM monitoring */
#define X86_FEATURE_CQM_MBM_LOCAL	(11*32+ 3) /* LLC Local MBM monitoring */
#define X86_FEATURE_FENCE_SWAPGS_USER	(11*32+ 4) /* "" LFENCE in user entry SWAPGS path */
#define X86_FEATURE_FENCE_SWAPGS_KERNEL	(11*32+ 5) /* "" LFENCE in kernel entry SWAPGS path */

/* AMD-defined CPU features, CPUID level 0x80000008 (EBX), word 13 */
#define X86_FEATURE_CLZERO		(13*32+ 0) /* CLZERO instruction */
#define X86_FEATURE_IRPERF		(13*32+ 1) /* Instructions Retired Count */
#define X86_FEATURE_XSAVEERPTR		(13*32+ 2) /* Always save/restore FP error pointers */
#define X86_FEATURE_WBNOINVD		(13*32+ 9) /* WBNOINVD instruction */
#define X86_FEATURE_AMD_IBPB		(13*32+12) /* "" Indirect Branch Prediction Barrier */
#define X86_FEATURE_AMD_IBRS		(13*32+14) /* "" Indirect Branch Restricted Speculation */
#define X86_FEATURE_AMD_STIBP		(13*32+15) /* "" Single Thread Indirect Branch Predictors */
#define X86_FEATURE_AMD_STIBP_ALWAYS_ON	(13*32+17) /* "" Single Thread Indirect Branch Predictors always-on preferred */
#define X86_FEATURE_AMD_SSBD		(13*32+24) /* "" Speculative Store Bypass Disable */
#define X86_FEATURE_VIRT_SSBD		(13*32+25) /* Virtualized Speculative Store Bypass Disable */
#define X86_FEATURE_AMD_SSB_NO		(13*32+26) /* "" Speculative Store Bypass is fixed in hardware. */

/* Thermal and Power Management Leaf, CPUID level 0x00000006 (EAX), word 14 */
#define X86_FEATURE_DTHERM		(14*32+ 0) /* Digital Thermal Sensor */
#define X86_FEATURE_IDA			(14*32+ 1) /* Intel Dynamic Acceleration */
#define X86_FEATURE_ARAT		(14*32+ 2) /* Always Running APIC Timer */
#define X86_FEATURE_PLN			(14*32+ 4) /* Intel Power Limit Notification */
#define X86_FEATURE_PTS			(14*32+ 6) /* Intel Package Thermal Status */
#define X86_FEATURE_HWP			(14*32+ 7) /* Intel Hardware P-states */
#define X86_FEATURE_HWP_NOTIFY		(14*32+ 8) /* HWP Notification */
#define X86_FEATURE_HWP_ACT_WINDOW	(14*32+ 9) /* HWP Activity Window */
#define X86_FEATURE_HWP_EPP		(14*32+10) /* HWP Energy Perf. Preference */
#define X86_FEATURE_HWP_PKG_REQ		(14*32+11) /* HWP Package Level Request */

/* AMD SVM Feature Identification, CPUID level 0x8000000a (EDX), word 15 */
#define X86_FEATURE_NPT			(15*32+ 0) /* Nested Page Table support */
#define X86_FEATURE_LBRV		(15*32+ 1) /* LBR Virtualization support */
#define X86_FEATURE_SVML		(15*32+ 2) /* "svm_lock" SVM locking MSR */
#define X86_FEATURE_NRIPS		(15*32+ 3) /* "nrip_save" SVM next_rip save */
#define X86_FEATURE_TSCRATEMSR		(15*32+ 4) /* "tsc_scale" TSC scaling support */
#define X86_FEATURE_VMCBCLEAN		(15*32+ 5) /* "vmcb_clean" VMCB clean bits support */
#define X86_FEATURE_FLUSHBYASID		(15*32+ 6) /* flush-by-ASID support */
#define X86_FEATURE_DECODEASSISTS	(15*32+ 7) /* Decode Assists support */
#define X86_FEATURE_PAUSEFILTER		(15*32+10) /* filtered pause intercept */
#define X86_FEATURE_PFTHRESHOLD		(15*32+12) /* pause filter threshold */
#define X86_FEATURE_AVIC		(15*32+13) /* Virtual Interrupt Controller */
#define X86_FEATURE_V_VMSAVE_VMLOAD	(15*32+15) /* Virtual VMSAVE VMLOAD */
#define X86_FEATURE_VGIF		(15*32+16) /* Virtual GIF */

/* Intel-defined CPU features, CPUID level 0x00000007:0 (ECX), word 16 */
#define X86_FEATURE_AVX512VBMI		(16*32+ 1) /* AVX512 Vector Bit Manipulation instructions*/
#define X86_FEATURE_UMIP		(16*32+ 2) /* User Mode Instruction Protection */
#define X86_FEATURE_PKU			(16*32+ 3) /* Protection Keys for Userspace */
#define X86_FEATURE_OSPKE		(16*32+ 4) /* OS Protection Keys Enable */
#define X86_FEATURE_AVX512_VBMI2	(16*32+ 6) /* Additional AVX512 Vector Bit Manipulation Instructions */
#define X86_FEATURE_GFNI		(16*32+ 8) /* Galois Field New Instructions */
#define X86_FEATURE_VAES		(16*32+ 9) /* Vector AES */
#define X86_FEATURE_VPCLMULQDQ		(16*32+10) /* Carry-Less Multiplication Double Quadword */
#define X86_FEATURE_AVX512_VNNI		(16*32+11) /* Vector Neural Network Instructions */
#define X86_FEATURE_AVX512_BITALG	(16*32+12) /* Support for VPOPCNT[B,W] and VPSHUF-BITQMB instructions */
#define X86_FEATURE_TME			(16*32+13) /* Intel Total Memory Encryption */
#define X86_FEATURE_AVX512_VPOPCNTDQ	(16*32+14) /* POPCNT for vectors of DW/QW */
#define X86_FEATURE_LA57		(16*32+16) /* 5-level page tables */
#define X86_FEATURE_RDPID		(16*32+22) /* RDPID instruction */
#define X86_FEATURE_CLDEMOTE		(16*32+25) /* CLDEMOTE instruction */
#define X86_FEATURE_MOVDIRI		(16*32+27) /* MOVDIRI instruction */
#define X86_FEATURE_MOVDIR64B		(16*32+28) /* MOVDIR64B instruction */

/* AMD-defined CPU features, CPUID level 0x80000007 (EBX), word 17 */
#define X86_FEATURE_OVERFLOW_RECOV	(17*32+ 0) /* MCA overflow recovery support */
#define X86_FEATURE_SUCCOR		(17*32+ 1) /* Uncorrectable error containment and recovery */
#define X86_FEATURE_SMCA		(17*32+ 3) /* Scalable MCA */

/* Intel-defined CPU features, CPUID level 0x00000007:0 (EDX), word 18 */
#define X86_FEATURE_AVX512_4VNNIW	(18*32+ 2) /* AVX-512 Neural Network Instructions */
#define X86_FEATURE_AVX512_4FMAPS	(18*32+ 3) /* AVX-512 Multiply Accumulation Single precision */
#define X86_FEATURE_MD_CLEAR		(18*32+10) /* VERW clears CPU buffers */
#define X86_FEATURE_TSX_FORCE_ABORT	(18*32+13) /* "" TSX_FORCE_ABORT */
#define X86_FEATURE_PCONFIG		(18*32+18) /* Intel PCONFIG */
#define X86_FEATURE_SPEC_CTRL		(18*32+26) /* "" Speculation Control (IBRS + IBPB) */
#define X86_FEATURE_INTEL_STIBP		(18*32+27) /* "" Single Thread Indirect Branch Predictors */
#define X86_FEATURE_FLUSH_L1D		(18*32+28) /* Flush L1D cache */
#define X86_FEATURE_ARCH_CAPABILITIES	(18*32+29) /* IA32_ARCH_CAPABILITIES MSR (Intel) */
#define X86_FEATURE_SPEC_CTRL_SSBD	(18*32+31) /* "" Speculative Store Bypass Disable */

/*
 * BUG word(s)
 */
#define X86_BUG(x)			(NCAPINTS*32 + (x))

#define X86_BUG_F00F			X86_BUG(0) /* Intel F00F */
#define X86_BUG_FDIV			X86_BUG(1) /* FPU FDIV */
#define X86_BUG_COMA			X86_BUG(2) /* Cyrix 6x86 coma */
#define X86_BUG_AMD_TLB_MMATCH		X86_BUG(3) /* "tlb_mmatch" AMD Erratum 383 */
#define X86_BUG_AMD_APIC_C1E		X86_BUG(4) /* "apic_c1e" AMD Erratum 400 */
#define X86_BUG_11AP			X86_BUG(5) /* Bad local APIC aka 11AP */
#define X86_BUG_FXSAVE_LEAK		X86_BUG(6) /* FXSAVE leaks FOP/FIP/FOP */
#define X86_BUG_CLFLUSH_MONITOR		X86_BUG(7) /* AAI65, CLFLUSH required before MONITOR */
#define X86_BUG_SYSRET_SS_ATTRS		X86_BUG(8) /* SYSRET doesn't fix up SS attrs */
#ifdef CONFIG_X86_32
/*
 * 64-bit kernels don't use X86_BUG_ESPFIX.  Make the define conditional
 * to avoid confusion.
 */
#define X86_BUG_ESPFIX			X86_BUG(9) /* "" IRET to 16-bit SS corrupts ESP/RSP high bits */
#endif
#define X86_BUG_NULL_SEG		X86_BUG(10) /* Nulling a selector preserves the base */
#define X86_BUG_SWAPGS_FENCE		X86_BUG(11) /* SWAPGS without input dep on GS */
#define X86_BUG_MONITOR			X86_BUG(12) /* IPI required to wake up remote CPU */
#define X86_BUG_AMD_E400		X86_BUG(13) /* CPU is among the affected by Erratum 400 */
#define X86_BUG_CPU_MELTDOWN		X86_BUG(14) /* CPU is affected by meltdown attack and needs kernel page table isolation */
#define X86_BUG_SPECTRE_V1		X86_BUG(15) /* CPU is affected by Spectre variant 1 attack with conditional branches */
#define X86_BUG_SPECTRE_V2		X86_BUG(16) /* CPU is affected by Spectre variant 2 attack with indirect branches */
#define X86_BUG_SPEC_STORE_BYPASS	X86_BUG(17) /* CPU is affected by speculative store bypass attack */
#define X86_BUG_L1TF			X86_BUG(18) /* CPU is affected by L1 Terminal Fault */
#define X86_BUG_MDS			X86_BUG(19) /* CPU is affected by Microarchitectural data sampling */
#define X86_BUG_MSBDS_ONLY		X86_BUG(20) /* CPU is only affected by the  MSDBS variant of BUG_MDS */
#define X86_BUG_SWAPGS			X86_BUG(21) /* CPU is affected by speculation through SWAPGS */

```

### flag的一些特性的解释
特性|解释
---|---
lm|long mode,即长模式。64位扩展，AMD的AMD64或Intel的EM64T
vmx  |查看CPU是否支持全虚拟化技术，  Intel的CPU可以在终端中输入  cat /proc/cpuinfo 
svm|svm 是 secure virtual machine的简写.它是AMD的虚拟化扩展到64位的x86CPU架构。等同于Intel的vmx，在xen 虚拟化管理程序中它们统称为hvm
pae |物理地址扩展。pae是IA32处理器的附加功能用于寻址超过4GB的物理内存，通过使用Intel的36位页面寻址替代标准的32位页面寻址方式，处理器可以访问总达64GB的内存。许多AMD芯片也支持pae
vme |virtual-8086 mode enhancement。
de |debugging extensions.
sse |Streaming SIMD Extensions. Developed by Intel for its Pentium III but also    implemented by AMD processors from Athlon XP onwards
pse |页面大小扩展功能。
pse36| 页面大小扩展36. IA-32支持两种方式访问4GB以上的内存，pse和pae
3DNOW|A multimedia extension created by AMD for its processors, based on / almost equivalent to Intel’s MMX extensions
3DNOWEXT|3DNOW Extended. Also known as AMD’s 3DNow!Enhanced 3DNow!Extensions
APIC|Advanced Programmable Interrupt Controller
CLFSH/CLFlush|Cache Line Flush
CMOV|Conditional Move/Compare Instruction
CMP_Legacy|Register showing the CPU is not Hyper-Threading capable
Constant_TSC|on Intel P-4s, the TSC runs with constant frequency independent of cpu frequency when EST is used
CR8Legacy|-unknown-
CX8|CMPXCHG8B Instruction. (Compare and exchange 8 bytes. Also known as F00F, which is an abbreviation of the hexadecimal encoding of an instruction that exhibits a design flaw in the majority of older Intel Pentium CPU).
CX16|CMPXCHG16B Instruction. (CMPXCHG16B allows for atomic operations on 128-bit double quadword (or oword) data types. This is useful for high resolution counters that could be updated by multiple processors (or cores). Without CMPXCHG16B the only way to perform such an operation is by using a critical section.)
DE|Debugging Extensions
DS|Debug Store
DS_CPL|CPL qualified Debug Store (whatever CPL might mean in this context)
DTS|Could mean Debug Trace Store or Digital Thermal Sensor, depending on source
EIST/EST|Enhanced Intel SpeedsTep
FXSR|FXSAVE/FXRSTOR. (The FXSAVE instruction writes the current state of the x87 FPU, MMX technology, Streaming SIMD Extensions, and Streaming SIMD Extensions 2 data, control, and status registers to the destination operand. The destination is a 512-byte memory location. FXRSTOR will restore the state saves).
FXSR_OPT|-unknown-
HT|Hyper-Transport. Note that the same abbreviation might is also used to indicate Hyper Threading (see below)
HTT/HT|Hyper-Threading. An Intel technology that allows quasi-parallel execution of different instructions on a single core. The single core is seen by applications as if it were two (or potentially more) cores. However, two true CPU cores are almost always faster than a single core with HyperThreading. This flag indicates support in the CPUwhen checking the flags in /proc/cpuinfo on Linux systems. For more info how you can detect active HyperThreading, see the first comment in my blog post about this page at[2]
LAHF_LM|Load Flags into AH Register, Long Mode.
LM|Long Mode. (64bit Extensions, AMD’s AMD64 or Intel’s EM64T).
MCA|Machine Check Architecture
MCE|Machine Check Exception
MMX|It is rumoured to stand for MultiMedia eXtension or Multiple Math or Matrix Math eXtension, but officially it is a meaningless acronym trademarked by Intel
MMXEXT|MMX Extensions – an enhanced set of instructions compared to MMX
MON/MONITOR|CPU Monitor
MSR|RDMSR and WRMSR Support
MTRR|Memory Type Range Register
NX|No eXecute, a flag that can be set on memory pages to disable execution of code in these pages
PAE|Physical Address Extensions. PAE is the added ability of the IA32 processor to address more than 4 GB of physical memory using Intel’s 36bit page addresses instead of the standard 32bit page addresses to access a total of 64GB of RAM. Also supported by many AMD chips
PAT|Page Attribute Table
PBE|Pending Break Encoding
PGE|PTE Global Bit
PNI|Prescott New Instruction. This was the codename for SSE3 before it was released on the Intel Prescott processor (which was later added to the Pentium 4 family name).
PSE|Page Size Extensions. (See PSE36)
PSE36|Page Size Extensions 36. IA-32 supports two methods to access memory above 4 GB (32 bits), PSE and PAE. PSE is the older and far less used version. For more information, take a look at [1].
SEP|SYSENTER and SY***IT
SS|Self-Snoop
SSE|Streaming SIMD Extensions. Developed by Intel for its Pentium III but also implemented by AMD processors from Athlon XP onwards
SSE2|Streaming SIMD Extensions 2. (An additional 144 SIMDs.) Introduced by Intel Pentium 4, on AMD since Athlon 64
SSE3|Streaming SIMD Extensions 3. (An additional 13 instructions) introduced with “Prescott” revision Intel Pentium 4 processors. AMD introduced SSE3 with the Athlon 64 “Venice” revision
SSSE3|Supplemental Streaming SIMD Extension 3. (SSSE3 contains 16 new discrete instructions over SSE3.) Introduced on Intel Core 2 Duo processors. No AMD chip supports SSSE3 yet.
SSE4|Streaming SIMD Extentions 4. Future Intel SSE revision adding 50 new instructions which will debut on Intel’s upcoming “Nehalem” processor in 2008. Also known as “Nehalem New Instructions (NNI)
TM|Thermal Monitor
TM2|Thermal Monitor 2
TSC|Time Stamp Counter                                               
XTPR|TPR register chipset update control messenger. Part of the APIC code


## 有助于MySQL的编译特性
```
CXX=gcc \
CHOST=”x86_64-pc-linux-gnu” \
CFLAGS=” -O3 \
-fomit-frame-pointer \
-pipe \
-march=nocona \
-mfpmath=sse \
-m128bit-long-double \
-mmmx \
-msse \
-msse2 \
-maccumulate-outgoing-args \
-m64 \
-ftree-loop-linear \
-fprefetch-loop-arrays \
-freg-struct-return \
-fgcse-sm \
-fgcse-las \
-frename-registers \
-fforce-addr \
-fivopts \
-ftree-vectorize \
-ftracer \
-frename-registers \
-minline-all-stringops \
-fbranch-target-load-optimize2″ \
CXXFLAGS=”${CFLAGS}” \
./configure –prefix=/usr/alibaba/install/mysql-ent-official-5.1.56 \
–with-server-suffix=alibaba-mysql \
–with-mysqld-user=mysql \
–with-plugins=partition,blackhole,csv,heap,innobase,myisam,myisammrg \
–with-charset=utf8 \
–with-collation=utf8_general_ci \
–with-extra-charsets=gbk,gb2312,utf8,ascii \
–with-big-tables \
–with-fast-mutexes \
–with-zlib-dir=bundled \
–enable-assembler \
–enable-profiling \
–enable-local-infile \
–enable-thread-safe-client \
–with-readline \
–with-pthread \
–with-embedded-server \
–with-client-ldflags=-all-static \
–with-mysqld-ldflags=-all-static \
–without-query-cache \
–without-geometry \
–without-debug \
–without-ndb-debug
```

参数|解释
---|---
-fomit-frame-pointer|对于不需要栈指针的函数就不在寄存器中保存指针，因此可以忽略存储和检索地址的代码，同时对许多函数提供一个额外的寄存器。所有”-O”级别都打开它，但仅在调试器可以不依靠栈指针运行时才有效。在AMD64平台上此选项默认打开，但是在x86平台上则默认关闭。建议显式的设置它。
-pipe|在编译过程的不同阶段之间使用管道而非临时文件进行通信，可以加快编译速度。建议使用。
march=nocona|Xoen 55xx处理器在GCC 4.1.3
-mfpmath=sse|启用cpu支持”sse”标量浮点指令。
m128bit-long-double|指定long double为128位，pentium以上的cpu更喜欢这种标准，并且符合x86-64的ABI标准，但是却不附合i386的ABI标准。
-mmmx -msse -msse2|使用相应的扩展指令集以及内置函数
-maccumulate-outgoing-args|指定在函数引导段中计算输出参数所需最大空间，这在大部分现代cpu中是较快的方法；缺点是会明显增加二进制文件尺寸。
-m64|生成专门运行于64位环境的代码，不能运行于32位环境，仅用于x86_64[含EMT64]环境。
-ftree-loop-linear|在trees上进行线型循环转换。它能够改进缓冲性能并且允许进行更进一步的循环优化。
-fprefetch-loop-arrays|生成数组预读取指令，对于使用巨大数组的程序可以加快代码执行速度，适合数据库相关的大型软件等。具体效果如何取决于代码。
-freg-struct-return|如果struct和union足够小就通过寄存器返回，这将提高较小结构的效率。如果不够小，无法容纳在一个寄存器中，将使用内存返回。建议仅在完全使用GCC编译的系统上才使用。
-fgcse-sm|在全局公共子表达式消除之后运行存储移动，以试图将存储移出循环。
-fgcse-las|在全局公共子表达式消除之后消除多余的在存储到同一存储区域之后的加载操作。
-frename-registers -fforce-addr|必须将地址复制到寄存器中才能对他们进行运算。由于所需地址通常在前面已经加载到寄存器中了，所以这个选项可以改进代码。
-fivopts|在trees上执行归纳变量优化。
-ftree-vectorize|在trees上执行循环向量化。
-ftracer|执行尾部复制以扩大超级块的尺寸，它简化了函数控制流，从而允许其它的优化措施做的更好。
-frename-registers|试图驱除代码中的假依赖关系，这个选项对具有大量寄存器的机器很有效。
-minline-all-stringops|默认时GCC只将确定目的地会被对齐在至少4字节边界的字符串操作内联进程序代码。该选项启用更多的内联并且增加二进制文件的体积，但是可以提升依赖于高速 memcpy, strlen, memset 操作的程序的性能。数据库系统使用这个参数可以显著提高内存操作性能。
-fbranch-target-load-optimize2|在执行序启动以及结尾之前执行分支目标缓存器加载最佳化。