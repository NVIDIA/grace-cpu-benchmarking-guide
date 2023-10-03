# Language-Specific Considerations

This section contains language-specific information with recommendations. If no section exists for a language, it is because there is no specific guidance beyond using a suitably current version of the language. You can proceed as usual on other CPUs, Arm-based, or otherwise.

Broadly speaking, applications that are built using interpreted or JIT'd languages ([Python](python.md), [Java](java.md), PHP, Node.js, and so on) should run as-is on Arm64. Applications using compiled languages, including [C/C++](c-c++.md), [Fortran](fortran.md), and [Rust](rust.md) need to be compiled for the Arm64 architecture.  Most modern build systems (Make, CMake, Ninja, and so on) will just work on Arm64.  
