# non-determinism

The code generation phase in LLVM suffers from the problem of non-determinism. This means that the same input program might have different object code generated depending on the way the LLVM toolchain was built. We might get different output between release only and release plus asserts builds, between a Windows and a Linux build or even different back-to-back runs of the same toolchain.

While the generated code might not necessarily be “wrong code”, this non-determinism might result in unexpected runtime crashes or simply hard to reproduce bugs on the customer side making it harder to debug and fix.

A common cause of non-determinism is the iteration of unordered containers in LLVM. Many such cases of non-deterministic behavior can be uncovered by forcing reverse iteration of these unordered containers and checking if the generated code changes. Reverse iteration simply means reversing the direction of iteration of a container. It is transparent to the user and comes with almost zero runtime cost.
