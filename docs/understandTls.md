# Understanding TLS

**The most important take away of this document is to know the compiler/linker emitted symbols**: constants `__tls_size` and `__tls_align`, mut variable `__tls_base`, and the function `__wasm_init_tls`.

## General TLS supports of the compiler and linker without the presence of a threading library

**Compiler**:

- recognizes the variables declared with TLS qualifier (`__thread` or `_Thread_local`, different qualifiers depends on the compiler)
- put TLS variables into special segments of the object file, `tdata` for initialized variables, and `tbss` for uninitialized ones
- generate code for accessing each TLS variable with the address calculated by adding an offset to the `TLS_base`. Note that this   `TLS_base` is a mutable, latter I'll mention how to set this value. `TLS_base` is a thread-specific pointer points to the starting address of the TLS block for that thread. The actual place storing this `TLS_base` pointer varries depends on the compiler and architecture (e.g., for x86_64, GCC put this pointer at the `fs` register, and for WASM, this can be accessed by `global.set` and `global.get`).

**Linker**:

- consolidates the TLS segments from different objet files into a single one, which need to recalculate the offsets of each TLS variable
- the linker also create symbols `TLS_size`, `TLS_align`, and a TLS init function. In the case of WASM these are `__tls_size`, `__tls_align`, `__tls_base`, and the function `__wasm_init_tls(mem)`, where the `mem` parameter is a pointer specifying the location to init the TLS block.

**Threading library** (wasm):

Note that the `__wasm_init_tls` simply initialize the TLS block according to the TLS variables declared in the source file, it doesn't allocate the memory. So, when `wasi-libc` implements the threading library, it uses the const symbols `__tls_size` and `__tls_align` to pre-allocate the TLS memory before invoking `__wasm_init_tls`. And of course, like we anticipated, the `pthread_create` calls `__wasm_init_tls` to set up the new thread's TLS block. Eventually the `pthread_create` also set the `__tls_base` variable, so that later the user's code will access the TLS variables starting from this base address.

For more detailed description of `__tls_size` and `__tls_align`, see this LLVM doc https://github.com/WebAssembly/tool-conventions/blob/main/Linking.md#thread-local-storage.