---

title: "Ruby JIT Explained: How Ruby Makes Your Code Faster"
date: 2026-10-10
draft: false
description: "Understand Just-In-Time compilation in Ruby, how YJIT works, how to enable it, and what it means for Ruby on Rails performance."
tags: ["Ruby", "Performance", "YJIT", "Ruby on Rails"]
categories: ["Backend Engineering"]
showToc: true
TocOpen: true
-------------

# Ruby JIT Explained: How Ruby Makes Your Code Faster

Ruby is known for its expressive syntax and developer-friendly experience. But when we talk about backend performance, one question often comes up:

**How can Ruby execute code faster without changing the way we write it?**

One important part of the answer is **JIT compilation**.

In this article, we'll explore what JIT means, how Ruby's YJIT compiler works at a high level, how to enable it, and what backend engineers should consider before using it in production.

## 1. What is JIT compilation?

JIT stands for **Just-In-Time compilation**.

Traditionally, Ruby's CRuby implementation executes Ruby code using a virtual machine. Ruby source code is parsed and compiled into an internal instruction format, which the virtual machine executes.

A JIT compiler takes another step: it compiles selected parts of a program into native machine code while the program is running.

Instead of interpreting the same frequently executed instructions every time, Ruby can execute compiled machine code for suitable portions of the application.

The goal is to improve execution speed while preserving Ruby's familiar programming model.

## 2. How does Ruby execute code?

Consider this simple Ruby method:

```ruby
def calculate_total(price, quantity)
  price * quantity
end

100_000.times do
  calculate_total(100, 5)
end
```

Conceptually, the execution flow looks like this:

1. Ruby reads and parses the source code.
2. The compiler produces an internal instruction representation.
3. The Ruby virtual machine executes those instructions.
4. With JIT enabled, eligible code can be compiled into native machine code.
5. Subsequent executions can benefit from that compiled code.

This is a simplified explanation. The exact compilation process depends on the Ruby implementation and its JIT compiler.

## 3. What is YJIT?

YJIT stands for **Yet Another Ruby JIT**.

It is a JIT compiler built into CRuby, the standard Ruby implementation. It was developed to improve the performance of real-world Ruby applications without requiring developers to rewrite their programs.

YJIT uses lazy compilation. Rather than compiling every method before the program starts, it compiles code as execution reaches suitable portions of the program.

This approach can help avoid spending compilation time on code that never runs.

YJIT also uses runtime information to optimize compiled code. When its assumptions no longer hold, execution can fall back to other execution paths and the compiler can adapt.

The important idea is that YJIT tries to spend optimization effort where it can provide a practical benefit.

## 4. How to enable YJIT

First, check your Ruby version:

```bash
ruby -v
```

Then check whether YJIT is available:

```bash
ruby --yjit -v
```

To run a Ruby script with YJIT enabled:

```bash
ruby --yjit app.rb
```

You can also enable it through an environment variable:

```bash
RUBY_YJIT_ENABLE=1 ruby app.rb
```

For a Rails application, you can try starting the server with:

```bash
ruby --yjit bin/rails server
```

Check your Ruby version and deployment environment before enabling it. Available options and behavior vary across Ruby releases.

## 5. What about Ruby on Rails?

Imagine a Rails application that processes thousands of requests and repeatedly executes the same application code.

Some of that work may involve:

* Calling service objects.
* Applying business rules.
* Transforming collections.
* Running calculations.
* Executing Ruby methods repeatedly.

YJIT may improve the execution speed of eligible Ruby code in these paths.

However, a web request also involves other work:

* Database queries.
* Redis operations.
* Network calls.
* JSON serialization.
* External API requests.

JIT compilation does not automatically make all these operations faster.

For example, if a request spends most of its time waiting for a slow SQL query, enabling YJIT might have little effect on the total response time.

**The lesson:** measure where your application spends its time before choosing an optimization.

## 6. Does JIT always make Ruby faster?

No. JIT compilation has trade-offs.

Potential benefits include:

* Faster execution of suitable, frequently executed Ruby code.
* Better performance for CPU-intensive application workloads.
* Improved throughput in applications where Ruby execution is a bottleneck.

Potential costs include:

* Additional memory for compiled code and metadata.
* Compilation overhead.
* Benefits that vary with workload and process lifetime.
* Little improvement when database or network latency dominates.

A short-lived process might not execute enough code to recover the cost of compilation. A long-running process with repeated workloads may benefit more.

This is why benchmarks should reflect actual application behavior rather than only isolated examples.

## 7. How should you benchmark YJIT?

Do not assume that enabling a compiler automatically improves production performance.

Start with a representative workload and compare runs with and without YJIT.

For a Rails application, measure:

* Requests per second.
* Median and p95 response latency.
* CPU utilization.
* Memory consumption.
* Startup time.
* Error rates and application stability.

Keep the Ruby version, workload, database state, and other conditions consistent.

For example, if you are benchmarking an API endpoint, use the same request mix and similar database conditions in both runs.

Also remember that benchmark results from a synthetic loop may not predict improvements in a real Rails application.

## 8. YJIT versus ZJIT

Ruby's JIT story continues to evolve.

YJIT is an established option for optimizing production Ruby workloads.

Ruby 4.0 also introduced ZJIT, a newer experimental method-based JIT compiler intended to improve Ruby performance further. The Ruby 4.0 release notes recommend experimenting with ZJIT rather than deploying it in production at that stage.

Always check the documentation for your specific Ruby version before choosing a JIT compiler.

## 9. What should backend engineers remember?

As backend engineers, we should think about performance as a system-level problem.

Before enabling JIT, ask:

1. Is the application CPU-bound?
2. Are database queries already optimized?
3. Does the workload repeatedly execute the same Ruby code?
4. Is there enough process lifetime to benefit from compilation?
5. Have we measured memory usage and latency under realistic traffic?

JIT is one tool in the performance optimization toolbox. It complements other techniques such as database indexing, reducing unnecessary queries, caching, profiling, and improving application architecture.

## Conclusion

JIT compilation gives Ruby another way to improve execution performance without changing the way we write Ruby code.

YJIT is particularly interesting because it focuses on optimizing code as the application runs, using information gathered during execution.

But the biggest lesson is simple: **understand your bottleneck, measure the results, and optimize based on evidence.**

That's how we build faster and more reliable backend systems.

### References

* [Ruby 4.0 release notes](https://www.ruby-lang.org/en/news/2025/12/25/ruby-4-0-0-released/)
* [Official YJIT documentation](https://docs.ruby-lang.org/en/master/jit/yjit_md.html)
