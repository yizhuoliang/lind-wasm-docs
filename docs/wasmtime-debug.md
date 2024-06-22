# Introduction to Wasmtime and Debugging with GDB

## What is Wasmtime?

Wasmtime is a standalone JIT-style runtime for WebAssembly, designed for use with WebAssembly System Interface (WASI) and other WASI-inspired environments. It is part of the Bytecode Alliance, an open-source effort to create secure software foundations.

Wasmtime can run WebAssembly modules that follow the WASI standard, providing a robust and efficient environment for running WebAssembly outside of the browser.

## Getting Started with Wasmtime

To get started with Wasmtime, you can download and install it from the [official Wasmtime releases](https://github.com/bytecodealliance/wasmtime/releases) page. Follow the installation instructions specific to your operating system.

## Compiling a WebAssembly Module with Clang

Let's start with a simple example, `malloc-test.c`, which demonstrates dynamic memory allocation using `malloc` in C. We will compile this C program to a WebAssembly module using Clang with the WASI target.

### Example C Program: `malloc-test.c`

```c
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main() {
    const char *str = "Hello from Dennis's WASM!\n";

    size_t str_len = strlen(str) + 1;

    char *buf = malloc(str_len);

    if (buf == NULL) {
        return -1;
    }

    strcpy(buf, str);

    write(1, buf, str_len - 1);

    free(buf);

    return 0;
}
```

### Compiling the Program

Use the following command to compile the `malloc-test.c` program to WebAssembly:

```sh
../../clang+llvm-16.0.4-x86_64-linux-gnu-ubuntu-22.04/bin/clang-16 --target=wasm32-unknown-wasi --sysroot /home/dennis/Documents/Just-One-Turtle/wasi-libc/sysroot malloc-test.c -g -O0 -o malloc-test.wasm
```

- `--target=wasm32-unknown-wasi`: Specifies the target to be WebAssembly with WASI.
- `--sysroot /home/dennis/Documents/Just-One-Turtle/wasi-libc/sysroot`: Points to the WASI sysroot directory.
- `-g`: Includes debugging information.
- `-O0`: Disables optimizations for easier debugging.

## Running the WebAssembly Module with Wasmtime

After compiling the WebAssembly module, you can run it using Wasmtime:

```sh
../wasmtime/target/debug/wasmtime run malloc-test.wasm
```

## Debugging with GDB

To debug the WebAssembly module, you can use GDB with Wasmtime. Ensure that you have compiled the module with the `-g` flag to include debugging information.

**NOTE**: currently this debugging tool does not support inspecting instructions. And operations like `layout split` and `si` might break the terminal. Using `layout src` is recommended.

### Running GDB with Wasmtime

Use the following command to run GDB with Wasmtime:

```sh
gdb --args ../wasmtime/target/debug/wasmtime run -D debug-info -O opt-level=0 malloc-test.wasm
```

- `gdb --args`: Passes the arguments to GDB.
- `../wasmtime/target/debug/wasmtime run`: Specifies the Wasmtime executable.
- `-D debug-info`: Enables debugging information.
- `-O opt-level=0`: Sets the optimization level to 0 for debugging.

### Example Debugging Session

1. **Start GDB**:
   ```sh
   gdb --args ../wasmtime/target/debug/wasmtime run -D debug-info -O opt-level=0 malloc-test.wasm
   ```

2. **Set Breakpoints**:
   In the GDB prompt, set breakpoints as needed, for example:
   ```sh
   (gdb) break main
   ```

3. **Run the Program**:
   Start the execution of the WebAssembly module:
   ```sh
   (gdb) run
   ```

4. **Inspect and Debug**:
   Use GDB commands to inspect variables, step through the code, and debug your program:
   ```sh
   (gdb) next
   (gdb) print p
   (gdb) continue
   ```

By following these steps, you can compile, run, and debug WebAssembly modules using Wasmtime and GDB. This provides a powerful environment for developing and debugging WebAssembly applications.

For more details, refer to the official [Wasmtime documentation](https://wasmtime.dev/) and the [GDB documentation](https://www.gnu.org/software/gdb/documentation/).

