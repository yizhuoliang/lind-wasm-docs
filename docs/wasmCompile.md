# Troubleshooting Compiling WASM

**Assuming using `clang-18` provided by the `wasi-sdk-22.0`.**

## Frequently Used Flags
- `--target=wasm32-unkown-wasi` for compiling to wasm
- `-c` for compiling as a library without main executable
- `-pthread` then the compiler to understand `__tls_base` etc
- `--sysroot` specifying the stand library path

## Inspecting an Object File
Use `wasm-objdump -x` to show all functions, globals, etc included in the object file. Use `-d` to disassemble and see text instructions.

## Locating Type Mismatch Error
Even successfully compile a source into .wasm, a typical error due to mismatch will manifest during secondary compilation, like this
```
root@2299d4e20d1f:/lind-glibc/replace-sysroot/test# /wasmer/target/debug/wasmer run newhello.wasm
error: Unable to compile "newhello.wasm"
╰─▶ 1: Validation error: type mismatch: values remaining on stack at end of block (at offset 0x26a)
```
One can also verify and get the same error by `wasm2wat`, which gives a more detailed message. To debug, we can use `wasm-objdump -x` to see the function types, a typical output is this

```
Section Details:

Type[16]:
 - type[0] (i32, i32) -> i32
 - type[1] (i32) -> nil
 - type[2] (i32, i32, i32, i32, i32, i32, i32, i32) -> i32
 - type[3] (i32, i64, i64, i64, i64, i64, i64, i64) -> i32
 - type[4] () -> nil
 - type[5] () -> i32
 - type[6] (i32, i32, i32) -> i32
 - type[7] (i32) -> i32
 - type[8] (i32, i32, i32, i64) -> i32
 - type[9] (i32, i32, i32) -> nil
 - type[10] (i32, i32, i32, i32) -> i32
......
Import[1]:
 - func[0] sig=3 <__imported_wasi_snapshot_preview1_lind_syscall> <- wasi_snapshot_preview1.lind_syscall
Function[92]:
 - func[1] sig=4 <_start>
 - func[2] sig=4 <__wasm_call_dtors>
 - func[3] sig=5 <__original_main>
 - func[4] sig=6 <__libc_write>
 - func[5] sig=7 <__brk>
 - func[6] sig=7 <__sbrk>
 - func[7] sig=0 <__clock_gettime64>
 - func[8] sig=0 <__GI___munmap>
 - func[9] sig=6 <__GI___madvise>
 - func[10] sig=6 <__open64_nocancel>
......
```

 Note that the `Type` section defines all the function args/ret types, and for each function, the `sig=X` just means the function has `type[X]`.