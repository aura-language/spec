# Aura Language Design Specification
## v0.1.0-nightly — Pre-Alpha

> *"The code should never get in the way of the story."*

**Mascot:** Capy (2D flat-illustrated Brazilian capybara)  
**Backend:** LLVM (Beta+) / C transpilation (Alpha)  
**Interop:** C-first, polyglot via APM  
**Origin:** Brazil 🇧🇷

---

## Table of Contents

1. Core Philosophy — The Axioms
2. Design Constraints
3. Lexical Structure
4. Reserved Keywords
5. Syntactic Equivalence Pairs
6. Type System
7. Data Architecture — Structure vs. Concept
8. Memory Management
9. Control Flow
10. Functions
11. Functional Programming
12. Strings
13. Numbers and Operators
14. Module System
15. Multilingual Support — The Babel Fish Layer
16. Compiler Architecture and Roadmap
17. Aura Package Manager (APM)
18. Standard Library — Full Plan
19. Error Messages and Compiler Voices
20. Pointer Syntax
21. Bitwise Operations
22. Formatting and Tooling
23. Complete Code Examples
24. Brand Identity
25. Open Questions and Future Work
26. Contributing Guidelines

---

## 1. Core Philosophy — The Axioms

Aura is designed around six guiding principles. When two design choices
conflict, resolve them using the priority order listed here.
Axiom 0 is always the tiebreaker.

### Axiom 0 — Accessibility

Anyone who can read English (or their native language, once a locale
file exists) should understand at least 80% of Aura pseudocode with
zero programming background. Code should be showable to a
non-programmer and immediately meaningful. This is the highest priority
in all design decisions.

### Axiom 1 — No Friction

The language never gets in the way of development. A Python programmer,
a C programmer, and a Go programmer should all be able to start writing
Aura within one hour. The compiler, package manager, and tooling should
make decisions for you when you haven't specified otherwise — and make
good ones.

### Axiom 2 — Self-Commenting by Default

Code should read like a narrative told to a peer. Aura pseudocode
requires virtually no inline comments to be understandable. The
structure of the language itself documents intent.

### Axiom 3 — Syntactic Equivalence

There is no single "correct" way to write Aura. The pseudocode style
(verbose, natural language, accessible) and the terse style (concise,
symbol-heavy, familiar to experienced developers) compile to identical
Abstract Syntax Trees. They can be mixed freely within the same file.
Neither style is superior.

### Axiom 4 — Cultural Neutrality

Keywords are mapped via locale JSON files. The programming language you
write in is a compiler flag, not a barrier. A student in Brazil, Japan,
or France should be able to write and read Aura in their native tongue.
All popular programming languages use English keywords — learning to
code should not require learning English first.

### Axiom 5 — Performance is Non-Negotiable

Aura compiles via LLVM and must be able to compete with C and Go in
benchmarks. High-level, readable syntax must never mean high-level
runtime cost. The language should be usable for embedded systems,
systems programming, and performance-critical applications.

### Priority Order

When axioms conflict: 0 > 1 > 2 > 3 > 4 > 5

Cultural neutrality is the most deferrable.
No friction and accessibility are always the highest priorities.

### The Elevator Pitch

- "I want to teach coding to my 12-year-old. We don't speak a lick of
  English." → Aura is for you.
- "I need performance-critical code for an embedded system with 64 KB
  of RAM." → Aura is for you.
- "I want to rapid-prototype a web scraper this afternoon without
  fighting syntax." → Aura is for you.
- "I'm a C veteran and I want full pointer control and manual memory
  management." → Aura is for you.
- "I hate whitespace-sensitive languages and want my curly braces."
  → Aura is for you.
- "I want to write Python-style code that actually compiles to fast
  native binaries." → Aura is for you.

---

## 2. Design Constraints

These are logical consequences of the axioms that must guide every
syntax decision.

### 2.1 The Guess Test

Before finalizing any syntax, ask: if someone who has never seen Aura
but speaks English tries to write this feature from scratch, what would
they write? If their guess would compile and work correctly at least
some of the time, the syntax passes. If their guess would silently do
something wrong, the syntax fails — regardless of how elegant it is.

Examples of languages that fail the Guess Test badly:

  C:
    int y = x++ + ++x;
    // undefined behavior. Nobody guesses this correctly.

  JavaScript:
    [] + {}  // "[object Object]"
    {} + []  // 0

  Python:
    x = [[0] * 3] * 3
    x[0][0] = 1
    print(x)  // [[1,0,0],[1,0,0],[1,0,0]] — all rows changed.
              // Almost nobody guesses this.

The Guess Test is a formal part of the contributing guidelines. Any new
syntax or behavior must be guessable by a competent English speaker who
has read the axioms but never seen the specific feature.

### 2.2 Structured Natural Language

Aura pseudocode is not arbitrary natural language. It is a controlled
vocabulary of natural-sounding templates with defined slots. The
surface form feels like natural English prose. The underlying structure
is unambiguous, finite, and mechanically translatable.

Example: "prices keeping only the ones where the price is less than 20"
looks like prose but has the hidden template:
  [collection] keeping only the ones where [condition]

The translator only needs to map three things: the phrase
"keeping only the ones where", the surrounding word order, and the
slot positions. The contents of the slots ([collection] and
[condition]) stay flexible.

When readable and structured conflict — structure wins.
But the structured form must still pass the Guess Test.

### 2.3 The Deep Copy Guarantee

Unlike Python, the * operator on collections always performs a deep
copy. No shared references.

  Aura:
    Set grid to [[0] * 3] * 3.
    Set grid[0][0] to 1.   // only row 0 changes

This is an intentional, documented divergence from C and Python.

### 2.4 The Safety Default

When there are two ways to implement a feature — one safe and one
unsafe — the safe way must be the default. The unsafe way must be
explicitly opted into.

Examples:
- Memory: ARC by default, manual requires "new manual"
- SQL: parameterized queries only — string concatenation is a
  compile error
- Null: variables cannot be null unless declared "optional"
- Overflow: panics loudly in debug, defined behavior in release

---

## 3. Lexical Structure

### 3.1 Encoding

All Aura source files are UTF-8. Identifiers may include any Unicode
codepoint including CJK characters and emoji, enabling fully
native-language variable names. The compiler normalizes text to NFC
before lexing.

### 3.2 Statement Termination

Three equivalent forms:

  Newline          — end of line (default, Python style)
  Period + space   — Set x to 10. Set y to 5.
  Semicolon        — x = 10; y = 5

### 3.3 Member Access vs. Statement Separation

The period is context-sensitive, resolved entirely by whitespace:

  object.method()    // no spaces → member access
  object . method()  // spaces around dot → two separate statements

### 3.4 Block Delimiters — Hybrid Scoping

Both indentation-based and brace-based scoping are supported and may
be mixed within the same file.

  Indentation:  colon opens, dedent closes
  Braces:       { opens, } closes
  One-liner:    period, semicolon, or newline closes

### 3.5 Keywords — Case Insensitivity

All reserved keywords are case-insensitive. IF, If, and if are all
the same keyword. Identifiers remain case-sensitive.

Implementation: the lexer calls lowercase() on every token before
comparing it to the keyword table. One line of code.

### 3.6 Comments

  // Single-line comment

  /* Multi-line
     comment */

### 3.7 Multi-Word Identifiers

Function and variable names may contain spaces. The lexer uses a
greedy longest-match algorithm against the known symbol table:

  Define clear the screen() as:
      // ...

  clear the screen.                    // valid call
  clear the screen and then exit.      // call + sequence

The lexer tries the longest possible match first, then backtracks.

### 3.8 String Interpolation

Square bracket interpolation. Full expressions supported. No prefix
character required.

  Say "Hello [name], you are [age] years old."
  Say "The result is [calculate(x) * 2]."

Literal square brackets inside strings: \[ and \]

---

## 4. Reserved Keywords (~22 Core)

Target: beat Lua (22 keywords). Every keyword earns its place.

  Define / As        — all declarations (fn / type / let)
  Is / Set / To      — assignment (=)
  If / Else / Then   — branching (if / else)
  While / Repeat     — looping (while / for)
  Return             — exit function with value (return)
  Try / In case      — error handling (try / catch)
  Package / Import / From  — module system (pkg / import / from)
  Begin / End        — program entry and exit
  Run / Await        — concurrency (run / await)
  And / Or / Not     — boolean logic (&& / || / !)
  A / An / The       — grammar sugar — ignored by compiler
  Thing / [T]        — generic placeholder (any / <T>)
  New                — heap allocation (new)
  Null               — absence of value (null)
  Delete             — manual deallocation (free)
  Shared             — public export (pub)
  Test               — inline test block
  Where              — generic constraints
  Constant           — immutable value (const)
  Yield              — iterator value production (yield)
  Pure               — no side effects annotation (pure)
  Match              — pattern matching (match)

### 4.1 Contextual Keywords (not reserved as identifiers)

  address of, value at   — pointer operations
  cast ... to            — type conversion
  size of                — count / length
  fixed                  — fixed-size array modifier
  manual                 — memory management override
  in background          — concurrency modifier
  exists                 — null check (If x exists:)
  optional               — nullable type modifier
  and so on              — variadic parameters
  by default             — default parameter values
  wrapping               — overflow behavior modifier
  saturating             — overflow behavior modifier

### 4.2 Grammar Sugar Words

A, An, and The are completely ignored by the compiler:

  Set n to a new Node.    // "a" ignored
  Return the result.      // "the" ignored
---

## 5. Syntactic Equivalence Pairs

Every concept in Aura has a pseudocode form and a terse form.
These compile to identical Abstract Syntax Trees.

The aura translate command rewrites source files between styles:

  aura translate --style terse my_file.aura
  aura translate --style pseudocode my_file.aura

Mixed-style files are handled gracefully — only the portions matching
the target style are rewritten.

