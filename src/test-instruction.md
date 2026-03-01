# The test instruction

## What is at address `7FFE0308h`?

For those who have sat the class, you know exactly what that address is. For those that haven't sat that class, this is looking into an undocumented structure. If you look at `308h` as an offset into a `struct`, you'd be left with `7FFE0000h`. That user mode address is the static memory location for the **`_KUSER_SHARED_DATA`** struct. Here is a small snippet of that struct dumped from WinDbg on a Windows 10 system without Virtual Based Security enabled.

```text
0: kd> dt ntdll!_KUSER_SHARED_DATA 0x7ffe0000
   +0x000 TickCountLowDeprecated 0 : Uint4B
   +0x004 TickCountMultiplier 0xfa00000 : Uint4B
   +0x008 InterruptTime    : _KSYSTEM_TIME
   +0x014 SystemTime       : _KSYSTEM_TIME
   +0x020 TimeZoneBias     : _KSYSTEM_TIME
   +0x02c ImageNumberLow   0x8664 : Uint2B
   +0x02e ImageNumberHigh  0x8664 : Uint2B
   +0x030 NtSystemRoot     "C:\Windows": [260] Wchar
[...SNIP...]
   +0x308 SystemCall      0 : Uint4B
```

> [!NOTE]
> `_KUSER_SHARED_DATA` is read-only for user mode and is mapped at the same address in every process. Exploit and implant code often reads it to detect VBS or to find kernel-related hints.

It's a massive structure that is often abused by exploit devs and implant devs. One of the interesting fields is at offset `308h`, **SystemCall**.

## `_KUSER_SHARED_DATA.SystemCall`

Back in the day, this field used to hold the address that was going to be executed in the kernel. This became abused and now the meaning has changed for it. Currently, the value will be either clear (0) or set (1), and is a way to detect if **Virtual Based Security** is enabled for a system. If it is, then the `TEST` instruction will result in **TRUE**, the **EFLAGS** will be updated accordingly, and the `JMP` will be taken. The `JMP` will then land on the `INT 2Eh` instruction. This also means that there are **Virtual Trust Levels** and the syscalls will now happen in **VTL0**.

> [!TIP]
> The `SYSCALL` instruction is measurably faster than `INT 2Eh` by about one clock tick. When VBS is off and both paths are valid, the kernel uses `SYSCALL`; when VBS is on, the stub uses `INT 2Eh` and VTL0.
