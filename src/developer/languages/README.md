# Language-specific Considerations

This section contains language-specific guides with recommendations. If there is no language specific section, it is because there is no specific guidance beyond using a suitably current version of the language.  Simply proceed as you would on any other CPUs, Arm-based or otherwise.

Broadly speaking, applications built using interpreted or JIT'd languages ([Python](python.md), [Java](java.md), PHP, Node.js, etc.) should run as-is on Arm64. Applications using compiled languages including [C/C++](c-c++.md), [Fortran](fortran.md), and [Rust](rust.md) need to be compiled for the Arm64 architecture.  Most modern build systems (Make, CMake, Ninja, etc.) will "just work" on Arm64.  
