## Behind the Scenes: Compiling a C Program

### 1. Preprocessing

* The preprocessor handles directives starting with `#`.
* Expands macros (`#define`), includes headers (`#include`), and removes comments.
* Output is an expanded source file (often `.i`).

Example:

```bash
gcc -E program.c > program.i
```

---

### 2. Compilation

* The preprocessed code is checked for syntax errors.
* Converted into assembly code.
* Optimizations may occur at this stage.

Example:

```bash
gcc -S program.i
```

Output: `program.s`

---

### 3. Assembly

* Assembly code is translated into machine (object) code.
* Produces an object file (`.o`).

Example:

```bash
gcc -c program.s
```

Output: `program.o`

---

### 4. Linking

* Object files are combined with libraries.
* Resolves external symbols (e.g., `printf`).
* Produces the final executable.

Example:

```bash
gcc program.o -o program
```

## Linking can be of two types:

* Static Linking: All the code is copied to the single file and then executable file is created.
* Dynamic Linking: Only the names of the shared libraries is added to the code and then it is referred during the execution.

---

### 5. Execution

* The generated executable is loaded into memory.
* The operating system starts execution from `main()`.

Example:

```bash
./program
```

<img width="1059" height="687" alt="image" src="https://github.com/user-attachments/assets/29e7a679-cc05-4976-a73e-63a5c758f4c9" />
