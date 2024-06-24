# Lind-WASM Project Documentation
Main Repo: [https://github.com/Lind-Project/lind-wasm](https://github.com/Lind-Project/lind-wasm)
![Lind-WASM Highlevel Arch](lind-wasm-arch.png"Lind-WASM Highlevel Arch")
## What is Lind-WASM project?
**We are building a secure and performant runtime system**, which provides POSIX style system support for the WebAssembly (WASM) platform.
Lind-WASM is compatible for unmodified C programs written for Linux, and can be executed as lightweight cages that enjoys Software Fault Isolation (SFI), fast Inter Process Communications (IPC), and many other advantages.

The core components that we made contribution to are:

- **WASM-glibc** derived from glibc, the mot complete standard C library for the WASM platform
- **Wasmtime** runtime modified tos support Linux style memory management, threading, etc
- **rustposix** library OS that contributes to SFI and fast IPC