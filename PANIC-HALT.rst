.. SPDX-License-Identifier: GPL-2.0-only

Fatal error policy in this fork
===============================

The panic() and vpanic() entry points remain non-returning for compatibility
with existing callers. Fatal errors print a message, stop other CPUs, and
halt or request an emergency reboot. They never resume the failed caller.

The existing panic= boot parameter and kernel.panic sysctl select the action:

* 0: stop indefinitely with local interrupts disabled (default unless the
  configuration or architecture sets a different panic timeout).
* Negative: request an immediate emergency restart.
* Positive: wait that many seconds, then request an emergency restart.

If emergency restart returns, the CPU stops indefinitely. This is not an
orderly shutdown: filesystems are not synced and device shutdown callbacks
are not run. Platform watchdogs can still reset a halted machine.

The normal panic path no longer invokes crash kernels, panic notifier
callbacks, kgdb, stack/system dumps, console replay, panic blink callbacks,
or arch_do_panic(). Their interfaces remain for source compatibility, but
configuring those features does not restore the old panic pipeline.
The existing panic_force_cpu redirection and CPU arbitration are retained;
redirection can still print the originating CPU stack. emergency_restart()
may invoke the restart log dumper through the normal reboot implementation.

Validate on a disposable VM, never a machine holding unsaved data: trigger
SysRq-c with panic=0, panic=-1 and panic=3, checking the halt, immediate reboot
and delayed reboot respectively. Repeat with multiple vCPUs and a loaded
crash kernel to verify that the crash kernel is bypassed. Architecture-specific
restart and stop behavior requires testing on each intended target.
