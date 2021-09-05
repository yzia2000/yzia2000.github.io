---
layout: post
title: "Sourceror Compiler Proposal: Tail Call Optimizations"
date: 2021-09-4 10:14:48 +0530
categories: Portfolio
---

# Sourceror, Source, WASM

Sourceror is a compile from NUS SourceAcademy's Source programming language to
Web Assembly. Source was inspired heavily by several programming notions like meta-circulator evaluation,
virtual machines, functional programming amongst others. Web Assembly has opened
the door to wide array of experimentation with the available cross platform specification
boasting a very competitive runtime and resource utilization for both client side
and server side applications. A key feature of the WASM spec is the protected linear memory.
The Sourceror compiler is written in Rust and compiles
several Source abstractions including primitive data types, functions, modules and remote imports
and adds several optimizations like inlining.

# Proposal

Unfortunately, majority of the browsers do not support proper tail calls (or return calls)
from the WASM spec. A proper tail call happens when a function call in the return of
the calling function removes the caller completely from the stack before inserting the new stack frame
as the return of the caller now is just the return of the callee. This significantly reduces
the chances of stack overflow and is a great optimization to have.

```
function x() {
  return 1;
}

function y() {
  return x();
}

y();
```

In the above code, the stack frame of `y()` can be removed before calling `x()` as
returning from `y()` is the same as returning form `x()` now that `x()` has all the
context that it needs.

Another part of this spec is the interaction with conditional expressions.

```
function x() {
  return 1;
}

function y(n) {
  return n == 0 ? x() : 1;
}

y();
```

The `x()` will also be considered a tail call. This adds a difficult runtime dynamic
to the spec.

The immediate solution to this problem is to manually edit the stack frame. However,
WASM's stack is protected which prevents us from doing so. This project deals with the
trampoline proposal to tail call optimizations.

[Proposal](https://github.com/sourceror-t4-gt-yz/sourceror/wiki/Proper-Tail-Calls)

[Branch](https://github.com/sourceror-t4-gt-yz/sourceror/tree/proper-tail-calls)

_This project is only for academic purposes and not for commercial consumption_
