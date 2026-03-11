---
layout: post
title: "Smart Pointers Are Not Magic — Here's What Actually Happens"
date: 2025-05-12
author: "Alex Mercer"
tags: [C++, Memory, Performance]
description: "Everyone says just use unique_ptr but few understand the zero-cost abstraction underneath. We disassemble the generated code and stare at what the compiler actually produces."
---

Every C++ tutorial tells you the same thing: *"use smart pointers, avoid raw pointers."*
And they're right. But most tutorials stop there, leaving you with a vague sense that
`unique_ptr` is somehow safe while `T*` is dangerous — without explaining the mechanism.

Today we're going deeper. We'll look at exactly what `std::unique_ptr` compiles to,
when it's truly zero-cost, and where the abstraction leaks.

## The Ownership Model

C++ memory ownership is a contract. When you `new` an object, you become responsible
for calling `delete`. Smart pointers enforce that contract through RAII —
Resource Acquisition Is Initialization.

> RAII means: the resource is acquired in a constructor and released in the destructor.
> Since destructors always run (even during stack unwinding from exceptions),
> the resource is always freed.

Here's the classic raw-pointer problem:

```cpp
Widget* w = new Widget();
process(w);  // throws an exception?
delete w;    // never reached — leak.
```

The fix isn't a try-catch around every allocation. The fix is making the resource's
lifetime tied to the scope:

```cpp
auto w = std::make_unique<Widget>();
process(w.get());
// w destroyed here — delete called automatically.
// Even if process() throws.
```

## What Does the Compiler Actually Generate?

Let's look at a minimal example and see what happens at the assembly level.
Take these two equivalent programs:

```cpp
// Version A: raw pointer
int* raw_version() {
    int* p = new int(42);
    return p;
}

// Version B: unique_ptr
std::unique_ptr<int> smart_version() {
    return std::make_unique<int>(42);
}
```

On x86-64 with `-O2`, both functions generate **identical assembly**.
The compiler sees through the wrapper completely. `unique_ptr` holds a single
pointer — no overhead, no vtable, no size difference.

## When unique_ptr Has Cost

The zero-cost claim breaks down in a few real scenarios:

**Custom deleters.** If you pass a non-empty deleter (like a lambda with captures),
`unique_ptr` must store it alongside the pointer. The type changes from 8 bytes to
potentially 24+ bytes. Use stateless function pointers or empty classes when possible.

**shared_ptr is different.** `shared_ptr` maintains a reference count
and a control block — two pointer-sized allocations, atomic increments, cache pressure.
Never use `shared_ptr` where `unique_ptr` will do.

**Passing by value in hot paths.** Passing a `unique_ptr` by value involves a move,
which nulls the source. Not expensive, but not free if you're doing it a million times
a second inside a tight loop.

## The Rule of Five (and Zero)

If you define any of: destructor, copy constructor, copy assignment, move constructor,
or move assignment — you should define all five. Or, better, use the Rule of Zero:
let the compiler generate all of them by using smart pointers as members.

```cpp
// Rule of Zero — compiler handles everything
class Document {
public:
    Document(std::string title, size_t size)
        : title_{std::move(title)},
          buffer_{std::make_unique<char[]>(size)} {}

    // No destructor, copy, or move needed.
    // unique_ptr handles memory. string handles its own.

private:
    std::string              title_;
    std::unique_ptr<char[]>  buffer_;
};
```

## Conclusion

Smart pointers aren't magic — they're a compiler-enforced contract. `unique_ptr`
is a zero-cost abstraction in the vast majority of cases, meaning you get all the safety
of automatic memory management with none of the runtime overhead.

The real skill is knowing *which* smart pointer to reach for: `unique_ptr`
for sole ownership, `shared_ptr` for shared ownership (sparingly), and raw
`T*` or references for non-owning access.

Next time: we'll look at how the compiler handles `shared_ptr`'s control block
and why `make_shared` is almost always the right choice over `shared_ptr(new T)`.
