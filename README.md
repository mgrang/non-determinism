# Fighting Non-determinism in C++ Compilers

## INTRODUCTION
A C++ compiler exhibits non-deterministic behavior if, for the same input
program, the object code generated by the compiler differs from run to run. In
this work, I first explore the causes of such non-determinism. Then I outline
the scenarios where non-determinism is observed and examine why such behavior
is undesirable. I then present a case study on my work to uncover and fix
non-deterministic behavior in the **LLVM C++ Compiler**. Finally, I report on
the impact that my work has had on the LLVM community, the total number of bugs
found and how I fixed them.

## RELEVANCE
Millions of C++ developers around the world use compilers to develop their
programs. Not every developer necessarily understands the internals of the
compiler. So, having a robust compiler becomes supremely important. However,
the behavior of a compiler may not always be deterministic. For the same input
program, it may generate different code in different scenarios. This
non-determinism can make debugging difficult, result in hard-to-reproduce bugs,
cause unexpected runtime crashes or unpredictable performance.  My work
attempts to uncover non-deterministic behavior in the LLVM compiler thereby
making LLVM more robust.

## DISCUSSION
A C++ compiler may exhibit non-deterministic behavior. This means that for the
same input program the object code generated by the compiler may differ from
run to run.  This non-deterministic behavior can either remain hidden or
manifest itself in several scenarios. For example, the same compiler hosted on
different operating systems might generate different object code for the same
input program. Or there might be differences in behavior between asserts and
non-asserts version of the same compiler. Or even back-to-back runs of the same
compiler can produce different object code for the same input.

I have identified three main causes of non-deterministic behavior in a C++
compiler:
1. Iteration of unordered containers
2. Hashing of pointer keys
3. Use of non-stable sort functions

All three arise due to poor understanding of the behavior of various containers
and algorithms.  The detection of such non-deterministic behavior is often
challenging since the compiler may not always behave in an expected way. In
LLVM I try to uncover non-determinism in 2 ways:

### 1. Iteration order non-determinism
I implemented a “reverse iteration” mode for all supported unordered containers
in LLVM. The CMake flag LLVM_REVERSE_ITERATION enables the reverse iteration
mode. This mode makes all supported containers iterate in reverse, by default.
The idea is to compare the output of a reverse iteration compiler with that of
a forward iteration compiler to weed out iteration order randomness.  This mode
is transparent to the user and comes with almost zero runtime cost.

The following upstream buildbot tracks this mode:
http://lab.llvm.org:8011/builders/reverse-iteration

### 2. Sorting order non-determinism
I added a wrapper function to LLVM called llvm::sort which randomly shuffles a
container before invoking std::sort. The idea is that randomly shuffling a
container would weed out non-deterministic sorting order of keys with the same
values.

The following upstream buildbot tracks this mode:
http://lab.llvm.org:8011/builders/llvm-clang-x86_64-expensive-checks-win

Some best practices that were followed in LLVM to avoid or fix
non-deterministic behavior are:
1. Sort the container before iteration
2. Use a stronger sort predicate
3. Use a stable sort function
4. Use an ordered container

## COMPLETION STATUS
My work to enable reverse iteration and random shuffling a container is
complete and available upstream in the latest 6.0 release of LLVM. I have so
far uncovered and fixed 42 iteration order bugs and 44 sorting order bugs. The
upstream buildbots regularly catch non-determinism bugs and the community
promptly fixes them. As a result of my work, the LLVM community has become more
diligent in their use of containers and sorting algorithms. I have also added
coding standards for LLVM compiler developers on the correct use of unordered
containers and sorting algorithms:

[1] https://llvm.org/docs/CodingStandards.html#beware-of-non-determinism-due-to-ordering-of-pointers

[2] https://llvm.org/docs/CodingStandards.html#beware-of-non-deterministic-sorting-order-of-equal-elements

## PRESENTATIONS
I have presented my work at the following conferences:

[1] Fighting Non-determinism in C++ Compilers (CppCon 2018, Bellevue, WA)
https://github.com/CppCon/CppCon2018/tree/master/Posters/fighting_nondeterminism_in_cpp_compilers

[2] Non-determinism in LLVM Code Generation (LLVM Developers' Meeting 2017, San Jose, CA)
http://llvm.org/devmtg/2017-10/#poster11

https://github.com/mgrang/non-determinism/blob/master/poster__nondeterminism_in_llvm_code_generation__llvmdevmeet_2017.pdf

## NEXT STEPS
The next logical step would be to apply the ideas presented here in a more
wider context to help find non-determinism in user code. With that in mind, I
have started writing a Clang Static Analyzer checker to detect instances of
non-determinism caused by sorting pointer-like keys.

The beauty of the analyzer checks is that they run at compile time and hence
are cheap to test. The drawback is that there may be some false positives which
need to be pruned with better heuristics.  Here is the RFC and the patch (WIP)
for this:

[1] http://lists.llvm.org/pipermail/llvm-dev/2018-August/125191.html

[2] https://reviews.llvm.org/D50488

## REFERENCES
My work has featured several times in the LLVM weekly newsletters and other
places:

[1] https://bugs.swift.org/browse/SR-6154

[2] https://blog.jetbrains.com/clion/2017/12/cpp-annotated-sep-dec-2017

[3] http://bitupdate.us/compiler-infrastructure-llvm-5-0-provides-new-tools

[4] http://llvmweekly.org/issue/224

[5] http://llvmweekly.org/issue/201

[6] http://llvmweekly.org/issue/193

[7] http://llvmweekly.org/issue/192

[8] http://llvmweekly.org/issue/184

[9] http://llvmweekly.org/issue/151

[10] http://lists.llvm.org/pipermail/llvm-dev/2016-November/107098.html

[11] http://lists.llvm.org/pipermail/llvm-dev/2017-July/115025.html

[12] http://lists.llvm.org/pipermail/llvm-dev/2017-August/116975.html

[13] http://lists.llvm.org/pipermail/llvm-dev/2017-October/118639.html