### Complete Equivalence Table

  Concept               Pseudocode                           Terse
  ─────────────────────────────────────────────────────────────────────
  Assignment            Set x to 10.                         x = 10
  Member access         Object's Property                    Object.Property
  Boolean not           is not null / does not exist         != null / !x
  Increment             Increase x by 1.                     x += 1
  Decrement             Decrease x by 5.                     x -= 5
  Type declaration      x is an integer.                     x: int
  Pointer type          p is a pointer to a Node.            p: *Node
  Dereference read      the value at p / what p points to    *p
  Dereference write     Set the value at p to 42.            *p = 42
  Address of            the address of x                     &x
  Null check            If x exists:                         if x:
  Heap allocation       Set n to a new Node.                 let n = new Node
  Manual allocation     Set n to a new manual Node.          let n = new manual Node
  Cast                  Cast x to an integer.                x as int
  Generic               List of Thing / List of String       List<T> / List<str>
  Fixed array           A fixed list of 10 integers.         int[10]
  Dynamic list          A new list of integers.              List<int>
  Return                Return the result.                   return result
  Package declare       Package math_utils.                  pkg math_utils
  Import                Import networking.                   import networking
  Import alias          Import networking as net.            import networking as net
  Selective import      From networking import connect.      from networking import connect
  Scope indent          Define f(x) as:                      fn f(x) {
  Instantiation         a new Person where Name is "Alice"   Person("Alice")
  Inheritance           Define Student as a type of Person.  type Student : Person
  Constant              Constant PI is 3.14159.              const PI: dec = 3.14159
  Math add              x plus y / x + y                     x + y
  Math multiply         x multiplied by y / x * y            x * y
  Power                 x to the power of y                  x ^ y
  Modulo                x modulo y / x mod y                 x % y
  Pipe                  then                                  |>
  Variadic              numbers and so on                    ...nums: int
  Default param         greeting which is "Hello" by default greeting: str = "Hello"
  Multiple return       Return x, y.                         return x, y
  Destructure           Set a, b to my_function().           let a, b = my_function()
  Anonymous function    a function that takes x and          fn(x) => x * 2
                        returns x * 2
  Enum                  Define Direction as one of:          enum Direction {
                            North, South, East, West.            North, South, East, West }
  Interface             Define something that can            interface Printable {
                            be printed as:
  Representation        Representation: (inside Concept)     fn to_str() -> str
  Inspect               Inspect my_object.                   debug my_object
  Assert                Assert x is greater than 0.          assert x > 0
  Type check            If x is an integer:                  if x is int:
  Visibility            Shared Define calculate(x) as:       pub fn calculate(x)
  Enum access           Direction.North                      Direction.North

---

## 6. Type System

### 6.1 Primitive Types

  Natural Name        Terse Alias    Size              Notes
  ──────────────────────────────────────────────────────────────────
  integer             int            64-bit signed     Default integer
  short integer       int16          16-bit signed     Embedded use
  small integer       int32          32-bit signed     When 64-bit is wasteful
  decimal             dec/float64    64-bit IEEE 754   Default float
  single decimal      float32        32-bit IEEE 754   GPU / performance
  boolean             bool           1-bit (8-aligned) true, false
  character           char           8-bit UTF-8 unit  Single code unit
  string              str            Variable (heap)   UTF-8, immutable default
  byte                u8             8-bit unsigned    Raw memory / I/O
  wrapping integer    int_wrap       64-bit signed     C-style wrapping on overflow
  saturating integer  int_sat        64-bit signed     Clamps to min/max on overflow

### 6.2 Compound Types

  Pseudocode                           Terse           Notes
  ──────────────────────────────────────────────────────────────────────
  A fixed list of 10 integers.         int[10]         Stack-allocated, fixed size
  A new list of integers.              List<int>        Heap-allocated, dynamic
  A map of string to integer.          Map<str, int>   Hash map
  A set of strings.                    Set<str>        Unique values only
  An optional string.                  str?            May be null
  A pointer to a Node.                 *Node           Raw pointer (manual mode)
  A list of list of integers.          int[][]         N-dimensional / nested
  Stack of Thing                       Stack<T>        Generic container
  Result of String or Error            Result<str,Err> Success or failure

### 6.3 Type Inference

  Set x to 10.           // inferred: int
  Set name to "Alice".   // inferred: str
  Set ratio to 3.14.     // inferred: dec
  Set flag to true.      // inferred: bool

  // Explicit (terse)
  x: int = 10
  name: str = "Alice"

### 6.4 Static Typing — No Silent Type Changes

  Set x to 10.
  Set x to "hello".   // ERROR: 'x' was declared as integer on line 1.
                      // Did you mean to create a new variable?
                      // For a flexible type, use: x is a Thing.

### 6.5 Option Types — No Null by Default

  Set name to "Alice".                  // string — can never be null
  Set nickname to an optional string.   // explicitly nullable

  If nickname exists:
      Say "Nickname: [nickname]."
  Else:
      Say "No nickname set."

Using an optional value without checking is a compile-time error.

### 6.6 Result Types

  Define fetch(url which is a string) returns Result of String or Error as:
      // ...

  Set outcome to fetch("https://aura-lang.org").
  Match outcome:
      If it succeeded with content: Say "Got [the size of content] bytes."
      If it failed with error: Say "Failed: [error's message]."

  // Terse
  fn fetch(url: str) -> Result<str, Error> { ... }

  match fetch("https://aura-lang.org") {
      Ok(content) => say "Got [content.len()] bytes."
      Err(e)      => say "Failed: [e.message]"
  }

### 6.7 Generic Constraints — The where Clause

  Define sort(items which is a List of Thing
              where Thing can be compared) as:
      // compiler guarantees every item has a Compare action

  // Terse
  fn sort<T>(items: List<T>) where T: Comparable { ... }

### 6.8 Runtime Type Checking

  If x is an integer:
      Say "[x] is a number."
  If x is a string:
      Say "[x] is text."

  // Terse
  if x is int: ...
  if x is str: ...

---

## 7. Data Architecture — Structure vs. Concept

  Feature          Structure (Data-Oriented)          Concept (Object-Oriented)
  ─────────────────────────────────────────────────────────────────────────────
  Purpose          Named memory layout — pure data    High-level model with behavior
  Methods          No. Functions take struct as arg.  Yes. Implicit self.
  Inheritance      No — flat layout                   Yes — "is a type of"
  Allocation       Typically stack (fast)             Typically heap (ARC-managed)
  Use case         GPU buffers, file headers, tensors UI components, domain models
  Memory cost      Exactly sizeof(fields)             sizeof(fields) + vtable ptr

### 7.1 Structure Example

  // Pseudocode
  Define Header as a structure:
      Version is an integer.
      Size is a decimal.
      Checksum is a byte.

  // Terse
  type Header { version: int, size: dec, checksum: u8 }

Functions that operate on structures are defined externally and
receive the structure as an argument. No implicit self or this.

### 7.2 Concept Example

  // Pseudocode
  Define Person as a concept:
      Name is a string.
      Age is an integer.

      Action Greet:
          Say "Hello, my name is [Name]."

      Action Birthday:
          Increase Age by 1.

  // Terse
  type Person {
      name: str,
      age:  int,

      fn Greet()    { say "Hello, my name is [name]." }
      fn Birthday() { age += 1 }
  }

### 7.3 Instantiation with Named Fields

  // Pseudocode
  Set dev to a new Person where Name is "Alice" and Age is 30.

  // Terse — positional
  let dev = Person("Alice", 30)

  // Terse — named (order-independent)
  let dev = Person(name: "Alice", age: 30)

### 7.4 Inheritance

  // Pseudocode
  Define Student as a type of Person:
      Student ID is an integer.

      Action Study:
          Say "[Name] is studying."   // Name inherited from Person

  // Terse
  type Student : Person {
      student_id: int,
      fn Study() { say "[name] is studying." }
  }

### 7.5 Module Visibility

By default, all definitions are private to the package.
Mark a definition as Shared to export it:

  Shared Define calculate(x) as:
      Return x * x.

  Define helper(x) as:   // private — not accessible outside this package
      Return x + 1.

### 7.6 Generics with Constraints

  Define Stack of Thing as a concept:
      Items is a list of Thing.

      Action Push(item which is a Thing):
          Add item to Items.

      Action Pop:
          If Items is empty: Return null.
          Return the last item of Items.

      Action Is empty:
          Return the size of Items is 0.

  // Usage
  Set number_stack to a new Stack of Integer.
  Push 42 to number_stack.
  Push 17 to number_stack.
  Say Pop from number_stack.   // 17

### 7.7 Interfaces — "Things That Can"

Instead of the word "interface", Aura pseudocode uses natural
descriptions. Any Concept that has the required Actions automatically
satisfies the interface — no explicit "implements" declaration needed
(structural typing, like Go):

  // Defining an interface
  Define something that can be printed as:
      It must have an Action called Print.

  Define something that can be compared as:
      It must have an Action called Compare that returns an integer.

  // Using an interface
  Shared Define log everything(items which is a list of
                               things that can be printed) as:
      Repeat for each item in items:
          item.Print().

  // Terse
  interface Printable { fn Print() }
  interface Comparable { fn Compare() -> int }

  pub fn log_all(items: List<Printable>) {
      for item in items { item.Print() }
  }

### 7.8 Representation and Inspect

Every Concept can define how it appears as a string. If no
Representation is defined, the compiler generates a default one
that shows all field names and values:

  Define Point as a concept:
      X is a decimal.
      Y is a decimal.

      Representation:
          Return "Point([X], [Y])".

  Set p to a new Point where X is 3.0 and Y is 4.0.
  Say p.                      // Point(3.0, 4.0)
  Say "The point is [p]."     // string interpolation works too

  // Debug output — full structure, types, addresses in debug mode
  Inspect p.
  Inspect my_list.

  // Terse
  debug p
  debug my_list

### 7.9 Enumerations

  // Pseudocode
  Define Direction as one of: North, South, East, West.
  Define Status as one of: Pending, Running, Done, Failed.

  Set direction to Direction.North.

  Match direction:
      If North: move up().
      If South: move down().
      If East or West: move sideways().

  // Terse
  enum Direction { North, South, East, West }
  enum Status { Pending, Running, Done, Failed }

  let d = Direction.North
  match d {
      North => move_up()
      South => move_down()
      East | West => move_sideways()
  }

The compiler warns if a Match block does not cover all enum cases
(exhaustiveness checking).

---

## 8. Memory Management

### 8.1 Memory Modes

  Mode              Activation                   Behavior
  ─────────────────────────────────────────────────────────────────────────
  ARC (default)     new Node                     Retain/release inserted by
                                                 compiler. Freed when
                                                 ref-count hits 0. No GC
                                                 pauses.
  Scoped            Open / use block             Freed deterministically at
                                                 end of block.
  Manual            new manual Node              Programmer calls Delete.
                                                 Compiler warns on likely
                                                 leak.
  --mem auto        Compiler flag (default)      Escape analysis picks best
                                                 mode per variable.
  --mem manual      Compiler flag                All allocations must be
                                                 freed. Won't compile if
                                                 leak detected.

### 8.2 Escape Analysis

In --mem auto mode the compiler performs static lifetime analysis.
If a variable never outlives its scope, it may be stack-allocated even
when "new" is used. This optimization is invisible to the programmer
and has zero runtime cost.

### 8.3 Scoped Resources — Open / use

Resources are automatically released when the block exits, even if
an error occurs:

  // Pseudocode
  Open "data.txt" in read mode as my_file:
      Set first_line to my_file.read_line().
      Say "First line: [first_line]."
  // my_file is automatically closed here

  // Terse
  use my_file = open("data.txt", "r") {
      let line = my_file.read_line()
      say line
  }

This pattern works for any resource, not just files:

  Use a database connection to "mydb" as db:
      db.query("SELECT * FROM users").

  Use a mutex lock called data_lock:
      Set shared_counter to shared_counter + 1.

### 8.4 Manual Memory

  Set n to a new manual Node.
  n's Value is 42.
  n's Next is null.

  // ... use n ...

  Delete n.   // required — compile error in --mem manual if omitted

### 8.5 Allocators (Beta)

In Alpha, one implicit heap allocator is used everywhere.
In Beta, a full Zig-inspired allocator system is introduced.

Standard library functions that allocate accept an optional allocator
parameter, defaulting to the heap allocator:

  // Beta pseudocode
  Set my_list to a new list of integers using arena_allocator.

  // Beta terse
  let my_list = List<int>.new(arena_allocator)
  let my_list = List<int>.new()   // uses default heap allocator

Allocator types (Beta):

  ArenaAllocator       — allocate freely, free everything at once.
                         Perfect for: request handlers, parsers.

  PoolAllocator        — pre-allocate N objects of the same type.
                         Perfect for: game entities, connections.

  FixedBufferAllocator — allocate from a fixed stack array. No heap.
                         Perfect for: embedded systems.

  TrackingAllocator    — wraps another allocator, records every
                         allocation. Perfect for: leak detection in
                         tests.

  StackAllocator       — LIFO allocation, extremely fast.
                         Perfect for: temporary scratch space.

  // Arena example
  Use a new arena allocator as arena:
      Set results to a new list of strings using arena.
      Repeat for each item in large_dataset:
          Add process(item) to results.
      Return combine(results).
  // entire arena freed here in one operation

### 8.6 Freestanding Mode (Embedded)

--freestanding disables the heap. The compiler rejects any code that
uses the default heap allocator without an explicit allocator.
ARC and dynamic collections are unavailable. Only stack allocation
and fixed arrays permitted.

Teacher voice error: "You're in freestanding mode, which means there's
no heap. You need to tell me where to get memory from.
Try: using my_fixed_buffer."

---

## 9. Control Flow

### 9.1 Branching

  // Pseudocode
  If temperature is greater than 100:
      Say "Boiling!"
  Else if temperature is greater than 50:
      Say "Warm."
  Else:
      Say "Cold."

  // Terse
  if temp > 100 { say "Boiling!" }
  else if temp > 50 { say "Warm." }
  else { say "Cold." }

### 9.2 Loops

  // While loop
  While x is less than 10:
      Increase x by 1.

  // Repeat N times
  Repeat 5 times:
      Say "Hello!"

  // For-each
  Repeat for each item in my_list:
      Say item.

  // Indexed range
  Repeat for i from 0 to 10:
      Say i.

  // Terse
  while x < 10 { x += 1 }
  for i in 0..10 { say i }
  for item in my_list { say item }

### 9.3 Loop Control

  Stop the loop.   // break
  Go to next.      // continue

### 9.4 Pattern Matching

Pattern matching is switch/case done right. Unlike traditional
switch/case, it matches on structure and type simultaneously,
can destructure (unpack) what it matches, and the compiler enforces
exhaustiveness (warns if a case is missing):

  // Matching on value (like switch/case)
  Match direction:
      If North: move up().
      If South: move down().
      If East or West: move sideways().
      Otherwise: Say "Unknown direction."

  // Matching on type — not possible in switch/case
  Match my_result:
      If it is a string called text: Say "Got text: [text]."
      If it is an error called e: Say "Failed: [e's message]."
      If it is null: Say "Nothing returned."

  // Matching on structure — unpacking simultaneously
  Match point:
      If it is (0, 0): Say "Origin."
      If it is (x, 0): Say "On the x-axis at [x]."
      If it is (0, y): Say "On the y-axis at [y]."
      If it is (x, y): Say "At [x], [y]."

  // Result type matching
  Match fetch("https://aura-lang.org"):
      If it succeeded with content:
          Say "Page length: [the size of content]."
      If it failed with error:
          Say "Error: [error's message]."

  // Terse
  match direction {
      North       => move_up()
      South       => move_down()
      East | West => move_sideways()
      _           => say "Unknown."
  }

### 9.5 Error Handling

  // Pseudocode
  Try to open "config.json" as file:
      Set data to file.read().
      Return data.
  In case of error:
      For FileNotFoundError do:
          Say "The file is missing!"
      For PermissionError do:
          Say "Access denied."
      Otherwise:
          Say "Unknown error."

  // Terse
  try open("config.json") as file {
      return file.read()
  } catch (e) {
      FileNotFoundError: say "Missing file"
      PermissionError:   say "No access"
      _:                 say "Unknown error"
  }

The In case of error block uses the same Match syntax internally.
They are unified — Match is the single pattern-matching construct
used everywhere.

### 9.6 Assertions

Assertions state that something must be true. If not, the program
crashes immediately with a clear message rather than silently
continuing with bad data. Disabled in --release builds.

  Assert x is greater than 0.
  Assert my_list is not empty.
  Assert the size of name is less than 255,
      otherwise say "Name too long".

  // Terse
  assert x > 0
  assert len(my_list) > 0
  assert len(name) < 255, "Name too long"

### 9.7 Concurrency Model

  Model             Analogy                  Use Case
  ──────────────────────────────────────────────────────────────────
  Async / Await     One chef multitasking    I/O-bound: web, files
  Run in background Two chefs, shared kitchen CPU-bound parallel work
  Isolated task     Two separate kitchens    Parallel, no shared state

  // Async I/O
  Set data to await fetch_website("https://aura-lang.org").

  // Background task
  Run calculate_large_sum in the background as my_task.
  If my_task is finished:
      Say "Result: [my_task's result]."

  // Terse
  let data = await fetch_website("https://aura-lang.org")
  run calculate_large_sum as my_task

### 9.8 Native Test Blocks

Test blocks are first-class — no import required.
Run with: aura test my_file.aura
Excluded entirely from --release builds.

  Test "add works correctly":
      Assert add(2, 3) equals 5.
      Assert add(0, 0) equals 0.
      Assert add(-1, 1) equals 0.

  Test "stack returns last item":
      Set s to a new Stack of Integer.
      Push 10 to s.
      Push 20 to s.
      Assert Pop from s equals 20.
      Assert Pop from s equals 10.

---

## 10. Functions

### 10.1 Basic Definition

  // Pseudocode
  Define greet(name) as:
      Say "Hello, [name]!"

  // Terse
  fn greet(name: str) {
      say "Hello, [name]!"
  }

### 10.2 Return Values

  // Pseudocode
  Define square(x which is an integer) as:
      Return x * x.

  // Terse
  fn square(x: int) -> int {
      return x * x
  }

### 10.3 Type Declarations in Parameters

Variable declarations follow name-first order throughout Aura.
This mirrors natural English: "counter, which is an integer"
rather than "integer called counter":

  // Pseudocode — natural English order
  Define process(name which is a string,
                 count which is an integer,
                 ratio which is a decimal) as:
      // ...

  // Terse — name: type
  fn process(name: str, count: int, ratio: dec) { ... }

### 10.4 Default Parameter Values

  // Pseudocode
  Define greet(name which is a string,
               greeting which is a string and is "Hello" by default) as:
      Return "[greeting], [name]."

  greet("Alice").                            // Hello, Alice.
  greet("Alice", greeting is "Goodbye").     // Goodbye, Alice.

  // Terse
  fn greet(name: str, greeting: str = "Hello") -> str {
      return "[greeting], [name]."
  }

  greet("Alice")
  greet("Alice", greeting: "Goodbye")

### 10.5 Named Arguments

Named arguments can be passed in any order:

  // Pseudocode
  Define create user(name which is a string,
                     age which is an integer,
                     admin which is a boolean and is false by default) as:
      // ...

  create user(name is "Alice", age is 30).
  create user(age is 25, name is "Bob", admin is true).

  // Terse
  fn create_user(name: str, age: int, admin: bool = false) { ... }

  create_user(name: "Alice", age: 30)
  create_user(age: 25, name: "Bob", admin: true)

### 10.6 Multiple Return Values

The compiler wraps multiple return values in an anonymous struct
under the hood. The caller can destructure them directly:

  // Pseudocode
  Define min and max(list which is a list of integers) as:
      Return the smallest item of list, the largest item of list.

  Set low, high to min and max(my_numbers).
  Say "Range: [low] to [high]."

  // Terse
  fn min_and_max(list: List<int>) -> (int, int) {
      return list.min(), list.max()
  }

  let low, high = min_and_max(my_numbers)

### 10.7 Variadic Functions

  // Pseudocode
  Define sum of all(numbers and so on) as:
      Set total to 0.
      Repeat for each n in numbers:
          Increase total by n.
      Return total.

  Say sum of all(1, 2, 3, 4, 5).   // 15

  // Terse
  fn sum_of_all(...nums: int) -> int {
      let total = 0
      for n in nums { total += n }
      return total
  }

### 10.8 Pure Functions

A pure function promises it has no side effects — it only reads its
arguments and returns a value. Never modifies global state, never
does I/O. The compiler verifies this and can use it for optimization
and safe automatic parallelization:

  // Pseudocode
  Pure Define square(x which is an integer) as:
      Return x * x.

  // Terse
  pure fn square(x: int) -> int { return x * x }

### 10.9 Closures — Anonymous Functions

  // Pseudocode
  Set double to a function that takes x and returns x * 2.
  Set greet  to a function that takes name and
                says "Hello, [name].".

  Say double(5).     // 10
  greet("Alice").    // Hello, Alice.

  // Terse
  let double = fn(x) => x * 2
  let greet  = fn(name) { say "Hello, [name]." }

Closures capture variables from their surrounding scope:

  // Pseudocode
  Set multiplier to 3.
  Set triple to a function that takes x and returns x * multiplier.
  Say triple(7).   // 21

  // Terse
  let multiplier = 3
  let triple = fn(x) => x * multiplier

### 10.10 Tail Call Optimization

Aura performs tail call optimization (TCO) automatically when a
recursive function calls itself as its last action. This compiles the
recursion into a loop, preventing stack overflow for deep recursion:

  // This will not stack overflow even for 1,000,000 items
  Define sum of list(items,
                     running total which is 0 by default) as:
      If items is empty: Return running total.
      Return sum of list(the rest of items,
                         running total + the first item of items).

  Say sum of list([1, 2, 3, 4, 5]).   // 15

### 10.11 Function Visibility

  Shared Define calculate(x) as:     // exported — accessible outside package
      Return x * x.

  Define helper(x) as:               // private — package-internal only
      Return x + 1.

---

## 11. Functional Programming

The core idea of functional programming is simple: functions are
values, just like numbers or strings. You can store a function in a
variable, pass a function to another function, and return a function
from a function. Everything else follows from that.

Python programmers already use this constantly:
  sorted(my_list, key=lambda x: x.age)
is functional programming. You're passing a function to another
function. Aura formalizes and extends this naturally.

### 11.1 The Pipe Operator

Instead of nesting function calls inside each other (which reads
inside-out), the pipe operator chains them left-to-right (which
reads like a sentence):

  // Without pipe — reads inside-out, unnatural
  Say format(sort(filter(my_list, is_positive)), as_table).

  // With pipe — reads left to right
  Set result to my_list
      then filter with is_positive
      then sort
      then format as table.
  Say result.

  // Terse
  let result = my_list
      |> filter(is_positive)
      |> sort()
      |> format(as_table)

The pipe operator passes the result of the left side as the first
argument to the right side. It is purely syntactic sugar — the
compiler rearranges the function calls and produces identical code.

### 11.2 Map — Transform Every Item

Apply a function to every item in a collection, producing a new
collection of the same size:

  Set prices to [10.00, 25.50, 8.75, 42.00].

  // Pseudocode
  Set doubled to prices with each one multiplied by 2.
  Set discounted to prices with 10 percent taken off each one.
  Set as text to prices transformed by casting each to a string.

  // With a named function
  Pure Define apply discount(price) as:
      Return price * 0.9.

  Set discounted to prices transformed by apply discount.

  // Terse
  let doubled    = prices.map(fn(p) => p * 2)
  let discounted = prices.map(fn(p) => p * 0.9)
  let as_text    = prices.map(fn(p) => p as str)

### 11.3 Filter — Keep Only Matching Items

  Set prices to [10.00, 25.50, 8.75, 42.00].

  // Pseudocode
  Set affordable to prices keeping only the ones
                    where the price is less than 20.

  Set positives to numbers keeping only the ones
                   where the number is greater than 0.

  // Terse
  let affordable = prices.filter(fn(p) => p < 20)
  let positives  = numbers.filter(fn(n) => n > 0)

### 11.4 Reduce — Collapse to a Single Value

  Set prices to [10.00, 25.50, 8.75, 42.00].

  // Pseudocode
  Set total   to prices added together.
  Set highest to prices where we keep the largest.
  Set product to prices multiplied together.

  // Terse
  let total   = prices.reduce(fn(acc, p) => acc + p, 0)
  let highest = prices.reduce(fn(acc, p) => if p > acc { p } else { acc }, 0)

### 11.5 Any and All

  // Pseudocode
  If any price in prices is greater than 40:
      Say "Some items are expensive."

  If all prices in prices are less than 100:
      Say "Everything is under budget."

  // Terse
  if prices.any(fn(p) => p > 40): say "Some expensive."
  if prices.all(fn(p) => p < 100): say "All under budget."

### 11.6 Chained Operations — Full Example

  Set prices to [10.00, 25.50, 8.75, 42.00, 3.99].

  // Pseudocode — reads like plain English
  Set checkout total to prices
      then keep only the ones where the price is less than 20
      then take 10 percent off each one
      then add them all together.

  Say "Your total is [checkout total]."

  // Terse
  let checkout_total = prices
      |> filter(fn(p) => p < 20)
      |> map(fn(p) => p * 0.9)
      |> reduce(fn(acc, p) => acc + p, 0.0)

  say "Your total is [checkout_total]."

### 11.7 Iterators and Lazy Evaluation (Beta)

Instead of building a full list in memory, produce values one at a
time. Memory-efficient for large or infinite sequences:

  // Pseudocode
  Define count up from(start) as a sequence:
      Set n to start.
      While true:
          Yield n.
          Increase n by 1.

  Repeat for each number in count up from(1):
      If number is greater than 10: Stop the loop.
      Say number.

  // Terse
  fn count_up_from(start: int) -> Sequence<int> {
      let n = start
      while true { yield n; n += 1 }
  }

  for number in count_up_from(1) {
      if number > 10: break
      say number
  }

### 11.8 Function Composition (Beta)

Combining two functions into a new function that applies them
in sequence:

  // Pseudocode
  Set double then add one to a function that
      first doubles and then adds 1.

  Say double then add one(5).   // 11

  // Terse
  let double_then_add_one = fn(x) => double(x) + 1
  // or using compose:
  let double_then_add_one = compose(double, add_one)

---

## 12. Strings

### 12.1 String Interpolation (recap)

  Say "Hello [name], you are [age] years old."
  Say "The result is [calculate(x) * 2]."
  Say "Pi is approximately [Math.PI]."

Full expressions are supported inside [ ].
Literal brackets: \[ and \]

### 12.2 Standard String Operations

  // Case
  Set upper to name in uppercase.
  Set lower to name in lowercase.
  Set title to name in title case.

  // Whitespace
  Set trimmed to name without leading or trailing spaces.
  Set left trimmed to name without leading spaces.
  Set right trimmed to name without trailing spaces.

  // Splitting and joining
  Set parts to sentence split by ", ".
  Set parts to sentence split by spaces.
  Set joined to parts joined with " and ".
  Set joined to parts joined with no separator.

  // Replacing
  Set cleaned to text with "bad word" replaced by "***".
  Set normalized to path with "\" replaced by "/".

  // Checking
  If name starts with "A":
  If name ends with "ing":
  If name contains "hello":
  If name does not contain "error":

  // Size
  Set length to the size of name.
  Set length to the number of characters in name.

  // Padding
  Set padded to name padded to 20 characters with spaces.
  Set padded to name padded to 20 characters with "0" on the left.

  // Terse equivalents
  name.upper()
  name.lower()
  name.trim()
  sentence.split(", ")
  parts.join(" and ")
  text.replace("bad", "***")
  name.starts_with("A")
  name.ends_with("ing")
  name.contains("hello")
  name.len()
  name.pad_right(20, " ")
  name.pad_left(20, "0")

### 12.3 Slicing

  // Pseudocode
  Set first three to the first 3 items of my_list.
  Set last two to the last 2 items of my_list.
  Set every other to every 2nd item of my_list.
  Set middle to items 2 through 5 of my_list.

  // Same syntax for strings
  Set first three chars to the first 3 characters of name.
  Set last two chars to the last 2 characters of name.

  // Terse (Python-compatible syntax)
  my_list[0:3]
  my_list[-2:]
  my_list[::2]
  my_list[1:5]
  name[0:3]

### 12.4 The Representation Action

Every Concept can define how it appears as a string:

  Define Point as a concept:
      X is a decimal.
      Y is a decimal.

      Representation:
          Return "Point([X], [Y])".

  Set p to a new Point where X is 3.0 and Y is 4.0.
  Say p.                      // Point(3.0, 4.0)
  Say "The point is [p]."     // works in interpolation too

If no Representation is defined, the compiler generates a default:
  // Auto-generated for Point if no Representation defined:
  // "Point { X: 3.0, Y: 4.0 }"

---

## 13. Numbers and Operators

### 13.1 Operator Precedence

From highest to lowest priority. When in doubt, use parentheses —
the compiler will never complain about extra parentheses:

  1.  ( )                      Parentheses — always evaluated first
  2.  x's y  /  x.y            Member access
  3.  f(x)  /  list[i]         Function calls, indexing
  4.  unary -,  not / !        Negation, logical not
  5.  ^  or  **                Exponentiation (right-associative)
  6.  *  /  mod / %            Multiply, divide, modulo
  7.  +  -                     Add, subtract
  8.  <  >  <=  >=             Comparisons
  9.  is  is not  ==  !=       Equality
  10. and / &&                 Logical and
  11. or / ||                  Logical or
  12. |>  then                 Pipe (lowest — always last)

This matches Python's operator precedence. Existing Python intuition
transfers directly.

### 13.2 Chained Comparisons

Comparisons can be chained, as in mathematics:

  If 0 < x < 10:
      Say "x is between 0 and 10."

  If 0 <= age <= 120:
      Say "Plausible age."

  // Terse — same syntax
  if 0 < x < 10: say "in range."

This is exactly how a mathematician or non-programmer would write it.
Most languages do not support this and force: x > 0 and x < 10.
Aura supports both forms.

### 13.3 Integer Overflow Policy

Default behavior:
- Debug builds: panic immediately with a clear message showing
  the value and operation that caused the overflow.
- Release builds: defined wrapping behavior (like Rust's
  release mode).

Explicit overflow types for when you need specific behavior:

  // Wrapping — C-style, wraps around on overflow
  Set x to a wrapping integer.
  Set x to 2147483647.
  Increase x by 1.   // x is now -2147483648 (wraps)

  // Saturating — clamps to min/max, never wraps
  Set x to a saturating integer.
  Set x to 2147483647.
  Increase x by 1.   // x stays at 2147483647 (saturates)

  // Terse
  let x: int_wrap = 2147483647
  let x: int_sat  = 2147483647

### 13.4 Mathematical Operators

  // Both forms are valid — they compile identically
  x plus y          ==  x + y
  x minus y         ==  x - y
  x multiplied by y ==  x * y
  x divided by y    ==  x / y
  x modulo y        ==  x mod y  ==  x % y
  x to the power of y  ==  x ^ y  ==  x ** y

  // Common math shorthands
  Increase x by 1.      // x += 1
  Decrease x by 1.      // x -= 1
  Double x.             // x *= 2
  Halve x.              // x /= 2

### 13.5 The ^ Disambiguation

  ^   means exponentiation:  2 ^ 8  =  256
  xor means bitwise XOR:     x xor y

This resolves the ambiguity present in many languages where ^ is
used for both. A non-programmer seeing 2 ^ 8 immediately guesses
"two to the power of eight" — which is correct in Aura.

---

## 14. Module System

### 14.1 Package Declaration

Package identity is defined by the Package declaration in the source
file, never by folder name or file name. Two files with
"Package geometry" in the same project are part of the same package
regardless of where they live on disk:

  Package math_utils.   // pseudocode
  pkg math_utils        // terse

### 14.2 Import Forms

  Import networking.                       // networking.connect()
  Import networking as net.                // net.connect()
  From networking import connect.          // connect()
  From networking import connect, listen.  // connect(), listen()
  Adopt networking.                        // all symbols in scope
                                           // (explicit wildcard)

Wildcard import (from package import *) is not supported.
Use Adopt for equivalent behavior — it makes the choice visible
and intentional. The compiler warns when Adopt causes a naming
collision.

### 14.3 Entry Point

  Package main.

  Import Aura.IO.

  Define greet(name) as:
      Say "Hello, [name]!"

  Begin:
      If the command line has arguments:
          greet(the command line's first argument).
      Else:
          greet("World").
  End.

### 14.4 Command-Line Arguments

  Begin with arguments:
      If the argument count is less than 2:
          Say "Usage: my_program <name>"
          Stop the program.
      Set name to the first argument.
      greet(name).
  End.

For flag-based CLI programs (--name Alice --verbose), use
Aura.System which provides a full argument parser. The language
itself handles positional arguments; the library handles flags.

### 14.5 Global Variable Disambiguation

  Package app.

  Set Config to load_config().    // package-level global

  Define run_server() as:
      Set Config to "local".        // WARNING: shadows package Config
      Set app's Config to "local".  // unambiguous global reference

### 14.6 Circular Imports

Circular imports (Package A imports Package B which imports Package A)
are forbidden and detected at compile time. The error message names
the full cycle so it can be resolved.

### 14.7 The build.aura File

For projects that need explicit dependency control instead of
APM auto-fetch:

  Package build.

  Define configuration as:
      Set name to "my_app".
      Set version to "1.0.0".
      Set entry point to "main.aura".
      Set output to "my_app".

      // Compilation behavior
      Set optimization to release.      // debug, release, or size
      Set tree shaking to aggressive.   // default: true
      Set link time optimization to true.

      // Dependencies
      Add dependency "cool-math" at version 2.4.
      Add dependency "github.com/user/pkg" at commit "a3f9b2".

      // Compiler flags
      Add flag "--mem manual".
      Add flag "--freestanding".

      // Targets
      Add target "linux-x86_64".
      Add target "windows-x86_64".
      Add target "embedded-arm-cortex-m4".

---

## 15. Multilingual Support — The Babel Fish Layer

Aura's multilingual system is implemented as a preprocessing stage
that runs before the parser. Source code in any locale is converted
to canonical English terse Aura first. The parser only ever sees
canonical English terse — it is language-agnostic.

### 15.1 The Two-Stage Preprocessing Pipeline

Stage 1 and Stage 2 run in parallel across all files before any
file reaches the parser. This cleanly separates translation bugs
from grammar bugs.

  Stage 1: Locale Translator
    Input:  source file in any language, any style
    Action: applies locale JSON map
            replaces translated keywords with canonical English
            applies grammar transformation rules
            (e.g., reorders possessives:
             "o status da resposta" → "response.status")
    Output: English pseudocode

  Stage 2: Style Normalizer
    Input:  English pseudocode (mixed or pure)
    Action: symbol collection pass (first pass — builds symbol table
            of all defined function and variable names, needed for
            multi-word identifier resolution)
            normalization pass (second pass — converts all pseudocode
            constructs to canonical English terse)
    Output: canonical English terse Aura

  Stage 3: Lexer
    Input:  canonical English terse
    Action: greedy longest-match tokenization
    Output: token stream

  Stage 4: Parser
    Input:  token stream
    Action: builds AST
    Output: Abstract Syntax Tree

  Stage 5: Semantic Analysis
    Input:  AST
    Action: type checking, null safety, scope resolution,
            exhaustiveness checking
    Output: typed AST

  Stage 6: Code Generation
    Input:  typed AST
    Action: Alpha → C code, Beta+ → LLVM IR
    Output: compiled binary (via GCC/Clang)

All stages run in parallel across files. Each stage's output is
independent of other files at that stage, enabling full
parallelization.

### 15.2 Why Two Stages Instead of One

Separation of concerns:
- A bug in Stage 1 is a translation bug. Fix the locale JSON.
- A bug in Stage 2 is a normalization bug. Fix the style rules.
- A bug in Stage 3/4 is a grammar bug. Fix the parser.
These will never be confused with each other.

Testability:
- Stage 1 can be tested by comparing input Portuguese source to
  expected English pseudocode output, with no compiler involved.
- Stage 2 can be tested by comparing pseudocode to expected terse
  output, with no locale system involved.

### 15.3 The Symbol Collection Pass (Why It's Needed)

Multi-word identifiers create a chicken-and-egg problem:

  // Is "clear the screen" a function call or three words?
  clear the screen and then exit.

The normalizer needs to know that "clear the screen" is a defined
function name before it can correctly tokenize this line. The symbol
collection pass (first pass of Stage 2) reads all definitions across
all files and builds the symbol table. The normalization pass (second
pass) then has the full symbol table available.

### 15.4 Locale JSON Structure

  {
    "language": "pt_BR",
    "word_order": "SVO",

    "keywords": {
      "DEFINE":    ["Defina", "Criar"],
      "PACKAGE":   ["Pacote"],
      "IMPORT":    ["Importe"],
      "AS":        ["como"],
      "SET":       ["Atribua", "Mude", "Configure"],
      "IS":        ["é", "e"],
      "TO":        ["para", "a"],
      "IF":        ["Se"],
      "ELSE":      ["Senão", "Caso contrário"],
      "WHILE":     ["Enquanto"],
      "REPEAT":    ["Repita"],
      "RETURN":    ["Retorne"],
      "SAY":       ["Diga", "Escreva"],
      "TRY":       ["Tente"],
      "IN_CASE":   ["Em caso de erro"],
      "AND":       ["e"],
      "OR":        ["ou"],
      "NOT":       ["não"],
      "NULL":      ["nulo", "nula"],
      "NEW":       ["novo", "nova"],
      "TRUE":      ["verdadeiro", "verdadeira"],
      "FALSE":     ["falso", "falsa"],
      "CONSTANT":  ["Constante"],
      "SHARED":    ["Compartilhado", "Compartilhada"],
      "TEST":      ["Teste"],
      "MATCH":     ["Corresponda", "Verifique"],
      "YIELD":     ["Produza"],
      "PURE":      ["Pura", "Puro"],
      "BEGIN":     ["Início", "Comece"],
      "END":       ["Fim", "Termine"]
    },

    "possessive_rules": {
      "pattern":   "[Attribute] d[o|a] [Concept]",
      "transform": "Concept.Attribute",
      "examples": [
        {"input": "o status da resposta",
         "output": "response.status"},
        {"input": "o valor do nó",
         "output": "node.value"}
      ]
    },

    "gender_rules": {
      "strategy":          "suffix_match",
      "masculine_suffix":  "o",
      "feminine_suffix":   "a",
      "note": "Handles ~95% of programmer-chosen variable names.
               Edge cases fall back to terse syntax."
    },

    "sequence_conjunctions": ["e então", "depois"],

    "articles": ["o", "a", "os", "as", "um", "uma"]
  }

### 15.5 Portuguese Source Example

The same Binary Search Tree insertion function written in
Portuguese natural-style Aura:

  Pacote colecoes.

  Defina No como uma estrutura:
      Valor e um inteiro.
      Esquerda e um ponteiro para um No.
      Direita e um ponteiro para um No.

  Defina inserir(raiz que e um ponteiro para um No, val) como:
      Se raiz nao existe:
          Retorne criar no(val).
      Se val e menor que o valor do raiz:
          A esquerda do raiz e inserir(a esquerda do raiz, val).
      Senao se val e maior que o valor do raiz:
          A direita do raiz e inserir(a direita do raiz, val).
      Retorne raiz.

Variable names (raiz, val, No) remain in Portuguese throughout
compilation. Only keywords are translated to canonical English.

After Stage 1, this becomes English pseudocode.
After Stage 2, it becomes canonical English terse.
The parser never sees Portuguese.

### 15.6 Word Order Support

  Order  Example Languages              Parser Strategy
  ──────────────────────────────────────────────────────────────────
  SVO    English, Portuguese, French,   Default. Verb in middle.
         Spanish, Italian
  SOV    Japanese, Korean, Hindi,       Verb at end; entities
         Turkish, Latin                 precede it.
  VSO    Irish, Arabic, Welsh,          First token is always the
         Filipino, Classical Hebrew     action verb.

v1.0 launch locales: English and Portuguese (creator's native
language). All word orders are supported architecturally but
translation quality depends on contributed locale files.

### 15.7 The Translation Export Tool

APM can export a symbol map to assist translators:

  aura --export-symbols my_package --format json

Output:
  {
    "original_language": "en",
    "symbols": {
      "scrape and analyze": "",
      "fetch website": "",
      "results": "",
      "create node": ""
    }
  }

The translator fills in the blanks, saves as my_package.pt.json,
and the package becomes natively readable by Portuguese developers.

### 15.8 The Linguist Problem and Resources

The hardest part of the multilingual system is not the programming —
it is defining "Structured Natural Language" templates that are both
readable in every target language and unambiguous enough to parse.

Key resource: the Universal Dependencies project
(universaldependencies.org) provides annotated grammatical structure
for 100+ languages in a consistent format. The Aura locale grammar
should draw from this when specifying transformation rules.

When ready to add a new language, consult someone in computational
linguistics — the intersection of linguistics and programming.
This is a consulting engagement, not a hire.

---

## 16. Compiler Architecture and Roadmap

### 16.1 Implementation Phases

  Phase       Written In           Target and Goals
  ──────────────────────────────────────────────────────────────────────
  Alpha       C (Flex + Bison)     Aura → C. GCC/MSVC compiles to
                                   native binary. Focus: grammar
                                   correctness, AST, translator layer.
                                   C and C++ interop automatic.
                                   Fortran via GCC.

  Beta        Aura (self-hosted)   Aura → LLVM IR. Clang generates
                                   native binary. Focus: performance,
                                   full type system, ARC runtime,
                                   APM, allocator system.
                                   Self-hosting milestone.

  1.0-stable  Aura                 Aura → Assembly. System assembler
                                   and linker. Focus: full
                                   independence, embedded targets,
                                   complete bootstrapping.

### 16.2 Why C First — Not LLVM IR Directly

- C is the universal assembly language. Every target that runs LLVM
  also has a C compiler.
- Emitting invalid C produces readable errors in a language you know.
  Emitting invalid LLVM IR produces cryptic diagnostics.
- GCC's 40+ years of optimization are free during Alpha.
- C and C++ interop is trivially correct when your output is C.
- Prior art: Nim, Haxe, and early Kotlin all used this strategy.

### 16.3 Why Not C++ as the Alpha Target

C++ is tempting because it provides RAII (maps to scoped blocks),
templates (maps to generics), and std::string (maps to Aura strings).
However: C++ compile times are high, the ABI is complex, and
debugging emitted C++ is significantly harder than C. Revisit in
Beta if the ARC runtime needs it.

### 16.4 The Alpha Compilation Pipeline

  [Source: .aura file]
         |
    [Stage 1: Translator]  ← locale JSON
         |                   (Portuguese → English pseudocode)
    [Stage 2: Normalizer]  ← symbol table
         |                   (pseudocode → canonical English terse)
    [Stage 3: Lexer]       ← Flex
         |                   (greedy multi-word token matching)
    [Stage 4: Parser]      ← Bison
         |                   (recursive descent / PEG grammar)
    [AST Builder]
         |
    [Semantic Analyzer]    ← type checking, null safety,
         |                   scope resolution,
         |                   exhaustiveness checking
    [C Emitter]            ← walks AST, writes _aura_temp.c
         |
    [GCC / MSVC]           ← invoked by compiler driver
         |
    [Native Executable]

### 16.5 Dead Code Elimination and Tree Shaking

The compiler traces the dependency graph from Begin outward, marks
everything reachable, and excludes everything that is not.

  Your code imports calculate() from library A.
  calculate() calls sum() and propagate() internally.
  sum() and propagate() call nothing outside themselves.
  Everything else in library A is unreachable.

  Result: only calculate(), sum(), and propagate() appear in your
  binary. The rest of library A is never compiled.

This is called whole-program optimization or LTO (link-time
optimization). LLVM supports this natively and it is enabled by
default in --release builds.

For foreign libraries (a compiled Rust crate, a pre-compiled C
library), tree-shaking cannot reach inside — you get whatever
their compiler produced. This is documented clearly so users
understand why "Import raylib" produces a larger binary than
"Import aura_geometry".

### 16.6 Bootstrapping

Alpha: compiler written in C.
Once Alpha can compile Aura programs reliably, rewrite the compiler
in Aura for Beta. Beta achieving self-hosting is the language's
coming-of-age moment. After Beta is self-hosting, the C compiler
is retired.

### 16.7 The Greedy Multi-Word Identifier Lexer

  // Symbol table contains: "clear the screen", "clear"
  // Input: "clear the screen and then exit"

  Lexer tries: "clear the screen and then exit" → no match
  Lexer tries: "clear the screen and then"      → no match
  Lexer tries: "clear the screen"               → MATCH (SYMBOL)
  Lexer continues: "and then"                   → AND_THEN keyword
  Lexer continues: "exit"                       → SYMBOL

Symbols are sorted by length (longest first) in the symbol table.
The lexer always tries the longest match before backtracking.

### 16.8 CLI Reference

  aura my_program.aura                    compile to native executable
  aura run my_program.aura                compile and immediately run
  aura test my_file.aura                  run all Test blocks in file
  aura translate --style terse f.aura     rewrite file in terse style
  aura translate --style pseudocode f.aura  rewrite in pseudocode style
  aura fmt my_file.aura                   format source file
  aura check my_file.aura                 type-check only, no binary
  aura lsp                                start language server (Beta)
  aura --lang pt_BR my_program.aura       compile Portuguese source
  aura --mem manual my_program.aura       strict manual memory mode
  aura --mem auto my_program.aura         smart auto memory (default)
  aura --voice teacher my_program.aura    supportive errors (default)
  aura --voice formal my_program.aura     strict concise errors
  aura --voice ramsay my_program.aura     Gordon Ramsay errors
  aura --voice sarcastic my_program.aura  witty dry errors
  aura --release my_program.aura          optimized release build
  aura --debug my_program.aura            debug build with traceback
  aura --size my_program.aura             optimize for binary size
  aura --freestanding my_program.aura     embedded / no-heap mode
  aura --strict-versions my_program.aura  require pinned versions
  aura --export-symbols pkg --format json generate translation map

---

## 17. Aura Package Manager (APM)

APM is designed around Axiom 1: the package manager should never
get in the way. It is proactive, not reactive.

### 17.1 Package Identity

Package identity is defined by the Package declaration in source,
never by folder name or file name. This decouples logical
organization from physical organization.

### 17.2 The libs Folder

When APM downloads a dependency it goes into ./libs/package_name/.
This is local to the project (like node_modules in JavaScript).
The libs folder is managed entirely by APM and should generally
not be committed to version control.

A .aura_lock file records the exact resolved versions for
reproducible builds. The lock file should be committed to version
control:

  // .aura_lock — auto-generated, commit this file
  package: cool-math
  version: 2.4.1
  source: aura-packages.org
  checksum: sha256:a3f9...

  package: http
  version: 1.0.3
  source: aura-packages.org
  checksum: sha256:7bc2...

### 17.3 Auto-Fetch During Compilation

When the compiler encounters an import not present in ./libs/,
it automatically pauses, invokes APM, downloads the package,
and resumes compilation. No separate install step required for
exploratory or rapid-prototype work.

In --strict mode or CI environments, auto-fetch is disabled.
The compiler errors if a dependency is missing, requiring explicit
"apm install" first. This prevents surprise network calls in
production build pipelines.

### 17.4 API-Aware Version Resolution

APM performs static analysis of how the imported library is used.
If the functions called exist in version 2 but were removed in
version 3, APM downloads version 2 automatically. The user never
needs to specify a version number for casual work.

This is the default behavior. In --strict-versions mode all
versions must be pinned in build.aura.

### 17.5 Standalone APM Commands

  apm install cool-math             latest compatible version
  apm install cool-math@2.4         specific version
  apm install github.com/user/pkg   from git repository
  apm remove cool-math              remove package
  apm update                        update all to latest compatible
  apm list                          show installed packages
  apm search tensor                 search package registry
  apm publish                       publish package to registry

### 17.6 Polyglot Dependencies

C: first-class. Types auto-converted. Zero boilerplate.

Other LLVM languages (Rust, Zig, Swift): APM downloads the
required compiler and auto-generates an Aura wrapper library.
Types not in the C ABI require explicit casts in the wrapper.

  Import raylib.           // C — fully automatic, no boilerplate
  Import my_rust_crate.    // APM fetches Rust compiler +
                           // auto-generated Aura wrapper

### 17.7 The build.aura File

See Section 14.7 for full build.aura format.

---

## 18. Standard Library — Full Plan

The standard library is intentionally minimal in the core language.
All high-level functionality is a library call, not a keyword.
This keeps the grammar specification small and allows the stdlib
to evolve independently of the language spec.

### 18.1 Alpha — Ships With the Compiler

These five modules are the absolute minimum for Aura to be usable
by real people on day one. No Alpha release without all five.

─────────────────────────────────────────────────────────────────
Aura.IO — Files, Console, Paths
─────────────────────────────────────────────────────────────────
File operations:
  Open "data.txt" in read mode as f:
  Open "output.txt" in write mode as f:
  Open "log.txt" in append mode as f:
  Set line to f.read_line().
  Set all_text to f.read_all().
  Set all_lines to f.read_all_lines().
  f.write("hello").
  f.write_line("hello").

Console:
  Say "Hello."              // stdout with newline
  Ask "Name?" and set n.    // stdin readline
  Write to error "oops."    // stderr

Paths:
  Set full to path join("folder", "file.txt").
  Set name to path name of "/home/user/file.txt".   // "file.txt"
  Set dir  to path directory of "/home/user/file.txt". // "/home/user"
  Set ext  to path extension of "file.txt".         // ".txt"
  If path exists "data.txt":
  If path is a file "data.txt":
  If path is a directory "src":
  Set files to list files in "src".
  Set files to list files in "src" matching "*.aura".
  Create directory "output".
  Delete file "temp.txt".
  Copy file "a.txt" to "b.txt".
  Move file "a.txt" to "b.txt".

─────────────────────────────────────────────────────────────────
Aura.Text — String Operations
─────────────────────────────────────────────────────────────────
All standard string operations listed in Section 12.2.

Additional:
  Set n to parse integer from "42".
  Set x to parse decimal from "3.14".
  If "42" is a valid integer:
  Set encoded to encode "hello" as base64.
  Set decoded to decode base64 "aGVsbG8=".
  Set hex to encode 255 as hex.          // "ff"
  Set n to parse hex "ff".               // 255

─────────────────────────────────────────────────────────────────
Aura.Collections — Data Structures
─────────────────────────────────────────────────────────────────
List (dynamic array):
  Set items to a new list of integers.
  Add 42 to items.
  Add all of other_list to items.
  Remove 42 from items.
  Remove the item at position 2 from items.
  Set x to items[0].
  Set x to the first item of items.
  Set x to the last item of items.
  Set sub to items[1:4].
  Sort items.
  Sort items by each item's age.
  Sort items in reverse.
  Reverse items.
  If items contains 42:
  If items is empty:
  Set n to the size of items.
  Set pos to the position of 42 in items.
  Set copy to a copy of items.

Map (hash map):
  Set scores to a new map of string to integer.
  Set scores["Alice"] to 100.
  Set x to scores["Alice"].
  If scores contains key "Alice":
  Remove "Alice" from scores.
  Set keys to the keys of scores.
  Set values to the values of scores.
  Repeat for each key, value in scores:

Set (unique values):
  Set unique to a new set of strings.
  Add "hello" to unique.
  If unique contains "hello":
  Set intersection to unique intersected with other_set.
  Set union to unique united with other_set.
  Set diff to unique minus other_set.

Queue (first in, first out):
  Set q to a new queue of strings.
  Enqueue "hello" to q.
  Set item to dequeue from q.

Stack (last in, first out):
  Set s to a new stack of integers.
  Push 42 to s.
  Set top to Pop from s.
  Set top to Peek at s.    // look without removing

─────────────────────────────────────────────────────────────────
Aura.Math.Core — Basic Mathematics
─────────────────────────────────────────────────────────────────
Constants:
  Math.PI        // 3.14159265358979...
  Math.E         // 2.71828182845904...
  Math.TAU       // 6.28318530717958... (2 * PI)
  Math.INFINITY
  Math.NAN

Basic operations:
  the absolute value of x
  the square root of x
  the cube root of x
  x rounded to 2 decimal places
  x rounded down        // floor
  x rounded up          // ceiling
  the minimum of x and y
  the maximum of x and y
  x clamped between 0 and 100
  x to the power of y   // also x ^ y

Trigonometry:
  Math.sin(angle)     Math.asin(x)
  Math.cos(angle)     Math.acos(x)
  Math.tan(angle)     Math.atan(x)
  Math.atan2(y, x)
  Math.degrees to radians(180)    // PI
  Math.radians to degrees(PI)     // 180

Logarithms:
  Math.log(x)          // natural log
  Math.log2(x)
  Math.log10(x)
  Math.exp(x)          // e^x

Random numbers:
  Set n to a random integer between 1 and 10.
  Set x to a random decimal between 0.0 and 1.0.
  Set item to a random item from my_list.
  Shuffle my_list.
  Seed the random number generator with 42.

─────────────────────────────────────────────────────────────────
Aura.Error — Error Types
─────────────────────────────────────────────────────────────────
Base type: Error
  error's message
  error's stack trace
  error's cause        // the error that caused this one

Built-in error types (all extend Error):
  FileNotFoundError
  PermissionError
  NetworkError
  TimeoutError
  ValueError           // wrong value for the context
  TypeError            // wrong type for the operation
  IndexError           // index out of bounds
  KeyError             // key not in map
  OverflowError
  DivisionByZeroError
  NullError            // attempted use of null value
  ParseError           // failed to parse a string

Creating custom errors:
  Define DatabaseError as a type of Error:
      Table name is a string.

      Representation:
          Return "DatabaseError on table [Table name]: [message]".

### 18.2 Beta — Once Language is Self-Hosting

─────────────────────────────────────────────────────────────────
Aura.Net — Networking
─────────────────────────────────────────────────────────────────
HTTP client (fully await-compatible):
  Set response to await Aura.Net.get("https://example.com").
  Set response to await Aura.Net.post("https://api.example.com",
      body is "{\"key\": \"value\"}",
      headers is ["Content-Type: application/json"]).
  response's status        // 200, 404, etc.
  response's body          // string
  response's headers       // map of string to string

Sockets, WebSocket, DNS lookup. Minimal HTTP server.

─────────────────────────────────────────────────────────────────
Aura.JSON
─────────────────────────────────────────────────────────────────
  Set data to parse JSON from response's body.
  Set json_text to convert data to JSON.
  Set json_text to convert data to JSON indented by 2 spaces.
  Set name to data["name"].
  Set age to data["age"] as an integer.

─────────────────────────────────────────────────────────────────
Aura.Concurrency
─────────────────────────────────────────────────────────────────
Mutex, semaphore, channels, atomic operations, thread pool.

  Use a mutex lock called data_lock:
      Set shared_counter to shared_counter + 1.

  Set channel to a new channel of strings.
  Send "hello" through channel.
  Set message to receive from channel.

─────────────────────────────────────────────────────────────────
Aura.Math.Tensor — The NumPy Equivalent
─────────────────────────────────────────────────────────────────
N-dimensional arrays with SIMD-optimized operations.

  Set matrix_a to a new Tensor of [[1, 2], [3, 4]].
  Set matrix_b to a new Tensor of [[5, 6], [7, 8]].
  Set result to matrix_a multiplied by matrix_b.

  Set t to a Tensor of shape [3, 4, 5] filled with zeros.
  Set row to t[0].              // first row
  Set col to t[:, 1].           // second column, all rows
  Set t to t reshaped to [60].  // flatten
  Set t to t transposed.
  Set mean to the mean of t along axis 0.
  Set total to the sum of t.

─────────────────────────────────────────────────────────────────
Aura.Test — Full Testing Framework
─────────────────────────────────────────────────────────────────
Built on top of native Test blocks. Adds:
  - Test suites (groups of related tests)
  - Setup and teardown (run code before/after each test)
  - Mocking (replace a real function with a fake one)
  - Property-based testing (check invariants with random inputs)
  - Benchmarking (measure how fast code is)
  - Code coverage (which lines were executed)

─────────────────────────────────────────────────────────────────
Aura.Crypto
─────────────────────────────────────────────────────────────────
SHA-256, SHA-512, HMAC, AES encryption/decryption, RSA,
cryptographically secure random numbers.

Security policy: the module loudly refuses insecure algorithms.
Using MD5 or SHA-1 produces a compiler warning that these are
broken for security purposes.

─────────────────────────────────────────────────────────────────
Aura.Parse
─────────────────────────────────────────────────────────────────
Regular expressions, CSV parsing, simple tokenizers.

  // Regular expressions with readable syntax
  Set pattern to a pattern matching one or more digits.
  Set pattern to a pattern matching "[0-9]+".
  If text matches pattern:
  Set matches to all matches of pattern in text.
  Set result to text with pattern replaced by "***".

─────────────────────────────────────────────────────────────────
Aura.Compress
─────────────────────────────────────────────────────────────────
ZIP, GZIP, LZ4 compression and decompression.

  Set compressed to compress data using GZIP.
  Set original to decompress compressed as GZIP.
  Create zip archive "output.zip" containing ["a.txt", "b.txt"].
  Extract zip archive "output.zip" to "output/".

### 18.3 v1.0 — Complete Standard Library

─────────────────────────────────────────────────────────────────
Aura.DB — Database Access
─────────────────────────────────────────────────────────────────
Common interface for SQLite (built-in), PostgreSQL, MySQL.
Parameterized queries only — string concatenation SQL is a
compile error (prevents SQL injection by design):

  Use a SQLite database at "mydb.db" as db:
      Set users to db.query(
          "SELECT * FROM users WHERE age > ?",
          with parameters [18]).
      Repeat for each user in users:
          Say "[user["name"]] is [user["age"]] years old."

  // String concatenation SQL — compile error
  db.query("SELECT * FROM users WHERE name = '" + name + "'")
  // ERROR: String concatenation in SQL queries is forbidden.
  // Use parameterized queries to prevent SQL injection.
  // Example: db.query("... WHERE name = ?", with parameters [name])

─────────────────────────────────────────────────────────────────
Aura.Web — Minimal Web Framework
─────────────────────────────────────────────────────────────────
Define routes, handle requests, send responses. Flask-equivalent.

  Import Aura.Web.

  Aura.Web.on get "/hello" respond with:
      Return Aura.Web.text response("Hello, World!").

  Aura.Web.on get "/user/[id]" respond with:
      Set user to find user by id(id).
      Return Aura.Web.json response(user).

  Begin:
      Aura.Web.start on port 8080.
  End.

─────────────────────────────────────────────────────────────────
Aura.System — OS Interop
─────────────────────────────────────────────────────────────────
  Set value to Aura.System.environment variable "HOME".
  Set pid to Aura.System.current process id.
  Set os to Aura.System.operating system name.
  Set time to Aura.System.current time.
  Set date to Aura.System.current date.
  Aura.System.sleep for 500 milliseconds.
  Aura.System.exit with code 0.
  Set result to Aura.System.run command "ls -la".

─────────────────────────────────────────────────────────────────
Aura.Memory — Low-Level Memory Tools
─────────────────────────────────────────────────────────────────
Full allocator system (see Section 8.5).
Raw allocation primitives for manual mode.

─────────────────────────────────────────────────────────────────
Aura.UI — Simple UI Toolkit
─────────────────────────────────────────────────────────────────
Minimal native UI. Wrapper around Raylib or similar.
Enough for educational tools, quick utilities, and games.
Not a full application framework.

─────────────────────────────────────────────────────────────────
Aura.Image — Image Operations
─────────────────────────────────────────────────────────────────
Load PNG/JPEG/BMP, resize, crop, convert to grayscale, save.
Not a full image processing library — just the essentials.

─────────────────────────────────────────────────────────────────
Aura.Audio — Audio Operations
─────────────────────────────────────────────────────────────────
Play audio files, basic recording. Needed for games and
educational tools.

─────────────────────────────────────────────────────────────────
Aura.Serial — Serial Port Communication
─────────────────────────────────────────────────────────────────
Communicate with Arduino, ESP32, and other embedded hardware.
Given Aura's embedded ambitions, serial port support belongs
in v1.0.

### 18.4 v2.0 and Community

  Aura.Math.ML     — basic ML primitives (linear regression,
                     logistic regression, k-means, simple neural
                     network layers). Not a PyTorch replacement —
                     enough to learn the concepts in Aura.

  Aura.Math.Symbolic — symbolic algebra (simplify, solve, derive,
                     integrate). The SymPy equivalent. May be
                     developed as a community project first and
                     officially adopted at v2.0.

  Aura.GPU         — GPU compute via Vulkan or CUDA.
                     Aura.Math.Tensor lays the groundwork.

  Aura.Lang        — tools for building languages in Aura.
                     Lexer generator, parser generator, AST library.
                     Meta: Aura's own compiler could eventually be
                     built using these tools.

---

## 19. Error Messages and Compiler Voices

### 19.1 Design Philosophy

Aura error messages are first-class features, not afterthoughts.
The compiler produces a Semantic Traceback — similar to Python's
tracebacks but richer, because the compiler has full compile-time
type information available.

A Semantic Traceback traces the entire logic chain back to the
root cause, not just the line that failed. Like Python, it shows
the most recent call last, so the actual error is always at the
bottom and the origin of the problem can be traced upward.

### 19.2 Semantic Traceback Example

  Aura Traceback (most recent call last):

    main.aura, line 5, in Begin:
      Set results to scrape and analyze("https://bad-url.com").

    web_tools.aura, line 12, in scrape and analyze:
      Set response to await networking.get(url).

    Aura.Net, in get(url):
      DNS resolution failed for host "bad-url.com".

  NetworkError: The server "bad-url.com" could not be reached.
    > Check that the URL is correct and the server is running.
    > If you expect failures, wrap this in a Try / In case block.
    > See: aura-lang.org/docs/errors/NetworkError

Every error includes:
- The full call chain from Begin to the failure point
- The exact line and the expression that failed
- A plain-language explanation of what went wrong
- One or two concrete suggestions for fixing it
- A documentation link (in release builds)

### 19.3 Voice Modes

Configured via the --voice flag. Default is teacher.

  --voice teacher (default)
  ─────────────────────────
  Tone: supportive, educational, never condescending.
  Suggests fixes. Explains why the error happened.

  Example — calling a method on a Structure:
    "I ran into a little trouble on line 12. You're calling
    .Greet() on Header, but Header is a Structure — Structures
    hold data and don't have actions. If you want Header to be
    able to Greet, change it to a Concept on line 4. Here's
    what that would look like: ..."

  --voice formal
  ──────────────
  Tone: strict, concise, professional. No suggestions unless
  requested. Minimal output.

  Example:
    "TypeError at line 12: method call on Structure type
    'Header'. Structures do not support actions. Declare
    as Concept."

  --voice sarcastic
  ─────────────────
  Tone: witty, dry. Still helpful but does not hide its
  opinions about your choices.

  Example:
    "Oh great, a method on a Structure. Header holds data,
    not feelings. Use a Concept."

  --voice ramsay
  ──────────────
  Tone: Gordon Ramsay. Aggressive. Chef-themed. Still
  technically correct.

  Example:
    "WHAT IS THIS?! You're calling Greet() on a STRUCTURE?!
    A Structure holds DATA! It doesn't have FEELINGS!
    Get out of my kitchen and use a Concept!"

### 19.4 Seasonal and Community Modes

Released as optional official modes to keep the community engaged.
These are purely cosmetic — the underlying error information is
identical.

  --voice halloween
    Keywords renamed: Define → Summon, Delete → Exorcise,
    Run → Haunt, Error → Curse, Warning → Omen.
    Errors are delivered as "curses."
    "Your code has been cursed on line 12..."

  --voice christmas
    Keywords renamed: Import → Unwrap, Error → Coal,
    Return → Deliver, Package → Gift.
    Errors delivered by "the Compiler Elf."
    "Ho ho ho! The Compiler Elf found some coal on line 12..."

  --voice capy
    All teacher-mode messages spoken in Capy's voice.
    Extra encouraging. Includes capybara puns.
    "Capy believes in you! Just a small snag on line 12..."

### 19.5 Verbosity Flags

  --verbose          Full traceback, all context (default in debug)
  --quiet            Show only the final error, no traceback
  --silent           No output — exit code only (for CI pipelines)
  --explain          After an error, show a detailed tutorial
                     about the error type and how to fix it

### 19.6 Common Error Messages — Teacher Voice Examples

Type mismatch:
  "On line 8, 'x' is an integer (you set it to 10 on line 3),
  but you're trying to give it the string "hello". In Aura,
  once a variable has a type it keeps that type. If you need
  a variable that can hold different types, declare it as:
  Set x to a Thing."

Missing null check:
  "On line 15, 'nickname' is an optional string, which means
  it might not have a value. Before using it, check whether
  it exists:
    If nickname exists:
        Say nickname.
  The compiler can't guarantee it's safe to use otherwise."

Structure method call:
  "On line 12, you're calling .Greet() on 'Header', but Header
  is a Structure (defined on line 2). Structures hold data —
  they don't have actions. If you want Header to have a Greet
  action, change 'structure' to 'concept' in its definition."

Exhaustiveness warning (enum not fully covered):
  "Your Match block on line 20 handles North, South, and East,
  but Direction also has West. Either add a case for West or
  add an Otherwise clause to handle any remaining cases."

Circular import:
  "Circular import detected:
     main.aura → networking.aura → utils.aura → main.aura
  Package A cannot import a package that imports Package A.
  Consider moving the shared code into a third package that
  both can import."

SQL injection prevention:
  "String concatenation in SQL queries is forbidden in Aura.
  This prevents SQL injection attacks by design.
  Instead of: db.query("SELECT * FROM users WHERE name = '"
              + name + "'")
  Use: db.query("SELECT * FROM users WHERE name = ?",
                with parameters [name])"

---

## 20. Pointer Syntax

### 20.1 Overview

Pointers are available in Aura for systems programming and manual
memory management. They are not needed for everyday Aura code —
ARC handles memory automatically by default. Pointers become
relevant when using --mem manual, writing embedded code, or
interfacing directly with C libraries.

### 20.2 Complete Pointer Syntax Table

  Concept              Pseudocode                      Terse
  ──────────────────────────────────────────────────────────────────
  Pointer type         a pointer to a Node             *Node
  Address of x         the address of x                &x
  Read through p       the value at p                  *p
                       what p points to                *p
  Write through p      Set the value at p to 42.       *p = 42
  Member via pointer   p's name  (auto-deref)          p.name
  Null pointer         null                            null
  Null check           If p exists:                    if p:
  Pointer arithmetic   Increase p by 4.                p += 4

### 20.3 Auto-Dereferencing for Member Access

In Aura terse syntax, the dot operator (.) always works for
member access regardless of whether the variable is a direct
struct or a pointer to a struct. The compiler checks the type
and automatically dereferences pointers.

This eliminates the reason C needed the -> operator:

  // C — you must remember when to use . and when to use ->
  struct Node  node;   node.value = 10;    // direct struct
  struct Node *ptr;    ptr->value = 10;    // pointer to struct

  // Aura — always use . (or 's), compiler handles the rest
  Set node to a new Node.
  node's Value is 10.     // direct — compiler uses .
  Set ptr to a pointer to node.
  ptr's Value is 20.      // pointer — compiler auto-dereferences

The -> operator is accepted as a synonym in terse mode for
developers coming from C, but . is the canonical form and what
aura fmt will normalize to.

### 20.4 How the Compiler Disambiguates *

The * character has three distinct meanings in Aura terse syntax.
The compiler resolves them by context — specifically, by which
grammatical position * appears in:

  p: *Node        // type position → pointer type declaration
  *p = 5          // start of statement or after = → dereference
  x = *p          // after = before single identifier → dereference
  a * b           // between two expressions → multiplication

This is identical to how C resolves the ambiguity. It works
without ambiguity because the parser knows whether it is in
a type declaration context, a statement context, or an
expression context.

### 20.5 Pointer Examples

  // Creating a pointer
  Set node to a new manual Node.
  node's Value is 42.
  Set ptr to the address of node.

  // Reading through a pointer
  Set val to the value at ptr.          // pseudocode
  let val = *ptr                        // terse

  // Writing through a pointer
  Set the value at ptr to 100.          // pseudocode
  *ptr = 100                            // terse

  // Pointer arithmetic (manual mode)
  Set p to the address of buffer.
  Increase p by 4.                      // advance 4 bytes
  Set val to the value at p.

  // Null pointer
  Set p to a pointer to a Node.
  p is null.

  If p exists:
      Say p's Value.
  Else:
      Say "Pointer is null."

### 20.6 Inspect Shows Binary in Debug Mode

When Inspect is used on an integer in a debug build, it shows
the binary representation alongside the decimal. This makes
pointer arithmetic and bit operations less mysterious:

  Inspect 42.
  // Output: 42 (integer, 64-bit)
  //   decimal: 42
  //   hex:     0x2A
  //   binary:  0b00101010
  //   address: 0x7ffd5c3a8b40  (in debug mode)

---

## 21. Bitwise Operations

Bitwise operations manipulate the individual bits of an integer.
They are essential for systems programming, embedded work,
cryptography, and performance-critical code, but are not needed
for everyday programming.

### 21.1 Operations

  Concept          Pseudocode                         Terse
  ──────────────────────────────────────────────────────────────────
  Shift left       x with bits shifted left by 2      x << 2
  Shift right      x with bits shifted right by 3     x >> 3
  Bitwise and      x combined with y using bitwise and  x & y
  Bitwise or       x combined with y using bitwise or   x | y
  Bitwise xor      x combined with y using xor          x xor y
  Bitwise not      x with bits flipped                  ~x

### 21.2 The ^ Disambiguation

  ^   means exponentiation:   2 ^ 8  =  256
  xor means bitwise XOR:      x xor y

This resolves the ambiguity present in many languages where ^
is used for both. A non-programmer seeing 2 ^ 8 correctly
guesses "two to the power of eight." This passes the Guess Test.

The xor keyword is only reserved in terse-style bitwise context.
It can be used as a variable name in pseudocode.

### 21.3 Examples

  // Setting a specific bit (bit 3)
  Set x with bit 3 set to x combined with
      1 with bits shifted left by 3 using bitwise or.

  // Terse
  x = x | (1 << 3)

  // Checking if a bit is set
  If x combined with 1 with bits shifted left by 3
     using bitwise and is not 0:
      Say "Bit 3 is set."

  // Terse
  if (x & (1 << 3)) != 0: say "Bit 3 is set."

  // Fast multiply by 4 (shift left by 2)
  Set result to x with bits shifted left by 2.

  // Terse
  let result = x << 2

### 21.4 Inspect for Bit Operations

When learning bitwise operations, Inspect in debug mode shows
the full binary representation, making the operations visible:

  Set x to 42.
  Inspect x.       // binary: 0b00101010

  Set y to x with bits shifted left by 1.
  Inspect y.       // binary: 0b01010100  (= 84)

---

## 22. Formatting and Tooling

### 22.1 aura fmt — The Official Formatter

aura fmt ships with the Alpha compiler. Every Aura codebase
looks the same. Reading someone else's Aura code is as easy as
reading your own. This directly serves Axiom 0.

Rules enforced by aura fmt:

  Indentation:        4 spaces (never tabs)
  Between definitions: 1 blank line between functions, structures,
                       concepts, and enumerations at the top level
  Inside functions:   No mandatory blank lines, but one blank line
                      between logical sections is encouraged
  Line length:        No hard limit enforced, but aura fmt will
                      warn at 100 characters
  Trailing whitespace: Always removed
  String quotes:       Always double quotes
  Semicolons:         Removed by aura fmt in pseudocode style,
                      preserved in terse style

aura fmt never changes the meaning of code — only whitespace
and formatting. It is safe to run on any file at any time.

### 22.2 Mixed Style Formatting

When a file contains both pseudocode and terse constructs,
aura fmt formats each section according to its own style
conventions. It does not force the entire file to one style.

  aura fmt my_file.aura               // format in place
  aura fmt --check my_file.aura       // check only, no changes
                                      // exit code 1 if changes needed
  aura fmt --diff my_file.aura        // show what would change

### 22.3 aura check — Fast Type Check

Runs the full preprocessing pipeline and semantic analysis
without producing a binary. Fast — intended for editor integration
and CI pre-commit hooks:

  aura check my_file.aura             // check one file
  aura check .                        // check entire project

### 22.4 aura lsp — Language Server (Beta)

Implements the Language Server Protocol (LSP). One language server,
every compatible editor for free: VS Code, Neovim, Helix, Emacs,
Sublime Text, and others.

  aura lsp                            // start LSP server over stdio

Features provided to editors:
  - Autocomplete (function names, field names, keywords)
  - Go to definition
  - Find all references
  - Inline type information on hover
  - Real-time error highlighting (calls aura check continuously)
  - Rename symbol
  - Format on save (calls aura fmt)
  - Inline style translation on hover — shows the terse version
    of any pseudocode line, or vice versa, without rewriting
    the file. This is the killer learning feature.

The language server is Beta work. It requires a stable parser
and type checker. It is planned from day one because LSP support
influences internal compiler architecture — the compiler must
be structured as composable stages (which the two-stage pipeline
already provides) rather than a monolithic pipeline.

### 22.5 aura translate — Style Conversion

Rewrites source files between pseudocode and terse styles.
Mixed-style files are handled gracefully:

  aura translate --style terse my_file.aura
  aura translate --style pseudocode my_file.aura
  aura translate --style terse --lang pt_BR my_file.aura

This is also the mechanism for the bi-directional learning tool:
write in pseudocode, run translate to see the professional terse
equivalent. Write in terse, translate to pseudocode to generate
self-documenting code.

---

## 23. Complete Code Examples

### 23.1 Binary Search Tree — Pseudocode Style

A fully manual, pointer-based BST with no standard library.
This is the canonical target program that Alpha must compile
successfully. It exercises: structures, pointers, manual memory,
recursion, null checks, and multi-word function names.

  Package main.

  Define Node as a structure:
      Value is an integer.
      Left  is a pointer to a Node.
      Right is a pointer to a Node.

  Define create node(val which is an integer) as:
      Set n to a new manual Node.
      n's Value is val.
      n's Left  is null.
      n's Right is null.
      Return n.

  Define insert(root which is a pointer to a Node,
                val which is an integer) as:
      If root does not exist:
          Return create node(val).
      If val is less than root's Value:
          Set root's Left to insert(root's Left, val).
      Else if val is greater than root's Value:
          Set root's Right to insert(root's Right, val).
      Return root.

  Define exists in tree(root which is a pointer to a Node,
                        target which is an integer) as:
      If root does not exist: Return false.
      If root's Value is target: Return true.
      If target is less than root's Value:
          Return exists in tree(root's Left, target).
      Else:
          Return exists in tree(root's Right, target).

  Define free tree(root which is a pointer to a Node) as:
      If root does not exist: Return.
      free tree(root's Left).
      free tree(root's Right).
      Delete root.

  Begin:
      Set my_tree to a pointer to a Node.
      my_tree is null.

      Set my_tree to insert(my_tree, 50).
      insert(my_tree, 30).
      insert(my_tree, 70).
      insert(my_tree, 20).
      insert(my_tree, 40).

      If exists in tree(my_tree, 20):
          Aura.IO.write("Found 20 in the tree!").
      Else:
          Aura.IO.write("20 is not in the tree.").

      If exists in tree(my_tree, 99):
          Aura.IO.write("Found 99.").
      Else:
          Aura.IO.write("99 is not in the tree.").

      free tree(my_tree).
  End.

  Test "insert and search":
      Set t to a pointer to a Node.
      t is null.
      Set t to insert(t, 50).
      insert(t, 30).
      insert(t, 70).
      Assert exists in tree(t, 50) equals true.
      Assert exists in tree(t, 30) equals true.
      Assert exists in tree(t, 99) equals false.
      free tree(t).

### 23.2 Binary Search Tree — Terse Style

Identical logic. Identical AST. Professional shorthand style.

  pkg main

  type Node { val: int, left: *Node, right: *Node }

  fn create_node(v: int) -> *Node {
      let n = new manual Node
      n.val = v; n.left = null; n.right = null
      return n
  }

  fn insert(root: *Node, v: int) -> *Node {
      if !root: return create_node(v)
      if v < root.val      { root.left  = insert(root.left,  v) }
      else if v > root.val { root.right = insert(root.right, v) }
      return root
  }

  fn exists(root: *Node, t: int) -> bool {
      if !root:         return false
      if root.val == t: return true
      return t < root.val
          ? exists(root.left,  t)
          : exists(root.right, t)
  }

  fn free_tree(root: *Node) {
      if !root: return
      free_tree(root.left)
      free_tree(root.right)
      delete root
  }

  Begin:
      let tree: *Node = null
      tree = insert(tree, 50)
      insert(tree, 30); insert(tree, 70)
      insert(tree, 20); insert(tree, 40)

      if exists(tree, 20) { Aura.IO.write("Found 20") }
      else                { Aura.IO.write("20 not found") }
      free_tree(tree)

  Test "insert and search" {
      let t: *Node = null
      t = insert(t, 50); insert(t, 30); insert(t, 70)
      assert exists(t, 50) == true
      assert exists(t, 30) == true
      assert exists(t, 99) == false
      free_tree(t)
  }

### 23.3 Mixed Style — Point Distance (Valid Aura)

  Package geometry.

  // Terse structure
  type Point { x: dec, y: dec }

  // Pseudocode function
  Shared Define distance between(a which is a Point,
                                 b which is a Point) as:
      Set dx to a.x - b.x.
      Set dy to a.y - b.y.
      Return the square root of (dx ^ 2 + dy ^ 2).

  // Terse function in the same file
  pub fn midpoint(a: Point, b: Point) -> Point {
      return Point((a.x + b.x) / 2, (a.y + b.y) / 2)
  }

  Begin:
      let origin = Point(0.0, 0.0)
      let p      = Point(3.0, 4.0)

      Say "Distance: [distance between(origin, p)]."   // 5.0
      Say "Midpoint: [midpoint(origin, p)]."           // Point(1.5, 2.0)
  End.

### 23.4 Async Web Scraper — Hybrid Style

  Package web_tools.

  Import networking.
  Import scraping_utils as scraper.

  Shared Define scrape and analyze(url which is a string) as:
      Try to wait for networking.get(url) and set response to it:
          If response's status is 200:
              Set html to response's body.
              Set links to await scraper.find all links(html).
              Set results to a new Map of String to String.

              Repeat for each link in links:
                  Set results[link] to "Pending".

              Return results.
          Else:
              Say "Error: server returned [response's status]."
              Return null.
      In case of error:
          For networking.TimeoutError do:
              Say "Server timed out."
              Return null.
          Otherwise:
              Say "Unknown networking error."
              Return null.

  Begin:
      Set site_map to await scrape and analyze(
          "https://aura-lang.org").

      If site_map exists:
          Say "Found [the size of site_map] links."
      Else:
          Say "Scrape failed."
  End.

### 23.5 Functional Pipeline — Real Example

  Package analytics.

  Import Aura.IO.
  Import Aura.Text.

  Define is valid price(price which is a decimal) as:
      Return price is greater than 0 and price is less than 10000.

  Pure Define apply discount(price which is a decimal) as:
      Return price * 0.9.

  Pure Define format as currency(price which is a decimal) as:
      Return "$[price rounded to 2 decimal places]".

  Begin:
      Set raw prices to [10.00, -5.00, 25.50, 8.75,
                         42.00, 99999.00, 3.99].

      Set checkout total to raw prices
          then keep only the ones where is valid price applies
          then take 10 percent off each one using apply discount
          then add them all together.

      Set formatted to raw prices
          then keep only the ones where is valid price applies
          then transform each one using format as currency
          then join with ", ".

      Say "Valid prices: [formatted]."
      Say "Checkout total: [format as currency(checkout total)]."
  End.

### 23.6 Generic Stack with Interface

  Package collections.

  Define something that can be displayed as:
      It must have an Action called Display
          that returns a string.

  Define Stack of Thing as a concept:
      Items is a list of Thing.

      Action Push(item which is a Thing):
          Add item to Items.

      Action Pop:
          If Items is empty: Return null.
          Set top to the last item of Items.
          Remove the last item from Items.
          Return top.

      Action Peek:
          If Items is empty: Return null.
          Return the last item of Items.

      Action Is empty:
          Return the size of Items is 0.

      Representation:
          Return "Stack([Items joined with ", "])".

  Shared Define print all(stack which is a Stack of something
                          that can be displayed) as:
      While stack is not empty:
          Say Pop from stack's Display().

  Begin:
      Set number stack to a new Stack of Integer.
      Push 10 to number stack.
      Push 20 to number stack.
      Push 30 to number stack.

      Say "Top: [Peek at number stack]."    // 30
      Say number stack.                     // Stack(10, 20, 30)

      While number stack is not empty:
          Say Pop from number stack.        // 30, 20, 10
  End.

### 23.7 Error Handling with Result Type

  Package files.

  Import Aura.IO.

  Define read config(path which is a string)
      returns Result of Map of String to String or Error as:
      If path does not exist as a file:
          Return a FileNotFoundError with message
              "Config file not found: [path]".

      Try to open path in read mode as f:
          Set content to f.read_all().
          Set config to parse config text(content).
          Return config.
      In case of error:
          For PermissionError do:
              Return a PermissionError with message
                  "Cannot read config: [path]".
          Otherwise:
              Return an Error with message
                  "Unexpected error reading [path]".

  Begin:
      Match read config("settings.cfg"):
          If it succeeded with config:
              Say "Loaded [the size of config] settings."
              Say "Host: [config["host"]]."
          If it failed with error:
              Say "Could not load config: [error's message]."
              Aura.System.exit with code 1.
  End.

### 23.8 C Library Interop — Raylib Window

  Import raylib.

  Begin:
      raylib.InitWindow(800, 450, "Aura + Raylib").

      While raylib.WindowShouldClose() is false:
          raylib.BeginDrawing().
          raylib.ClearBackground(raylib.RAYWHITE).
          raylib.DrawText("Aura is running!",
              190, 200, 20, raylib.DARKGRAY).
          raylib.EndDrawing().

      raylib.CloseWindow().
  End.

### 23.9 Portuguese Source — BST Insert (Locale Demo)

This code compiles identically to the English BST in 23.1.
After Stage 1 translation, it becomes English pseudocode.
After Stage 2 normalization, it becomes canonical English terse.
Variable names remain in Portuguese throughout.

  Pacote colecoes.

  Defina No como uma estrutura:
      Valor e um inteiro.
      Esquerda e um ponteiro para um No.
      Direita e um ponteiro para um No.

  Defina criar no(val que e um inteiro) como:
      Atribua n a um novo No manual.
      O valor do n e val.
      A esquerda do n e nulo.
      A direita do n e nulo.
      Retorne n.

  Defina inserir(raiz que e um ponteiro para um No,
                 val que e um inteiro) como:
      Se raiz nao existe:
          Retorne criar no(val).
      Se val e menor que o valor do raiz:
          A esquerda do raiz e inserir(a esquerda do raiz, val).
      Senao se val e maior que o valor do raiz:
          A direita do raiz e inserir(a direita do raiz, val).
      Retorne raiz.

---

## 24. Brand Identity

### 24.1 Name — Aura

Short, globally pronounceable in virtually every language,
memorable, and available in the programming language namespace.
Evokes a surrounding field of energy and communication — fitting
for a language designed to bridge humans and machines across
cultures and spoken languages.

One syllable. No ambiguous characters. No cultural baggage.

### 24.2 Tagline

"The code should never get in the way of the story."

### 24.3 Logo

The Aura logo is a professional wordmark separate from the
mascot, following the Go language convention of keeping the
brand mark clean and the mascot approachable.

  Wordmark:     AURA in a strong modern sans-serif
                (Inter or Geist recommended)
  Gradient:     Indigo (#3B0F8C) → Violet (#6A2DBF)
                Left to right, depth and energy
  Icon:         Minimal spark or ripple motif above the wordmark
                In cyan (#0EA5E9) and magenta, suggesting emission
  Background:   Clean white for professional use
                Transparent for dark-mode contexts
  Usage rule:   The wordmark and mascot are never combined in
                official materials. The wordmark is for
                documentation, conference slides, and tooling.
                The mascot is for learning materials, error
                messages, and community contexts.

### 24.4 Mascot — Capy

Capy is a 2D flat-illustrated Brazilian capybara. She is the
face of the --voice teacher and --voice capy compiler modes,
the mascot of the official learning materials, and the icon
of the community Discord.

  Species:        Brazilian capybara
                  (Hydrochoerus hydrochaeris)
                  Brazil is the creator's home country.
                  Aura ships with English and Portuguese as
                  its two launch locale files — this is
                  reflected in the mascot's origin.

  Art style:      2D flat illustration — simple, approachable,
                  not complex 3D. Cute but not infantile.
                  Similar register to Go's Gopher or Rust's
                  Ferris the crab.

  Colors:         Warm brown body with soft golden highlights.
                  Indigo-to-violet glow aura around the body
                  (matching brand colors).

  Accessories:    Small round spectacles — the wise teacher.
                  Holds a glowing </> symbol — the programmer.

  Personality:    Calm, non-judgmental, endlessly patient.
                  Wise without being condescending. Never
                  sarcastic (that is a different --voice mode).
                  Genuinely happy when your code compiles.

  Capy's motto:   "Capy believes in you."

### 24.5 Color Palette

  Name              Hex         Usage
  ───────────────────────────────────────────────────────────────
  Indigo            #3B0F8C     Primary brand — logo, H1, borders
  Violet            #6A2DBF     Secondary — H2, accents, callouts
  Purple            #8B47C9     Tertiary — H3, subtle highlights
  Cyan              #0EA5E9     Logo icon accent, hyperlinks
  Magenta           #D946EF     Logo icon accent (paired with cyan)
  Dark              #1A1A2E     Body text
  Gray              #555577     Captions, secondary text
  Light Background  #F0ECF8     Callout backgrounds, table alt rows
  Code Background   #1E1E2E     All code blocks (Catppuccin Mocha)
  Code Foreground   #CDD6F4     Code text (Catppuccin Mocha)
  Border            #C4B0E8     Table borders, dividers

### 24.6 Origin

Aura was designed by a Brazilian developer who is the sole
developer at the companies they work for — bringing the
experience of a self-taught independent developer with the
scope of someone who has built production systems alone.

The language is designed for the developer who was never in a
room full of senior engineers to learn from, the student who
cannot afford to also learn English before learning to code,
and the systems programmer who wants C-level performance
without C-level ceremony.

Both of Aura's launch locale files — English and Portuguese —
are first-class and maintained by the core team, not the
community. Portuguese is not an afterthought.

---

## 25. Open Questions and Future Work

These decisions are intentionally deferred to avoid
over-engineering before real implementation feedback. They are
not forgotten — they are waiting for the right moment.

### 25.1 Deferred to Beta

  Interfaces / Protocols
  ──────────────────────
  Current thinking: structural typing (like Go) fits Aura's
  philosophy best. A Concept automatically satisfies an interface
  if it has the required Actions, with no explicit "implements"
  declaration needed.

  The pseudocode syntax ("Define something that can be printed
  as:") will be finalized once real usage patterns emerge in
  Alpha. The mechanism is decided; only the exact surface syntax
  is deferred.

  Coroutines vs. Green Threads
  ────────────────────────────
  "Run in background" needs an implementation decision:
  OS threads, green threads (Go-style goroutines), or LLVM
  coroutines. This choice affects the entire async runtime.
  Defer to Beta when LLVM is available and benchmarks can
  inform the decision.

  Full Allocator System
  ─────────────────────
  The architecture is decided (see Section 8.5). The exact
  API surface of each allocator type needs real usage feedback
  from Alpha before being locked.

  Closures and Captures
  ─────────────────────
  The syntax is decided (Section 10.9). The implementation
  detail of how captured variables are stored (stack frame
  pointer, heap allocation, copy) depends on the ARC runtime
  design finalized in Beta.

### 25.2 Deferred to v1.0

  Compile-Time Code Execution
  ───────────────────────────
  Running arbitrary Aura code during compilation to generate
  tables, types, or other code at zero runtime cost. Like Zig's
  comptime or Jai's #run.

  Powerful but hard to implement correctly without a fully
  stable language. Scheduled for v1.0 when the language
  semantics are proven and the self-hosted compiler is stable.

  Compile-Time Reflection
  ───────────────────────
  Read-only: letting the compiler answer questions like "what
  fields does this structure have?" at compile time. Used for
  auto-generating locale maps, serialization, and debug output.

  Full runtime reflection (asking type questions at runtime)
  is expensive and rarely needed. Compile-time reflection only.

  Right-to-Left Language Support
  ──────────────────────────────
  The locale JSON architecture supports right-to-left languages
  structurally. Editor tooling — plugins, syntax highlighting,
  and the LSP — needs additional work to handle bidirectional
  text correctly. Planned for v1.0 with community partnership.

  Operator Overloading
  ────────────────────
  Likely supported for Concepts via named Actions:
    Action Plus(other) — defines behavior of +
    Action Multiply(other) — defines behavior of *
  This keeps the natural-language feel while enabling
  mathematical types (vectors, matrices, complex numbers).
  Exact syntax deferred until compile-time semantics are clear.

### 25.3 Deferred to v2.0

  Gradual Typing
  ──────────────
  The ability to declare a variable as "Thing" and have the
  compiler add runtime type checks, enabling optional dynamic
  typing in a static language. Useful for rapid prototyping.
  Adds runtime overhead — must be explicit opt-in.

  Full Metaprogramming
  ────────────────────
  A macro system or compile-time AST manipulation beyond
  comptime. Deferred until the language's normal surface syntax
  is stable enough that metaprogramming cannot accidentally
  undermine it.

  GPU Compute
  ───────────
  Write Aura code that runs on the graphics card. Likely
  based on Vulkan compute or CUDA. Aura.Math.Tensor lays the
  conceptual groundwork. Major undertaking — v2.0 at earliest.

### 25.4 Permanently Open (Community Decides)

  Partial Initialization
  ──────────────────────
  The "where X is 3.0 and Y is 4.0" instantiation syntax needs
  a formal spec for required vs. optional fields and default
  values. This is straightforward but the exact rules should
  emerge from real usage rather than be over-specified now.

  Error as Values vs. Exceptions
  ───────────────────────────────
  Aura supports both patterns: Try/In case (exception-style)
  and Result types (value-style). This coexistence is
  intentional. Whether one pattern should be preferred in
  the official style guide is a community decision once
  both patterns have seen real usage.

  Standard Library Expansion
  ──────────────────────────
  Everything beyond v1.0 (Aura.Math.ML, Aura.GPU, Aura.Lang)
  is better developed as community projects first and officially
  adopted once stable, rather than designed top-down by the
  core team.

---

## 26. Contributing Guidelines

### 26.1 The Spec is the Source of Truth

Before writing any code, read the spec. The spec is the single
authoritative reference for every language design decision.
If code contradicts the spec, the code is wrong.

If the spec is wrong or incomplete, open an Issue to discuss
changing the spec before writing any code to implement the
change. The spec is amended by consensus, not by pull request.

### 26.2 The Workflow

  1. Open an Issue describing what you want to build or fix.
  2. Reference the relevant section of the spec in your Issue.
  3. Discuss in the Issue until there is consensus.
  4. Create a branch from main:
       git checkout -b feature/your-feature-name
  5. Write code on your branch.
  6. Open a Pull Request referencing the Issue.
  7. Request review from at least one core team member.
  8. Address review comments.
  9. A core team member merges when approved.

Nobody pushes directly to main — including the project creator.
Everything goes through Pull Requests. This is enforced by
GitHub branch protection rules.

### 26.3 The Guess Test is Required for All Syntax PRs

Any Pull Request that adds or changes syntax must include a
Guess Test section in the PR description:

  Guess Test:
  - A competent English speaker who has read the axioms but
    never seen this feature would write: ...
  - Their guess compiles and works correctly: yes / partially
  - If partially: the failure mode is safe (compile error,
    not silent wrong behavior): yes / no

PRs that add syntax without a passing Guess Test will not
be merged.

### 26.4 The Structured Natural Language Constraint

Any PR that adds a new pseudocode construct must document
the template structure:

  Surface form:    "prices keeping only the ones where [cond]"
  Hidden template: [collection] keeping only the ones where
                   [condition]
  Translator slots: keyword = "keeping only the ones where"
                    slot 1  = [collection] (before keyword)
                    slot 2  = [condition]  (after keyword)
  Locale impact:   Translators must map this phrase and
                   word order for each target language.

PRs that add pseudocode constructs without documenting the
template structure will not be merged.

### 26.5 Core Team

The core team is a small group (2-3 people initially) whose
judgment is trusted on architectural decisions. Core team
members are not necessarily the most prolific contributors —
they are the people who ask the most thoughtful questions.

How to become a core team member: contribute consistently,
demonstrate that you have read and understood the spec, and
show that your instincts about the language align with the
axioms. Core team membership is invited, not applied for.

Find the initial core team by posting the spec to communities
where language design is discussed seriously:
  - r/ProgrammingLanguages
  - The LLVM Discord
  - Lobste.rs
  - Hacker News (Show HN)

Look for people who ask "why did you make this decision?"
rather than "I want to implement this feature." The former
are potential core team members. The latter are potential
contributors.

### 26.6 What Belongs in Issues vs. Pull Requests

  Issues are for:
  - Reporting bugs in the compiler
  - Proposing new language features (discuss before coding)
  - Proposing standard library additions
  - Asking "why does this work this way?"
  - Reporting that the spec is unclear or contradictory

  Pull Requests are for:
  - Implementing something that was agreed in an Issue
  - Fixing a bug that was reported in an Issue
  - Updating the spec to reflect a decision made in an Issue
  - Improving documentation or examples

  Pull Requests should never be the first mention of a new
  idea. Ideas belong in Issues first.

### 26.7 Versioning

  v0.x.x    Pre-alpha. Spec is being written. No compiler yet.
  v0.1.x    Alpha. C-based compiler. Core language features.
            Not recommended for production use.
  v0.2.x    Alpha continued. Standard library Alpha modules.
  v1.0.0-beta  Self-hosted compiler. LLVM backend. Beta stdlib.
  v1.0.0    Stable release. Full stdlib. LSP. Both locales.
  v1.x.x    Patch and minor releases. No breaking changes.
  v2.0.0    First major revision. Community-decided features.

### 26.8 The Language Is Not a Democracy

Final decisions on language design rest with the project
creator and the core team. This is not because contributions
are not valued — they are essential. It is because language
design by committee produces languages that try to please
everyone and end up with every feature anyone ever asked for,
regardless of whether those features serve the axioms.

When a proposal is declined, the reason will always reference
a specific axiom or constraint. "This conflicts with Axiom 0"
is a complete reason. "We just don't like it" is not.

---

## Appendix A — Decision Log

This appendix records every significant design decision made
during the v0.0.1 specification process, including the
reasoning. Future contributors should read this before
proposing changes to any of these decisions.

  String interpolation syntax: square brackets [ ]
  ─────────────────────────────────────────────────
  Rejected: $ prefix (restricts to variables unless $expr$,
            carries shell scripting and PHP baggage)
  Rejected: {} braces (used for terse-style blocks)
  Chosen:   [ ] square brackets — clean, unused in pseudocode,
            visually distinct, no cultural baggage.
  Full expressions allowed inside brackets, not just variable
  names. Literal brackets escaped with \[ and \].

  Variable declaration order: name first, then type
  ─────────────────────────────────────────────────
  Rejected: type name (C convention — "integer counter = 0")
  Chosen:   name: type (Kotlin/Swift/Rust/Go convention)
  Reason:   Humans care about what something is called before
            they care about what type it is. "counter, which
            is an integer" is natural English. "integer called
            counter" is how a compiler thinks, not a person.

  Constants: Constant keyword, not "always" modifier
  ─────────────────────────────────────────────────
  Rejected: "Set PI to 3.14159 always." — awkward, "always"
            does not read naturally
  Chosen:   "Constant PI is 3.14159." — declarative, matches
            how non-programmers describe unchanging values,
            "Set" implies mutation so a different keyword
            is appropriate for immutable values.

  Wildcard imports: Adopt keyword, not "import *"
  ─────────────────────────────────────────────────
  Reason:   Making the choice to import everything visible and
            intentional prevents accidental namespace pollution.
            Adopt is a single word that means exactly what it
            does: take all of this package's symbols as your own.

  Exponentiation vs XOR: ^ for power, xor for XOR
  ─────────────────────────────────────────────────
  Reason:   A non-programmer seeing 2 ^ 8 guesses "two to the
            power of eight" — which is correct in Aura. The
            Guess Test passes for exponentiation and fails for
            XOR. Therefore ^ is exponentiation and bitwise XOR
            uses the word xor.

  Member access through pointers: . not ->
  ─────────────────────────────────────────────────
  Reason:   The distinction between . and -> in C requires the
            programmer to track whether a variable is a direct
            struct or a pointer to a struct. This is mechanical
            work the compiler can do. In Aura, . always works —
            the compiler auto-dereferences when needed.
            -> is accepted as a synonym but . is canonical.

  Deep copy semantics for * on collections
  ─────────────────────────────────────────────────
  Reason:   Python's [[0]*3]*3 trap (shared references) violates
            the Guess Test — almost nobody guesses the behavior
            correctly. Aura's * on collections always performs
            a deep copy. This is a documented, intentional
            divergence from Python and C behavior.

  Keyword case insensitivity
  ─────────────────────────────────────────────────
  Reason:   Removes the arbitrary capitalization question from
            locale file authors. If keywords were case-sensitive,
            the Portuguese locale would have to decide "is it
            Defina or defina?" and every Portuguese programmer
            would have to remember that arbitrary choice.
            Implementation cost: one lowercase() call in the
            lexer. Benefit: removes friction for all locales.

  4 spaces for indentation, 1 blank line between definitions
  ─────────────────────────────────────────────────
  Reason:   2 spaces (Nim's convention) are difficult to
            distinguish visually at a glance, particularly for
            beginners and in terminal output. 4 spaces is the
            Python convention and the most widely used standard
            in educational contexts. One blank line between
            definitions provides visual rhythm without the
            vertical waste of two blank lines.

  Alpha target: C transpilation, not LLVM IR directly
  ─────────────────────────────────────────────────
  Reason:   LLVM IR is verbose and debugging emitted IR while
            simultaneously debugging a new parser is two hard
            problems at once. C is the universal assembly
            language — every target that runs LLVM also has
            a C compiler. Emitting invalid C produces readable
            errors. GCC's 40+ years of optimization are free.
            Prior art: Nim, Haxe, early Kotlin.

  No -> operator in canonical terse (. auto-dereferences)
  ─────────────────────────────────────────────────
  Reason:   See "Member access through pointers" above.
            -> is accepted as input for C-trained developers
            but aura fmt normalizes it to . on format.

  Package identity from declaration, not folder name
  ─────────────────────────────────────────────────
  Reason:   Decouples logical organization from physical
            organization. Two files with "Package geometry"
            anywhere on disk are part of the same package.
            Folder names are an OS concern, not a language
            concern.

  APM auto-fetch disabled in --strict mode
  ─────────────────────────────────────────────────
  Reason:   Auto-fetch is a developer experience feature for
            rapid prototyping. Production and CI builds should
            never make surprise network calls. --strict mode
            (or any CI environment) requires all dependencies
            to be present in ./libs/ before compilation begins.

  Parameterized SQL queries only — string concatenation is
  a compile error
  ─────────────────────────────────────────────────
  Reason:   The Safety Default (Section 2.4) requires that the
            safe way is also the easy way. SQL injection via
            string concatenation is one of the most common and
            most damaging security vulnerabilities in web
            applications. Aura makes it impossible at compile
            time rather than relying on developers to remember
            to do the right thing.

---

## Appendix B — Feature Timeline Summary

  Alpha (C transpiler, day one)
  ──────────────────────────────
  Core language: all syntax, types, control flow, functions
  Pointers and manual memory
  ARC (basic, without full runtime)
  Structures and Concepts (without full vtable)
  Enumerations
  Pattern matching (basic)
  String interpolation
  Slicing
  Native Test blocks
  Assertions
  Pure function annotation
  Pipe operator
  Multiple return values
  Default and named parameters
  Variadic functions
  Closures (basic, no complex captures)
  Two-stage preprocessing pipeline
  Locale translation (English and Portuguese)
  aura fmt (4 spaces, 1 blank line between definitions)
  aura check (fast type-check)
  aura translate (style conversion)
  aura test (run Test blocks)
  Dead code elimination
  Standard library: Aura.IO, Aura.Text, Aura.Collections,
                    Aura.Math.Core, Aura.Error
  C interop (automatic type marshaling)
  APM (auto-fetch, version resolution, apm commands)
  All --voice modes (teacher, formal, sarcastic, ramsay)
  Semantic tracebacks

  Beta (self-hosted Aura compiler, LLVM backend)
  ───────────────────────────────────────────────
  Full ARC runtime with escape analysis
  Full allocator system (Arena, Pool, FixedBuffer, Tracking)
  Iterators and lazy evaluation (Yield)
  Full closures with proper capture semantics
  Interfaces (structural typing)
  Result type matching
  Compile-time reflection (read-only)
  SIMD-optimized Aura.Math.Tensor
  Full generic system with where constraints
  aura lsp (Language Server Protocol)
  Standard library: Aura.Net, Aura.JSON, Aura.Concurrency,
                    Aura.Math.Tensor, Aura.Test (full),
                    Aura.Crypto, Aura.Parse, Aura.Compress
  Polyglot APM (Rust, Zig wrappers)
  --freestanding enforcement

  v1.0-stable (assembly output, full independence)
  ─────────────────────────────────────────────────
  Compile-time code execution (comptime)
  Operator overloading via named Actions
  Right-to-left language support in tooling
  Standard library: Aura.DB, Aura.Web, Aura.System (full),
                    Aura.Memory (full), Aura.UI, Aura.Image,
                    Aura.Audio, Aura.Serial
  Community locale files beyond English and Portuguese
  Full bootstrapping (no C dependency anywhere)

  v2.0 and Community
  ───────────────────
  Gradual typing (optional dynamic types)
  Full metaprogramming / macro system
  Aura.Math.ML
  Aura.Math.Symbolic
  Aura.GPU
  Aura.Lang (tools for building languages in Aura)

---

*Aura v0.0.1 Design Specification — Locked.*
*"Aura is no longer a 'What If'. It is a 'When'."*
*Capy believes in you. 🐾*                                           