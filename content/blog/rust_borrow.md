+++
author = "lwcas"
title = "#1: rust borrow"
date = "2025-11-07"
description = "what if i just dont ever read this again xd"
tags = [
    "rust",
]
+++

# what is "borrow" in rust?

borrow in rust is a way to use a value without taking ownership of it. instead of moving a value from one place to another, you can "borrow" it using references. this helps rust ensure memory safety without needing a garbage collector.

imagine you and a friend share a laptop with a draft document:
- **owner:** you own the laptop and the sole editable copy of the draft.
- **immutable borrow (viewing):** you give your friend a read-only copy or open the file in "view only" mode — multiple friends can view the draft at the same time, nobody can change it.
- **mutable borrow (editing):** you hand the laptop to one friend or grant exclusive edit access — that friend can modify the draft, but while they have edit access you (and others) cannot edit it.
- **lifetimes:** the borrow lasts while the friend has the laptop or edit session; when they finish and return it, you regain full access.

the result: many memory errors are caught before the program runs, so code is safer without needing a garbage collector.

## how it works in practice
- immutable borrow (reading): you borrow only to read. in rust this is `&`.
- mutable borrow (changing): you borrow to modify. in rust this is `&mut`.

immutable borrow:
```rust
let laptop = String::from("draft document"); // you own the laptop with the draft
let viewer1 = &laptop; // friend1 opens in "view only" mode
let viewer2 = &laptop; // friend2 also views at the same time
println!("{}", viewer1);
println!("{}", viewer2);
```

mutable borrow:
```rust
let mut laptop = vec!["intro".to_string(), "body".to_string()]; // owner of the document parts
let editor = &mut laptop; // one friend gets exclusive edit access
editor.push("conclusion".to_string()); // friend edits the document
println!("{:?}", editor);
```

more abstract &mut example showing immediate update and scope:
```rust
let mut laptop = String::from("raft v1"); // owner
{
    let editor = &mut laptop; // exclusive edit session starts
    editor.push_str(" — updated"); // changes happen immediately on the laptop
    println!("{}", editor); // prints "draft v1 — updated"
} // editor goes out of scope here
println!("{}", laptop); // prints "draft v1 — updated" — owner can access again
```

example that will not compile:
```rust
let mut laptop = String::from("draft v1");
let editor = &mut laptop;
// println!("{}", laptop); // error: cannot borrow `laptop` as immutable while `editor` (a mutable borrow) exists
```

## why these rules exist
these rules prevent common mistakes like using something that was already removed or modifying something at the same time (which can cause unexpected behavior). they help rust guarantee **memory safety** at compile time.

**memory safety** means a program can't read or write memory it shouldn't. this prevents common bugs like:

- **use-after-free**: reading data after it was dropped  
    - can cause undefined behavior: crashes, corrupted memory, or wrong values  
    - security risk: may allow attackers to execute code or disclose sensitive data  
    - typically timing-dependent and hard to reproduce, making bugs elusive

- **double-free**: freeing the same memory twice  
    - leads to heap/allocator corruption and undefined behavior, often crashing the program  
    - can corrupt allocator metadata so future allocations return invalid pointers  
    - a common vector for exploitation (control-flow hijack or arbitrary code execution)

- **data races**: two threads changing the same data at once  
    - causes nondeterministic behavior, lost updates, and inconsistent reads (corrupted state)  
        - can produce crashes, deadlocks, or subtle logic errors that are hard to reproduce  
        - may break program invariants and create security issues when sensitive data is involved  

## extras

garbage collector: a garbage collector (gc) is a runtime component that automatically detects and reclaims memory a program no longer uses, so developers don't have to manually free memory. this simplifies development but adds runtime overhead, potential pause times, and less predictable performance. rust instead provides deterministic, compile-time memory safety via ownership, borrowing, and lifetimes, dropping values at well-defined points without a gc.