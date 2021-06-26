---
layout: single
categories: 
    - Tools
author: Yue Yin

toc: true
toc_sticky: true
---

I haven't write serious C++ code for a while and somehow I have to today. I
totally forgot how to compile C++ code. This is to refresh my memory.

## How C++ compilation works

See detailed description here: https://github.com/yinfredyue/cpp_primer_solution#c-program-building-process.

```
Source C++ ---> Expanded C++ ---> Assembly code (.s) ----> Binary (.o) ---> Executable
```

Preprocessing: mainly expand macros (`#include`, etc)

Compiling: C++ to object file (`.s`)

Assembling: assembly code to binary code (`.o`)

Linking: binary code to executable


At compiling, C++ files are compiled separately. Object files can refer to symbols that are not defined. This is the case when you use a declaration, and don't provide a definition for it. The compiler doesn't mind this. The definition is not needed until linking.

At linking, linker replaces references to undefined symbols with correct addresses.

## Related topics

[Why C++ source file and header files](https://stackoverflow.com/q/333889/9057530)? The header only provides declaration, and it's enough for compilation (but not linking).

[Where does C++ compiler look for header files](https://stackoverflow.com/questions/344317/where-does-gcc-look-for-c-and-c-header-files)? Current directory of the source files, and some predefined directories like `usr/local/include/`, etc.

When using a compiler like g++ or clang, you only need to specify the source files (`.cpp`). You don't need to specify header files unless they are not in the current directory and also not in the predefined list of directories.

[What's the difference between static and dynamic library?](https://domiyanyue.medium.com/c-development-tutorial-4-static-and-dynamic-libraries-7b537656163e). Great explanation.
