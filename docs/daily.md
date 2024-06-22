# Daily Progress Log

## Mon-Tue 6/17-18/2024
Looking at how are LinearMemory instances provisioned by the runtime, will include these in a seperate document.

## Fri 6/14/2024
**Summary**: Transitioned from Wasmer to Wasmtime, and added userspace debugging document. Also fixed the syscall return value at the interface between glibc and runtime.\
**What's next**: Find someway to implement `brk`/`sbrk` in the runtime as host functions.

About host functions:\
Currently the `lind_syscall` host function is integrated within the `wasi_snapshot_preview1` interface, along with the `fd_write` stuff. I added this to preview1 for simplicity, where only the `from_witx!` and the witx file need to be changed. In the longer term we want to seperate lind_syscall with other wasi host functions of course.

About function return of host functions:\
In wasi-libc and Wasmtime (also Wasmer), the host functions doesn't return the values to the userspace caller directly. 
The last argument in the import signature (e.g. `__imported_wasi_snapshot_preview1_fd_write` in `wasi-libc`) is actually a pointer to the return value. When I implemented `lind_syscall` host function, the last argument in the import signature is also an `unsigned int`, where in the userspace I firstly decalre `int ret = 0`, then pass `&(unsigned int) ret` as the last argument. And copying the value returned by the actual `lind_syscall` in Wasmtime to the `ret` happens implicitly.

## Wed 6/12/2024
**Summary**: Consecutive syscalls okay in Wasmer (lind init in `run_wasm`). Added complete syscall support for malloc, except `brk`/`sbrk`.\
**Issues**: 1) **malloc fails after several mmap/munmap calls, hard to debug** 2) the `brk` syscall needs special handling, where `wasi-libc` is implemented with the compiler emitted function `__builtin_wasm_memory_grow`. Note that currently lind's `brk` is managed by NaCl.\
**What's next**: We _**MUST**_ find a way to trace/debug the wasm runtime's user space. We are currently exploring both Wasmer and Wasmtime's potential to do this. The `brk` need more investigation.

Wasmer has a "DWARF debugging repo here https://github.com/wasmerio/wasm-debug. [Well okay, this is way too obsolete]

Wasmtime's doc on debugging https://github.com/bytecodealliance/wasmtime/blob/main/docs/examples-debugging-native-debugger.md

trying to understand both

## Tue 6/11/2024
**Summary**: `malloc-hello` compiles successfully, Wasmer also validate the module successfully. \
**What's next**: the program fails after the first syscall to lind, as the initial modifications to Wasmer doesn't support consecutive syscalls. I'll update Wasmer to support consecutive syscalls, and Dennis will adapt more syscalls in glibc / rustposix.

Adapted `writev` and `munmap` in glibc to use lind syscalls. Add rustposix new dispatcher support for these 2 calls.

When we removed assembly implementations earlier, we added dummy replacement functions with argument types of `uint64_t`, which was absolutely wrong. We should just use standard posix arguments here, and the type conversion to `i64` is done by our `MAKE_SYSCALL` macro.

Added the WASM program compilation doc.
