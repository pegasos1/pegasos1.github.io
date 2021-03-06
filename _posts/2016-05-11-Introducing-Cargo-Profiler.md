---
layout: post
title: Introducing Cargo Profiler
comments: true
---

Profiling tools can help you write fast and efficient code. However, whether you use perf, oprofile, or valgrind, you have to exit the Rust ecosystem to profile your applications. This has always been a little cumbersome to me, so I built a cargo subcommand to perform the job: [cargo-profiler](https://github.com/kernelmachine/cargo-profiler).

Cargo-profiler interfaces with Linux-based profiling tools to:

  * Profile your applications
  * Parse profiler output into structs for further analysis
  * Display information to you in the most user-friendly way possible.

For example, instead of this gross cachegrind output:

`valgrind --tool=cachegrind $BINARY && cg_annotate $OUT_FILE`

<img src="http://kernelmachine.github.io/public/20160511/cachegrind_pic.png" width="900" height="400">

You get this prettier cachegrind output:

`cargo profiler cachegrind --bin=$BINARY -n 10`

![Cargo profiler](http://kernelmachine.github.io/public/20160511/cargoprofiler.png)

Since cargo-profiler parses performance statistics into machine-readable, structured objects, we can do a lot more with the data, even in a programmatic way.

This project is in its infancy, and here are some current ideas on the roadmap:

 * Comparisons between profiling runs
 * Creating macros that conditionally compile a binary with functions you want to profile
 * Getting better context around expensive functions in your library (location, types, etc.)
 * Building support for more profiling tools
 * Creating alternate consumptions of profiling data (e.g. visualizations)

Now, cargo-profiler is a simple and lightweight app that merely serves as an interface to existing tools. There's a whole world beyond this project that involves totally new and native Rust profiling workflows. These workflows could be really powerful and address some caveats to profiling Rust programs today. For example, compiler optimizations like inlining  render some functions at the code-level mangled or lost to valgrind. Perhaps native profiling at the MIR or LLVM level can solve this issue.

Native Rust profiling would definitely require major work, so in the meantime, leveraging existing tools seems like a good first step! I hope this tool is useful to you developers.
