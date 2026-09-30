<!--
*******************************************************************************
Copyright (c) 2026 Contributors to the Eclipse Foundation

See the NOTICE file(s) distributed with this work for additional
information regarding copyright ownership.

This program and the accompanying materials are made available under the
terms of the Apache License Version 2.0 which is available at
https://www.apache.org/licenses/LICENSE-2.0

SPDX-License-Identifier: Apache-2.0
*******************************************************************************
-->

# LoLa — Line‑by‑Line Reading Plan 📖 (5 h/day, 10 days)

> 👋 **Starting here? Read this first (30 seconds):**
> - This is your **main day‑by‑day guide**. Do it in order: **Day 0 → Day 1 → … → Day 10.**
> - **Day 0 (below) teaches every basic word** you need — do NOT skip it unless you already
>   know C++ and shared memory.
> - You do **not** need to read the other files first. When a step needs one, it links you there.
> - **Honest expectation:** if you have *never seen C++ at all*, Days 1–2 and the "big picture"
>   are fully understandable, but the deep code days (3, 4, 8) will be tougher — that's normal.
>   Read the plain‑English explanation in each step and the diagrams in
>   [MENTAL_MODEL.md](MENTAL_MODEL.md); you'll still understand *what* the code does even if some
>   C++ syntax feels new. Consider spending 1–2 extra hours on a free "C++ in 1 hour" video
>   before Day 3 if you've truly never coded in C++.

This is the **exact, sequential** plan. Every step tells you: **open this file → read these
lines → look for this → when done, go to the next step.** No vague pointers. Just do them in
order and tick the box.

> 🎬 **Keep it fun, not a chore:** each day opens with a 3‑line story block —
> 📖 *Story so far* (where you are in the journey), 🚗 *Where this fits* (a real car/app example
> so the code isn't abstract), and 💡 *Aha you're chasing* (the one cool insight to hunt for that
> day). If a day ever feels dry, jump to [MENTAL_MODEL.md](MENTAL_MODEL.md) for the picture‑book
> version, then come back here to do the work. **This file is your workbench, not a novel — you
> learn by *doing* the steps, not just reading them.**

> All line numbers were verified against the current source. If a line is off by a little
> (code moves), the **named function/class** in the step is the true anchor — search for it.

**How to work a step:** open the file, jump to the line range (📄), read it, then use the
👀 **look‑for** line to find the key part and the ✅ **confirm** line to check you understood.
Then follow **➡️ Next** to the next step.

Legend: `📄 open` · `👀 look for` · `✅ confirm` · `➡️ next`

---

# DAY 0 — Prerequisites & vocabulary (READ THIS FIRST) 🧱 · ~5h

> Skip this day **only** if you already know C++ (templates, pointers, `std::move`), what a
> "process" is, and how to run Bazel. Otherwise, do it — the rest of the plan assumes these.

### 🗓️ Day‑0 schedule at a glance (so you know where the 5 hours go)
| Block | Time | What you do | Where |
|-------|------|-------------|-------|
| A | 45m | Learn the 8 core words + the 5 C++ ideas | Part 1 + Part 2 |
| B | 90m | Tiny C++ crash course — read & run 6 examples | Part 2c |
| C | 20m | Skim the jargon buster (just know it exists) | Part 2b |
| D | 30m | Get Bazel building (prove your setup works) | Part 3 |
| E | 30m | Read the "whole system in one paragraph" + look at MENTAL_MODEL diagrams | Part 4 |
| F | 30m | Answer the Day‑0 check out loud | bottom of Day 0 |

> If C++ is totally new, spend the extra hour on a free "C++ basics in 1 hour" video during Block B.

## Part 1 — The 8 words that appear everywhere

Read these once. You don't need to master them, just recognise them.

| Word | Explain like you're 10 | Why it matters here |
|------|------------------------|---------------------|
| **Process** | One running program. Each has its own private memory — like separate houses. | LoLa's whole job is letting two *processes* share data. |
| **IPC** (Inter‑Process Communication) | Two separate programs talking to each other. | LoLa is an IPC library. |
| **Shared memory** | A room both houses can walk into. Put something there, both see it. | This is *how* LoLa is fast — no copying. |
| **Zero‑copy** | Nobody carries the data; it's read where it lies. | The "Lo(w) La(tency)" in LoLa. |
| **Pointer** | A sticky note saying "the thing is over there," not the thing itself. | `SamplePtr` / `SampleAllocateePtr` are pointers into shared memory. |
| **Namespace** | A folder for names, so two `Foo`s don't clash. Written `score::mw::com`. | Everything lives in `score::mw::com`. |
| **Template** | A recipe with a blank you fill in later, e.g. `Box<T>` → `Box<int>`. Written `<...>`. | `AsSkeleton<MyInterface>` fills the blank with your interface. |
| **`std::move`** | "I'm giving this away, don't copy it." Hands over ownership cheaply. | Used when sending samples so nothing is duplicated. |

## Part 2 — The 5 C++ things you'll keep seeing

Don't memorise syntax; just know what each *does*.

1. **`using NewName = LongName;`** — a nickname. `using AsSkeleton = impl::AsSkeleton;` means
   "when I say `AsSkeleton`, I mean that long thing."
2. **`class`** — a bundle of data + functions. The provider and consumer are objects of classes.
3. **`unique_ptr<T>`** — a pointer that owns one thing and auto‑deletes it. Only one owner allowed.
4. **`Result<T>`** — a box that holds **either** a value **or** an error. You must ask
   `.has_value()` before using it. LoLa never throws exceptions; it returns `Result` instead.
5. **Inheritance (`class B : public A`)** — "B is a kind of A and gets all of A's abilities."
   This is the single most important idea for understanding LoLa (Day 3).

> 🧠 If only ONE sticks: **`Result<T>` = value‑or‑error box; always check `.has_value()`.**
> You'll see it on literally every LoLa call.

## Part 2c — A tiny C++ crash course (do this if C++ is new) 🧪 · ~90m

You don't need to be a C++ expert — you need to *recognise* 6 shapes. For each one below,
read the 3‑line example and say out loud what it does. If you want, paste them into a free
online compiler (e.g. godbolt.org or any "online C++ compiler") and run them.

**1. A function returns a value**
```cpp
int add(int a, int b) { return a + b; }   // add(2,3) gives 5
```
👉 `int` before the name = "what type it gives back." `(int a, int b)` = the inputs.

**2. A class = data + functions bundled together**
```cpp
class Dog {
  public:
    int age;                 // data
    void bark() { /*...*/ }   // function
};
Dog d;  d.age = 3;  d.bark();
```
👉 In LoLa, the provider and consumer are objects of classes, just bigger.

**3. A template = a class/function with a "fill‑in‑the‑blank" type**
```cpp
template <typename T>
class Box { T item; };
Box<int> b1;      // a box holding an int
Box<string> b2;   // a box holding text
```
👉 `AsSkeleton<HelloWorldInterface>` is exactly this: fill the blank with your interface.

**4. A pointer = an address, not the thing**
```cpp
int x = 10;
int* p = &x;     // p "points at" x
*p = 20;         // change x through the pointer; now x == 20
```
👉 LoLa's `SamplePtr` points *into shared memory* — that's how two programs share one value.

**5. Inheritance = "is a kind of," gets the parent's abilities** ⭐ most important
```cpp
class Animal { public: void breathe() {} };
class Dog : public Animal { public: void bark() {} };
Dog d;  d.breathe();  d.bark();   // Dog got breathe() for FREE from Animal
```
👉 This is the WHOLE reason a LoLa skeleton object has `OfferService()` even though you never
wrote it — it inherited it. (You'll see this on Day 3.)

**6. `Result<T>` = a box holding value‑OR‑error (no exceptions)**
```cpp
Result<int> r = mightFail();
if (r.has_value()) { use(r.value()); }   // success path
else               { log(r.error());  }   // failure path
```
👉 EVERY important LoLa call returns this. Always check `has_value()` first. This is the single
most repeated pattern in the entire codebase.

> ✋ If any of the 6 feel shaky, watch a free 1‑hour "C++ basics" video before Day 3. You do NOT
> need loops, memory management, or advanced C++ — just these 6 shapes.

## Part 2b — Jargon buster (bookmark this; look back whenever a word confuses you)

These short words show up in later days. You don't need them now — just come back here.

| Word | Plain meaning |
|------|---------------|
| **ctor** | short for "constructor" — the function that runs when an object is created. |
| **SHM** | short for "shared memory" (the shared room). |
| **factory** | a helper whose only job is to build objects for you (e.g. `SkeletonBindingFactory::Create`). |
| **delegate / forward** | "pass the work to someone else." e.g. `FindService` just calls the runtime's version. |
| **mock / fake** | a pretend version used in tests (no real shared memory). |
| **sync vs async** | *sync* = do it now and wait for the answer. *async* = start it, get called back later. |
| **rollback / guard / `ScopeExit`** | an "undo button" that runs automatically if something fails halfway, so nothing is left half‑done. |
| **typedef / alias** | a nickname for a longer type (same as `using`). |
| **grep** | a search command. "grep in the file" just means "search the file for this word." |
| **binding** | the *how*: LoLa binding = real shared memory; mock binding = pretend. |
| **handle** | a ticket that points to one found service instance (you get it from discovery). |

## Part 3 — How to actually build & run (Bazel in 60 seconds)

- **Bazel** is the build tool here (like a fancy "make"). You run it from the repo root.
- Build something: `bazel build //path/to:target`
- Run something: `bazel run //path/to:target`
- The `//` means "start from the repo root." Targets are defined in `BUILD` files.
- 🛠️ Try it now (proves your setup works):
  ```bash
  bazel build //score/mw/com/doc/tutorial/chapter_1/...
  ```
  If that succeeds, you're ready.

## Part 4 — The whole system in one paragraph (the map)

A **provider** program puts data into a **shared‑memory** room and hangs a **sign** ("service
offered"). A **consumer** program looks for that sign (**discovery**), then reads the data
**in place** (zero‑copy). A shared **config file** (JSON) lets both agree on names. That's it —
everything in this plan is a detail of that one sentence.

**❓ Day‑0 check:** Can you say, in your own words, what a *process*, *shared memory*, and
*`Result<T>`* are? If yes → start Day 1. If no → re‑read Parts 1–2.
**✅ DAY 0 DONE.**

---

# DAY 1 — Public API surface (the front door) · 5h

`🏁 Day 1 of 10  [■□□□□□□□□□]  You're at the front door — let's open it.`

> 📖 **Story so far:** You know the big idea (provider shares data in a room, consumer reads it).
> Today you meet the *doorknobs* — the exact functions a real app touches.
>
> 🚗 **Where this fits (real car example):** Imagine a **speed sensor app** and a **dashboard app**
> in a car. The dashboard needs the speed 100×/second. Everything you learn today (`AsSkeleton`,
> `AsProxy`, `SamplePtr`) is the vocabulary those two apps use to hand speed values back and forth.
>
> 💡 **Aha you're chasing today:** "A user never touches the scary `impl/` code — there's *one*
> tiny header (`types.h`) that's the whole public menu."

> 🔑 **New words today:** *API* = the set of functions you're allowed to call (the "buttons on
> the machine"). *Header (`.h`) file* = a list of what functions/types exist. *Alias* = nickname.

### Block A (90m) — `types.h`, the only public header

**Step 1.1** — 📄 [../../types.h](../../types.h) lines **42–52**
👀 `namespace score::mw::com`, then `using InstanceIdentifier = impl::InstanceIdentifier;`
and `using InstanceSpecifier = impl::InstanceSpecifier;`
✅ You see every public name is just `using X = impl::X` (a thin re‑export).
➡️ Next: Step 1.2.

**Step 1.2** — 📄 [../../types.h](../../types.h) lines **54–84**
👀 `InstanceIdentifierContainer` (58), `FindServiceHandle` (63), `HandleType` (67),
`ServiceHandleContainer` (73), `FindServiceHandler` (79), `SubscriptionState` (84).
✅ These are the discovery + handle types. Note which are templates (`<T>`).
➡️ Next: Step 1.3.

**Step 1.3** — 📄 [../../types.h](../../types.h) lines **86–120**
👀 `SamplePtr` (88) = receive pointer, `SampleAllocateePtr` (93) = send pointer;
method ptrs (98,103); `EventReceiveHandler` (108); field tags `WithGetter/WithSetter/WithNotifier` (112,116,120).
✅ You now know the "carriers of data" and the field feature flags.
➡️ Next: Step 1.4.

**Step 1.4** — 📄 [../../types.h](../../types.h) lines **120–160**
👀 `AsProxy` (~130) and `AsSkeleton` (~130), then `GenericProxy/GenericSkeleton` and `DataTypeMetaInfo/EventInfo`.
✅ `AsProxy`/`AsSkeleton` are the two entry aliases you'll use everywhere.
➡️ Next: Block B.

### Block B (120m) — the error model + API cookbook

**Step 1.5** — 📄 [../error_handling_guide.md](../error_handling_guide.md) (whole file)
👀 The rule: no exceptions; every fallible call returns `Result<T>`; check `.has_value()`.
✅ You understand why the tutorial wraps every call in an assert on `.has_value()`.
➡️ Next: Step 1.6.

**Step 1.6** — 📄 [../../com_error_domain.h](../../com_error_domain.h) then [../../impl/com_error.h](../../impl/com_error.h)
👀 The `ComErrc` enum values (e.g. `kBindingFailure`, `kInvalidInstanceIdentifierString`).
✅ These are the exact error codes returned by `MakeUnexpected(...)` later.
➡️ Next: Step 1.7.

**Step 1.7** — 📄 [../user_facing_API_examples.md](../user_facing_API_examples.md) — read examples 1, 2, 3 and 11.
👀 `SkeletonWrapper::Create/OfferService` and `AsSkeleton` usage.
✅ You've seen the API used in isolation.
➡️ Next: Block C.

### Block C (60m) — write it yourself
**Step 1.8** — In a scratch `.cpp`, write (don't build yet) the two aliases:
`using MySkeleton = score::mw::com::AsSkeleton<...>;` and `AsProxy<...>`. Note what header you must include (`score/mw/com/types.h`).
✅ You can start a LoLa program from memory.

### Block D (30m) — Recall
**❓ Ask yourself (Day 1) — say each answer out loud, then check with a line reference:**
1. What is the *one* header file a user needs to include? (Hint: Step 1.1)
2. What's the difference between `AsProxy` and `AsSkeleton`? Which one is the sender?
3. Every public name in `types.h` follows what pattern? (`using X = impl::X`)
4. How does a LoLa function tell you it failed, if it never throws exceptions?
5. Before using the value inside a `Result<T>`, what must you always call first?
6. What's the difference between `SamplePtr` and `SampleAllocateePtr`?
7. What do the three field tags `WithGetter` / `WithSetter` / `WithNotifier` switch on?

**✅ DAY 1 DONE when you can answer all 7 without peeking.**

### 🧠 Day‑1 MCQs (pick ONE; answers hidden below)
1. `using AsSkeleton = impl::AsSkeleton<T>;` in `types.h` means:
   a) it copies the impl class  b) it creates a new runtime object
   c) it's a template alias — a nickname resolved at compile time  d) it forwards calls over the network
2. A user who includes only `score/mw/com/types.h` gets access to `impl::SkeletonBase` directly because:
   a) `types.h` re‑exports selected `impl` types via `using`  b) `impl` is public
   c) it doesn't — `SkeletonBase` isn't user‑facing  d) inheritance exposes it
3. `SampleAllocateePtr<T>` vs `SamplePtr<T>`:
   a) both are receive pointers  b) both are send pointers
   c) they are identical aliases  d) allocatee = send/write side, sample = receive/read side
4. Why does LoLa return `Result<T>` instead of throwing exceptions?
   a) exceptions are illegal in C++  b) deterministic, exception‑free error handling for safety
   c) `Result` is faster to type  d) to support network errors only
5. `ServiceHandleContainer<T>` is templated because:
   a) it needs runtime polymorphism  b) it holds handles of a service‑specific handle type
   c) templates are required for all containers  d) it stores raw bytes
6. The field tag `WithNotifier` on a field enables:
   a) `Get()` on the proxy  b) `Set()` on the proxy  c) subscribe/receive notifications on the proxy  d) nothing, it's cosmetic
7. Calling `.value()` on a `Result` that holds an error will:
   a) be undefined/abort — you must check `.has_value()` first  b) return a default  c) return the error  d) silently succeed
8. `InstanceSpecifier` vs `InstanceIdentifier` at the API level:
   a) same thing  b) identifier is a subclass  c) specifier = design‑time name, identifier = deployment identity  d) specifier is binary
9. Which is TRUE about `AsProxy`/`AsSkeleton`?
   a) they are functions you call  b) they allocate shared memory  c) they are runtime singletons  d) they are template aliases that pick a personality for your interface
10. `MethodReturnTypePtr<T>` carries a pointer to:
    a) a local stack value  b) an event slot  c) the method's return value living in shared memory  d) a config entry

<details><summary>Day‑1 answer key</summary>
1‑c · 2‑a · 3‑d · 4‑b · 5‑b · 6‑c · 7‑a · 8‑c · 9‑d · 10‑c
</details>

---

# DAY 2 — The tutorial, end to end · 5h

`🏁 Day 2 of 10  [■■□□□□□□□□]  Today you run your first real LoLa program!`

> 📖 **Story so far:** You've seen the doorknobs. Today you watch a **real working program** use
> them end to end — and you run it yourself.
>
> 🚗 **Where this fits:** This "Hello World" IS the speed-sensor→dashboard story in miniature.
> The provider = the sensor publishing values; the consumer = the dashboard reading them. Same
> shape a real car ECU uses, just with the word "message" instead of "speed".
>
> 💡 **Aha you're chasing today:** "Two totally separate programs, started independently, find
> each other and share data with **zero network, zero copying** — just a shared room and a name."

> 🔑 **New words today:**
> - *Callback / lambda* — a little function you hand to another function to run later. Written
>   `[capture](args){ body }`. In `GetNewSamples([](auto&& s){ ... }, 1)`, the `[](auto&& s){...}`
>   part is the callback that runs once per received message. Think: "here's what to DO with each sample."
> - *`memcpy`* — "copy these raw bytes from here to there." Used to put your text into the shared slot.
> - *Two terminals* — you'll run the provider in one terminal window and the consumer in another,
>   because they are two separate programs (processes).
>
> 🧠 Goal today: watch the whole story run — provider sends, consumer receives — and see how the
> config file ties them together.

### Block A (90m) — the interface + provider
**Step 2.1** — 📄 [../tutorial/chapter_1/hello_world_service.h](../tutorial/chapter_1/hello_world_service.h) lines **20–28**
👀 `template <typename Trait> class HelloWorldInterface : public Trait::Base` and
`typename Trait::template Event<FixedSizeString> message{*this, "message"};`
✅ One templated interface; `message` is an Event whose concrete type depends on `Trait`.
➡️ Next: Step 2.2.

**Step 2.2** — 📄 [../tutorial/chapter_1/provider.cpp](../tutorial/chapter_1/provider.cpp) lines **33–51**
👀 `using HelloWorldSkeleton = AsSkeleton<...>`, `InstanceSpecifier::Create`, `HelloWorldSkeleton::Create`, `OfferService()`.
✅ The create + offer sequence.
➡️ Next: Step 2.3.

**Step 2.3** — 📄 [../tutorial/chapter_1/provider.cpp](../tutorial/chapter_1/provider.cpp) lines **52–90**
👀 `message.Allocate()`, `std::memcpy(... Get()->data() ...)`, `message.Send(std::move(...))`.
✅ The allocate → fill → send loop.
➡️ Next: Block B.

### Block B (120m) — consumer + config
**Step 2.4** — 📄 [../tutorial/chapter_1/consumer.cpp](../tutorial/chapter_1/consumer.cpp) lines **32–63**
👀 `using HelloWorldProxy = AsProxy<...>`, the `FindService` retry loop, taking `handles[0]`.
✅ Discovery is a poll‑until‑found loop.
➡️ Next: Step 2.5.

**Step 2.5** — 📄 [../tutorial/chapter_1/consumer.cpp](../tutorial/chapter_1/consumer.cpp) lines **64–110**
👀 `HelloWorldProxy::Create(handle)`, `message.Subscribe(1)`, `message.GetNewSamples(cb, 1)`.
✅ create → subscribe → receive loop.
➡️ Next: Step 2.6.

**Step 2.6** — 📄 [../tutorial/chapter_1/mw_com_config.json](../tutorial/chapter_1/mw_com_config.json) (whole file)
👀 `serviceTypes` (serviceId 8711, event `message` id 1) and `serviceInstances`
(`instanceSpecifier: MyHelloWorldServiceInstance`, `numberOfSampleSlots: 10`, `maxSubscribers: 3`).
✅ The name in the config == the string in `InstanceSpecifier::Create` in both programs.
➡️ Next: Block C.

### Block C (60m) — run it
**Step 2.7** — Run provider and consumer in two terminals (targets in [../tutorial/chapter_1/BUILD](../tutorial/chapter_1/BUILD)). Watch the counter increment across processes.
✅ You saw zero‑copy IPC actually happen.

### Block D (30m) — Recall
**❓ Ask yourself (Day 2):**
1. Trace one message from `provider.cpp` `Send` all the way to `consumer.cpp` `GetNewSamples`. Name each call in order.
2. What config field makes the provider and consumer agree they mean the *same* service? (`instanceSpecifier`)
3. Why does the consumer run `FindService` in a **loop**? What does it return when the service isn't up yet?
4. On the provider side, what are the 3 steps to send data? (Allocate → fill → Send)
5. On the consumer side, what are the 3 steps to receive? (Create(handle) → Subscribe → GetNewSamples)
6. In the config, what does `numberOfSampleSlots: 10` control? What about `maxSubscribers: 3`?
7. Are the provider and consumer the **same** program or **two different** programs? Why does that matter?

**✅ DAY 2 DONE when you can answer all 7.**

### 🧠 Day‑2 MCQs
1. In `provider.cpp`, `message.Allocate()` returns a `Result`. If you skip `.has_value()` and it failed, the most likely bug is:
   a) compile error  b) silent success  c) using an invalid `SampleAllocateePtr` → crash/UB  d) network timeout
2. The consumer wraps `FindService` in a loop because:
   a) discovery is random  b) the provider may not have offered yet; it returns an empty container until then  c) to retry network packets  d) `FindService` always throws
3. `GetNewSamples([](auto&& s){...}, 1)` — the `1` means:
   a) wait 1 second  b) subscribe id  c) slot index  d) max number of samples to hand to the callback this call
4. If provider's config says `numberOfSampleSlots: 10` but the consumer calls `Subscribe(20)`:
   a) always fine  b) it may fail/limit — you can't hold more than the slots/subscriber budget allows  c) doubles the memory  d) ignored silently always
5. The link that makes provider and consumer meet is:
   a) same PID  b) same thread  c) identical `instanceSpecifier` string in both + config  d) shared global variable
6. `std::memcpy(sample.Get()->data(), ...)` writes into:
   a) local heap  b) the shared‑memory slot that will be published  c) the config file  d) the stack
7. Provider and consumer are:
   a) two threads in one process  b) one process  c) coroutines  d) two separate processes
8. `Send(std::move(slot))` uses `std::move` because:
   a) to copy the slot  b) it's required syntax  c) to hand over ownership of the allocatee ptr without copying  d) to delete the slot
9. If the consumer starts BEFORE the provider offers, then:
   a) it errors permanently  b) `FindService` returns empty and the loop retries until the service appears  c) it crashes  d) it creates the service itself
10. The event id (`message` = 1) in config is used to:
    a) order events alphabetically  b) set priority  c) uniquely identify the event within the service across processes  d) nothing

<details><summary>Day‑2 answer key</summary>
1‑c · 2‑b · 3‑d · 4‑b · 5‑c · 6‑b · 7‑d · 8‑c · 9‑b · 10‑c
</details>

---

# DAY 3 — Traits + wrapper magic (HARD — go slow) · 5h

`🏁 Day 3 of 10  [■■■□□□□□□□]  You can already build & run a service — now the magic.`

> 📖 **Story so far:** You've *used* `AsSkeleton` and `AsProxy`. Today you lift the hood and see
> the clever trick that makes them work. This is the "wow" day.
>
> 🚗 **Where this fits:** In a real project a code generator spits out dozens of service
> interfaces (BrakeService, RadarService, …). This one trait trick means the SAME generated
> interface works as both the sender (in the sensor app) and the receiver (in the dashboard app)
> — no duplicated code. That's a huge real-world maintenance win.
>
> 💡 **Aha you're chasing today:** "One interface + a swappable 'personality' = it becomes a
> sender OR a receiver, decided by the compiler for free. And the object 'magically' has
> `OfferService()` only because of plain inheritance."

> 🔑 **New words today:** *Trait* = a "personality pack" you plug into a template to change what
> it becomes. *Wrapper* = a class that puts a thin extra layer on top of another (here it adds
> the `Create()` function). *Inheritance* (from Day 0) is the key — re‑read Day‑0 Part 2 #5 if fuzzy.
>
> 🧠 One‑line goal for today: understand how **one** interface class can become **two** different
> things (a sender or a receiver) just by plugging in a different trait.

### Block A (90m) — the trait plug
**Step 3.1** — 📄 [../../impl/traits.h](../../impl/traits.h) lines **99–129** (the doc comment)
👀 The pattern: `class TheInterface : public Trait::Base` with `Trait::template Event<...>`.
✅ You understand the *shape* every interface must follow.
➡️ Next: Step 3.2.

**Step 3.2** — 📄 [../../impl/traits.h](../../impl/traits.h) lines **140–152** (`ProxyTrait`)
👀 `using Base = ProxyBase;`, `Event = ProxyEvent`, `Field = ProxyField`, `Method = ProxyMethod`.
✅ Proxy personality = receive types.
➡️ Next: Step 3.3.

**Step 3.3** — 📄 [../../impl/traits.h](../../impl/traits.h) lines **163–175** (`SkeletonTrait`)
👀 `using Base = SkeletonBase;`, `Event = SkeletonEvent`, `Field = SkeletonField`, `Method = SkeletonMethod`.
✅ Skeleton personality = send types. **This is the whole "two personalities" trick.**
➡️ Next: Block B.

### Block B (120m) — the wrapper adds `Create`
**Step 3.4** — 📄 [../../impl/traits.h](../../impl/traits.h) line **183**
👀 `class SkeletonWrapperClass : public Interface<SkeletonTrait>`
✅ Wrapper INHERITS the interface (which inherits `SkeletonBase`). That's why one object has everything.
➡️ Next: Step 3.5.

**Step 3.5** — 📄 [../../impl/traits.h](../../impl/traits.h) lines **200–214** (`Create(InstanceSpecifier)`)
👀 resolve name → `GetInstanceIdentifier`; on failure `MakeUnexpected(kInvalidInstanceIdentifierString)`; else call the identifier overload.
✅ Step 1 of create = name→identity.
➡️ Next: Step 3.6.

**Step 3.6** — 📄 [../../impl/traits.h](../../impl/traits.h) lines **225–253** (`Create(InstanceIdentifier)`)
👀 `SkeletonBindingFactory::Create` (233); build wrapper (241); `AreBindingsValid()` check; return.
✅ Step 2 of create = build binding + validate.
➡️ Next: Step 3.7.

**Step 3.7** — 📄 [../../impl/traits.h](../../impl/traits.h) lines **328–396** (`ProxyWrapperClass` + its `Create`)
👀 Same pattern but starts from a `HandleType` (345/388), uses `ProxyBindingFactory::Create` (396).
✅ Proxy create starts from a discovered handle, not a name.
➡️ Next: Step 3.8.

**Step 3.8** — 📄 [../../impl/traits.h](../../impl/traits.h) lines **461–467**
👀 `using AsProxy = ProxyWrapperClass<T>;` (462) and `using AsSkeleton = SkeletonWrapperClass<T>;` (467).
✅ Closes the loop back to Day‑1's `types.h` aliases.
➡️ Next: Block C.

### Block C (60m) — draw it
**Step 3.9** — On paper draw: `SkeletonWrapperClass → HelloWorldInterface<SkeletonTrait> → SkeletonBase`.
Mark which layer provides `Create`, `message`, and `OfferService`.
✅ You can reproduce the chain unaided.

### Block D (30m) — Recall
**❓ Ask yourself (Day 3):**
1. What is a *trait*, in your own words? What does plugging in `SkeletonTrait` vs `ProxyTrait` change?
2. When `HelloWorldInterface` is used with `SkeletonTrait`, what concrete type does `message` become? With `ProxyTrait`?
3. Draw the inheritance chain for a skeleton. Which layer gives you `Create`? Which gives `message`? Which gives `OfferService`?
4. Why does `instance.OfferService()` compile even though `OfferService` isn't written in `HelloWorldInterface`? (cite traits.h:183 + skeleton_base.h:76)
5. `Skeleton::Create` starts from a **name** (`InstanceSpecifier`). `Proxy::Create` starts from a **______**? Why the difference?
6. What are the two things `AsSkeleton` and `AsProxy` are just nicknames for? (traits.h:462/467)
7. What does "zero runtime cost" mean for traits — when is the personality decided, compile time or run time?

**✅ DAY 3 DONE when you can answer all 7 (this is the hardest day — take your time).**

### 🧠 Day‑3 MCQs (the trickiest set — read carefully)
1. `class SkeletonWrapperClass : public Interface<SkeletonTrait>` — the object ends up with `OfferService()` because:
   a) the wrapper defines it  b) `Interface<SkeletonTrait>` derives from `SkeletonBase` which defines it → inherited  c) the trait defines it  d) macro magic
2. With `Trait = SkeletonTrait`, the member `typename Trait::template Event<T> message` becomes:
   a) `ProxyEvent<T>`  b) `EventBase`  c) `SkeletonEvent<T>`  d) `std::function`
3. Deciding proxy‑vs‑skeleton personality happens at:
   a) run time via if/else  b) link time  c) install time  d) compile time via the trait template parameter
4. `SkeletonWrapperClass::Create(InstanceSpecifier)` internally calls the `InstanceIdentifier` overload after:
   a) opening shared memory  b) resolving the specifier to an identifier via `GetInstanceIdentifier`  c) offering the service  d) subscribing
5. `ProxyWrapperClass::Create` starts from a `HandleType` (not a name) because:
   a) names are illegal on proxies  b) handles are faster to type  c) by creation time discovery already resolved a concrete live instance  d) proxies don't use config
6. If `SkeletonBindingFactory::Create` returns `nullptr`, `Create` returns:
   a) a valid empty wrapper  b) throws  c) retries forever  d) `MakeUnexpected(kBindingFailure)`
7. `using AsSkeleton = SkeletonWrapperClass<T>;` — `T` here is:
   a) a data type like int  b) a template‑template interface (`template<class> class`)  c) a runtime value  d) a namespace
8. Two different interfaces `A` and `B` used as `AsSkeleton<A>` and `AsSkeleton<B>` produce:
   a) the same type  b) a runtime error  c) two distinct compile‑time types  d) a shared object
9. Why must the interface be templated on `Trait` rather than hard‑coding `SkeletonBase`?
   a) to allow the SAME interface to be reused as a proxy too  b) to save memory  c) C++ requires it  d) for logging
10. `AreBindingsValid()` is checked inside `Create` so that:
    a) offer happens automatically  b) a wrapper with a broken event/field binding is never returned as success  c) discovery starts  d) tracing turns on
11. The wrapper adds which capability that the plain interface lacks?
    a) `message`  b) `Trait::Base`  c) `OfferService`  d) the static `Create` factory + lifetime management

<details><summary>Day‑3 answer key</summary>
1‑b · 2‑c · 3‑d · 4‑b · 5‑c · 6‑d · 7‑b · 8‑c · 9‑a · 10‑b · 11‑d
</details>

---

# DAY 4 — Provider core: `OfferService` internals · 5h

`🏁 Day 4 of 10  [■■■■□□□□□□]  You understand the magic trick — now the engine room.`

> 📖 **Story so far:** You called `OfferService()` on Day 2 and it "just worked." Today you find
> out everything it secretly does in the half-second it runs.
>
> 🚗 **Where this fits:** When the brake-control ECU boots, it calls `OfferService()` once. If
> ANY step fails (no shared memory, a missing handler), the car must NOT end up half-offering a
> safety service. Today's "rollback guards" are literally what keeps a safety system safe on a
> bad startup.
>
> 💡 **Aha you're chasing today:** "`OfferService` is like a checklist with an undo button at
> every step — if step 5 fails, steps 1–4 automatically un-happen. No half-broken services."

> 🔑 **New words today** (all in the Day‑0 jargon buster): *ctor*, *SHM*, *factory*,
> *rollback/ScopeExit guard*, *mock*. 🧠 Goal: see exactly what `OfferService()` does, in order,
> and how it safely **undoes itself** if a step fails halfway.
>
> 💡 Big idea of the day: `OfferService` sets up several things (shared memory, each event,
> discovery sign). If step 4 fails, steps 1–3 must be undone — that's what the "guards" do.

Read [API_DEEP_DIVE.md](API_DEEP_DIVE.md) Part 1 first (30m), then trace the real code:

### Block A (90m) — the class shape
**Step 4.1** — 📄 [../../impl/skeleton_base.h](../../impl/skeleton_base.h) lines **45–61**
👀 `class SkeletonBase`; the `SkeletonEvents/Fields/Methods` map typedefs (48/50/52); ctor (61) takes `unique_ptr<SkeletonBinding>` + `InstanceIdentifier`.
✅ The skeleton OWNS the binding + maps of its events/fields/methods.
➡️ Next: Step 4.2.

**Step 4.2** — 📄 [../../impl/skeleton_base.h](../../impl/skeleton_base.h) lines **76–113**
👀 `OfferService()` (76), `StopOfferService()` (83), `AreBindingsValid()` (91),
private `OfferServiceEvents/Fields` (112/113), and `GetInstanceIdentifier` free function (145).
✅ The public surface + private helpers you'll trace next.
➡️ Next: Block B.

### Block B (120m) — `OfferService` step by step (the crown jewel)
**Step 4.3** — 📄 [../../impl/skeleton_base.cpp](../../impl/skeleton_base.cpp) lines **151–164**
👀 mock short‑circuit (155); build event/field binding maps; then `binding_->PrepareOffer(...)` (164) → this creates the shared memory.
✅ SHM region is created by the binding here.
➡️ Next: Step 4.4.

**Step 4.4** — 📄 [../../impl/skeleton_base.cpp](../../impl/skeleton_base.cpp) lines **171–186**
👀 `utils::ScopeExit binding_offer_guard` (171) = auto‑rollback; then `OfferServiceEvents()` (175) and `OfferServiceFields()` (181).
✅ Each event/field is prepared, each arming its own rollback guard.
➡️ Next: Step 4.5.

**Step 4.5** — 📄 [../../impl/skeleton_base.cpp](../../impl/skeleton_base.cpp) lines **103–124** (`OfferServiceEvents`)
👀 loop over `events_`; call `skeleton_event.PrepareOffer()` (110); push a `PrepareStopOffer` guard.
✅ This is where each event actually gets offered to the binding.
➡️ Next: Step 4.6.

**Step 4.6** — 📄 [../../impl/skeleton_base.cpp](../../impl/skeleton_base.cpp) lines **187–219**
👀 `VerifyAllMethodHandlersRegistered()` (190); `GetServiceDiscovery().OfferService(instance_id_)` (197) = drops the discovery flag; `service_offered_flag_.Set()` (207); `release_guards(...)` (218/219) = success, cancel rollback.
✅ Order: binding → events → fields → verify methods → **register in discovery** → release guards.
➡️ Next: Step 4.7.

### Block C (60m) — the event base + support utils
**Step 4.7** — 📄 [../../impl/skeleton_event_base.h](../../impl/skeleton_event_base.h) lines **34–92**
👀 `class SkeletonEventBase` (34); `PrepareOffer()` (63) calls `binding_->PrepareOffer(...)` (65); `binding_` member (92); `sample_allocatee_tracker_` (107).
✅ The base just forwards to the binding + tracks outstanding allocations.
➡️ Next: Step 4.8.

**Step 4.8** — 📄 [../../impl/reference_to_moveable.h](../../impl/reference_to_moveable.h) (skim) then [../../impl/flag_owner.h](../../impl/flag_owner.h) (whole)
👀 `FlagOwner` = the move‑aware flag behind `service_offered_flag_` / `is_service_owner_`.
✅ You understand how a moved skeleton avoids double‑stop‑offer.
➡️ Next: Block D.

### Block D (30m) — Recall
**❓ Ask yourself (Day 4):**
1. List the ordered steps of `OfferService` from start to finish (binding → events → fields → verify methods → discovery → release guards).
2. On which line does the service actually get registered in discovery (the "sign goes up")? (197)
3. What is a `ScopeExit` / rollback guard, and *why* does `OfferService` need several of them?
4. If registering in discovery (step near line 197) fails, what happens to the shared memory and events created earlier?
5. What does the skeleton object *own*? (the binding + maps of events/fields/methods)
6. When a mock is injected, what does `OfferService` do differently? (line 155)
7. What is `FlagOwner` for, and why does a *moved* skeleton not accidentally stop-offer twice?

**✅ DAY 4 DONE when you can answer all 7.**

### 🧠 Day‑4 MCQs
1. The correct order inside `OfferService` is:
   a) discovery → binding → events  b) events → discovery → binding  c) binding.PrepareOffer → events → fields → verify methods → discovery.OfferService → release guards  d) release guards → binding → events
2. The `utils::ScopeExit binding_offer_guard` exists to:
   a) log timing  b) auto‑rollback the binding offer if a LATER step fails before success  c) allocate slots  d) start tracing
3. If `GetServiceDiscovery().OfferService(instance_id_)` (line ~197) fails:
   a) the already‑prepared events/binding stay leaked  b) it retries  c) the scope‑exit guards fire and undo the earlier offers  d) it throws
4. `release_guards(...)` at the end is called because:
   a) the offer failed  b) to free memory  c) to stop tracing  d) success reached — cancel the rollbacks so they DON'T undo the good offer
5. When `skeleton_mock_ != nullptr`, `OfferService`:
   a) still creates SHM  b) short‑circuits to the mock's `OfferService`  c) crashes  d) ignores it
6. `binding_->PrepareOffer(...)` is the step that:
   a) writes the discovery flag  b) subscribes  c) creates/opens the shared‑memory region for the events  d) validates config
7. The skeleton owns event/field/method **binding pointers** via maps; `OfferService` collects them first because:
   a) to sort them  b) the binding's `PrepareOffer` needs all element bindings together  c) to copy data  d) for logging
8. `VerifyAllMethodHandlersRegistered()` runs AFTER field `PrepareOffer` because:
   a) fields need slots  b) methods are events  c) random order  d) field getters register their method handlers during `PrepareOffer`, so the check must come after
9. `FlagOwner` (`service_offered_flag_`) matters on MOVE because:
   a) it doubles memory  b) the moved‑from object must NOT stop‑offer in its destructor — ownership transfers  c) it copies the flag  d) it disables tracing
10. If one event's `PrepareOffer` fails mid‑loop:
    a) others stay offered forever  b) it skips that event silently  c) `OfferService` returns an error and guards roll back everything prepared so far  d) it retries that event

<details><summary>Day‑4 answer key</summary>
1‑c · 2‑b · 3‑c · 4‑d · 5‑b · 6‑c · 7‑b · 8‑d · 9‑b · 10‑c
</details>

---

# DAY 5 — Consumer core: find / subscribe / receive · 5h

`🏁 Day 5 of 10  [■■■■■□□□□□]  Halfway! Provider side done — now the consumer.`

> 📖 **Story so far:** You've seen how the provider offers. Today you flip to the other side:
> how the consumer finds it and starts receiving.
>
> 🚗 **Where this fits:** The dashboard app might boot BEFORE the speed sensor. It can't just
> crash. Today's `FindService` loop + `StartFindService` callback are exactly how a real app
> waits patiently for a service to appear (and reacts if it disappears while driving).
>
> 💡 **Aha you're chasing today:** "The consumer code is surprisingly *thin* — it barely does
> anything itself; it just politely asks the Runtime's discovery to do the work."

> 🔑 **New words today:** *delegate/forward* (pass work to the runtime), *sync vs async*
> discovery, *handle* (ticket to a found service). 🧠 Goal: see that the consumer side is thin —
> it mostly forwards to the Runtime's ServiceDiscovery — and understand subscribe vs receive.

Read [API_DEEP_DIVE.md](API_DEEP_DIVE.md) Part 2 first (30m).

### Block A (90m) — `ProxyBase` discovery API
**Step 5.1** — 📄 [../../impl/proxy_base.h](../../impl/proxy_base.h) lines **39–58**
👀 `class ProxyBase` (39); ctor (52) takes `unique_ptr<ProxyBinding>` + `HandleType`.
✅ A proxy is built FROM a handle (already discovered).
➡️ Next: Step 5.2.

**Step 5.2** — 📄 [../../impl/proxy_base.h](../../impl/proxy_base.h) lines **70–112**
👀 `FindService(InstanceSpecifier)` (70) & `(InstanceIdentifier)` (80); `StartFindService` (92); `StopFindService`.
✅ Sync vs async discovery entry points.
➡️ Next: Step 5.3.

**Step 5.3** — 📄 [../../impl/proxy_base.cpp](../../impl/proxy_base.cpp) lines **44–96**
👀 every `FindService`/`StartFindService`/`StopFindService` delegates to
`Runtime::getInstance().GetServiceDiscovery().<same method>` and maps errors.
✅ ProxyBase is a thin wrapper over the Runtime's ServiceDiscovery.
➡️ Next: Block B.

### Block B (120m) — subscribe + receive
**Step 5.4** — 📄 [../../impl/proxy_event_base.h](../../impl/proxy_event_base.h)
👀 (grep in file) `class ProxyEventBase`, `Subscribe`, `GetSubscriptionState`, `GetNewSamples`, `SetReceiveHandler`, `GetFreeSampleCount`.
✅ The receive‑side public surface.
➡️ Next: Step 5.5.

**Step 5.5** — 📄 [../../impl/proxy_base.cpp](../../impl/proxy_base.cpp) lines **100–120** (`AreBindingsValid`)
👀 checks every event/field binding construction result.
✅ Why `Create(handle)` can fail even after discovery succeeded.
➡️ Next: Block C.

### Block C (60m) — tutorials 3–5
**Step 5.6** — Read & run: [../tutorial/chapter_3/](../tutorial/chapter_3/) (async `StartFindService`),
[../tutorial/chapter_4/](../tutorial/chapter_4/) (subscription state), [../tutorial/chapter_5/](../tutorial/chapter_5/) (polling vs callback).
✅ You've used both discovery styles and both receive styles.

### Block D (30m) — Recall
**❓ Ask yourself (Day 5):**
1. Where does `FindService` actually do its work? (proxy_base.cpp:46 → Runtime → ServiceDiscovery)
2. What's the difference between `FindService` and `StartFindService`? When would you use each?
3. A proxy is built from a **______**, not a name. Where did that come from?
4. Why can `Proxy::Create(handle)` still fail *after* discovery already succeeded? (AreBindingsValid)
5. What does `Subscribe(n)` reserve, and what does the number `n` mean?
6. What are the two ways to receive samples? (polling with `GetNewSamples` vs a callback handler)
7. Is `ProxyBase` doing the heavy lifting itself, or forwarding to something else? To what?

**✅ DAY 5 DONE when you can answer all 7.**

### 🧠 Day‑5 MCQs
1. `ProxyBase::FindService` mostly:
   a) opens shared memory itself  b) parses config  c) delegates to `Runtime::getInstance().GetServiceDiscovery().FindService`  d) subscribes
2. `FindService` vs `StartFindService`:
   a) both async  b) FindService = snapshot now (sync); StartFindService = callback on availability changes (async)  c) both sync  d) StartFindService is deprecated
3. A `ProxyBase` is constructed from:
   a) an `InstanceSpecifier`  b) a config file  c) a `SkeletonBase`  d) a `ProxyBinding` + a `HandleType`
4. `AreBindingsValid()` means `Create(handle)` can fail because:
   a) discovery failed  b) even after discovery, an event/field binding construction can fail  c) the config is missing  d) the mock is off
5. `Subscribe(n)` primarily:
   a) sends data  b) offers the service  c) reserves capacity/registers this consumer so up to n samples can be held  d) starts discovery
6. Two receive styles are:
   a) push only  b) polling (`GetNewSamples`) vs event callback handler (`SetReceiveHandler`)  c) TCP vs UDP  d) sync vs config
7. `StartFindService` returns a `FindServiceHandle` so you can:
   a) read data  b) allocate slots  c) identify the event  d) later `StopFindService(handle)` to cancel the ongoing search
8. On error, `ProxyBase` maps binding errors to:
   a) raw exceptions  b) `MakeUnexpected(ComErrc::...)` results  c) `nullptr`  d) empty strings
9. `GetFreeSampleCount()` tells you:
   a) total slots configured  b) subscriber count  c) how many more samples you can currently take before hitting your budget  d) event id
10. The consumer is "thin" means:
    a) it does all IPC itself  b) it has no state  c) it's single‑threaded  d) most work is forwarded to Runtime/ServiceDiscovery and the binding

<details><summary>Day‑5 answer key</summary>
1‑c · 2‑b · 3‑d · 4‑b · 5‑c · 6‑b · 7‑d · 8‑b · 9‑c · 10‑d
</details>

---

# DAY 6 — Discovery, identity & config · 5h

`🏁 Day 6 of 10  [■■■■■■□□□□]  You know both sides — now how they find each other.`

> 📖 **Story so far:** Both sides call "find" and "offer" — but HOW do two separate programs
> actually locate each other? Today you find the real mechanism.
>
> 🚗 **Where this fits:** A car has ONE config file describing every service (which app offers
> what, how many data slots, who's allowed to listen). Change the deployment (move a service to
> another chip) and you edit the config — not the code. Today shows why that flexibility exists.
>
> 💡 **Aha you're chasing today:** "There's no magic network — one program literally drops a
> **file on disk** saying 'I'm here', and the other watches for that file. Beautifully simple."

### Block A (90m) — the discovery implementation
**Step 6.1** — 📄 [../../impl/i_service_discovery.h](../../impl/i_service_discovery.h) (whole)
👀 the interface: `OfferService`, `StopOfferService`, `StartFindService`, `FindService`.
✅ The contract ProxyBase/SkeletonBase call into.
➡️ Next: Step 6.2.

**Step 6.2** — 📄 [../../impl/service_discovery.cpp](../../impl/service_discovery.cpp)
👀 (grep) `FindService`, `StartFindService`, `OfferService` — see it resolve specifier→identifiers and delegate to a binding‑specific `IServiceDiscoveryClient`.
✅ Discovery is per‑binding under the hood.
➡️ Next: Block B.

### Block B (120m) — identity + config + flag files
**Step 6.3** — 📄 [../../impl/instance_specifier.h](../../impl/instance_specifier.h) then [../../impl/instance_identifier.h](../../impl/instance_identifier.h)
👀 `InstanceSpecifier::Create` (design name) vs `InstanceIdentifier` (deployment identity).
✅ The two identity types and how you get each.
➡️ Next: Step 6.4.

**Step 6.4** — 📄 [../../impl/configuration/configuration.h](../../impl/configuration/configuration.h) then [../../impl/configuration/config_parser.h](../../impl/configuration/config_parser.h)
👀 `Configuration` holds service types + instances parsed from `mw_com_config.json`.
✅ How the JSON becomes in‑memory identity mappings.
➡️ Next: Step 6.5.

**Step 6.5** — 📄 [../../impl/bindings/lola/shm_path_builder.h](../../impl/bindings/lola/shm_path_builder.h) + [../../impl/bindings/lola/service_discovery/](../../impl/bindings/lola/service_discovery/)
👀 how offered services become **files on disk** that consumers watch.
✅ The physical cross‑process discovery mechanism.
➡️ Next: Block C.

### Block C (60m) — tutorials 2, 10, 11
**Step 6.6** — Read [../tutorial/chapter_2/](../tutorial/chapter_2/), [../tutorial/chapter_10/](../tutorial/chapter_10/), [../tutorial/chapter_11/](../tutorial/chapter_11/).
✅ Config refinement, access control, and skipping the specifier via a direct identifier.

### Block D (30m) — Recall
**❓ Ask yourself (Day 6):**
1. Trace `"MyHelloWorldServiceInstance"` all the way: name → config → identifier → flag file → handle.
2. What's the difference between an `InstanceSpecifier` and an `InstanceIdentifier`? Which is design‑time, which is deploy‑time?
3. Where does the provider's "I'm offering this service" signal physically live? (a file on disk)
4. How does the config JSON turn into something the code can use? (Configuration + config_parser)
5. Does `ServiceDiscovery` do the discovery itself, or hand off to a per‑binding client?
6. Two processes can't share pointers — so how do they discover each other across process boundaries?
7. What does "access control" (Chapter 10) let you restrict, and why would a safety system want it?

**✅ DAY 6 DONE when you can answer all 7.**

### 🧠 Day‑6 MCQs
1. The specifier→identifier resolution uses:
   a) DNS  b) the parsed `Configuration` from `mw_com_config.json`  c) environment variables  d) hard‑coded IDs
2. Cross‑process "service is offered" is signalled physically by:
   a) a TCP socket  b) a shared global  c) flag/lock files on disk under a SHM path (built by `shm_path_builder`)  d) a mutex only
3. `ServiceDiscovery` itself:
   a) does all binding work  b) delegates to a per‑binding `IServiceDiscoveryClient`  c) parses JSON  d) allocates slots
4. `InstanceIdentifier` differs from `InstanceSpecifier` in that it:
   a) is a design‑time name  b) is human‑typed  c) is binary‑only  d) is the resolved deployment identity (concrete instance)
5. Two processes discover each other WITHOUT shared pointers because:
   a) they share a heap  b) discovery uses the filesystem (flag files) both can see  c) they use the same PID  d) magic
6. `EnrichedInstanceIdentifier` is:
   a) a user type  b) a config file  c) identifier + resolved deployment info used internally  d) an event
7. Access control (Chapter 10) lets you restrict:
   a) CPU usage  b) which instances/consumers may access a service  c) log level  d) slot count
8. `Configuration` is built by:
   a) the linker  b) the OS  c) discovery  d) the config parser reading the JSON manifest
9. If the specifier isn't in the config, `Create(specifier)` returns:
   a) a default instance  b) `MakeUnexpected(kInvalidInstanceIdentifierString)`  c) throws  d) empty proxy
10. `FindService(specifier)` may return multiple handles when:
    a) never  b) config is broken  c) the specifier maps to multiple offered instances  d) two threads call it

<details><summary>Day‑6 answer key</summary>
1‑b · 2‑c · 3‑b · 4‑d · 5‑b · 6‑c · 7‑b · 8‑d · 9‑b · 10‑c
</details>

---

# DAY 7 — Runtime singleton + mocks · 5h

`🏁 Day 7 of 10  [■■■■■■■□□□]  Meet the shared brain behind everything.`

> 📖 **Story so far:** Both proxy and skeleton kept calling `Runtime::getInstance()`. Today you
> meet that shared "brain" and learn how tests fake it.
>
> 🚗 **Where this fits:** Every app in the car has exactly one Runtime that loaded that app's
> config. And crucially — engineers must test brake logic on a laptop WITHOUT real shared memory
> or a real car. Today's "mock" is how they run the code safely at their desk.
>
> 💡 **Aha you're chasing today:** "Because everything goes through one swappable `IRuntime`, you
> can rip out the real shared-memory world and drop in a pretend one — that's how they test
> safety code without a car."

> 🔑 **New words today:** *singleton* = a class with exactly **one** shared instance for the whole
> program (like the one town hall everyone visits). *Meyers singleton* = a common safe way to make
> one. *gmock* = a testing tool that makes fake objects. 🧠 Goal: understand the one "brain" every
> proxy/skeleton talks to, and how tests swap it for a fake.

### Block A (90m) — the singleton
**Step 7.1** — 📄 [../../impl/runtime.h](../../impl/runtime.h) lines **38–60** (class doc comment)
👀 Meyers singleton; `Initialize()` overloads; `getInstance()` returns real or injected mock.
✅ Why there's exactly one runtime per process.
➡️ Next: Step 7.2.

**Step 7.2** — 📄 [../../impl/runtime.h](../../impl/runtime.h) (rest)
👀 members: `Configuration`, `ServiceDiscovery`, the `unordered_map<BindingType, IBindingRuntime>`, tracing runtime.
✅ The 4 things the runtime owns.
➡️ Next: Step 7.3.

**Step 7.3** — 📄 [../../impl/runtime.cpp](../../impl/runtime.cpp)
👀 (grep) `Initialize` — mutex lock + config parse into the singleton.
✅ How init actually happens.
➡️ Next: Block B.

### Block B (120m) — the test story
**Step 7.4** — 📄 [../../impl/i_runtime.h](../../impl/i_runtime.h) then [../../impl/runtime_mock.h](../../impl/runtime_mock.h)
👀 the interface + gmock mock used to replace the runtime in tests.
✅ How tests inject a fake brain.
➡️ Next: Step 7.5.

**Step 7.5** — 📄 [../../impl/bindings/mock_binding/](../../impl/bindings/mock_binding/)
👀 the fake binding that runs without shared memory.
✅ Why the 4‑layer split matters for testing.
➡️ Next: Block C.

### Block C (60m) — Chapter 12
**Step 7.6** — Read & run [../tutorial/chapter_12/](../tutorial/chapter_12/) (unit testing with mocks).
✅ You can unit‑test a proxy/skeleton without IPC.

### Block D (30m) — Recall
**❓ Ask yourself (Day 7):**
1. What is a *singleton*, and why does LoLa have exactly one Runtime per process?
2. Name the 4 things the Runtime owns. (config, service discovery, per‑binding runtimes, tracing)
3. How do a proxy and a skeleton both reach the *same* discovery? (through `Runtime::getInstance()`)
4. How does a unit test replace the real Runtime with a fake one? (inject a mock via `IRuntime`)
5. What is the *mock binding*, and why does it let tests run without shared memory?
6. Roughly what does `Initialize()` do, and why is a mutex involved?
7. Why is having an interface (`IRuntime`) — not just the concrete class — important for testing?

**✅ DAY 7 DONE when you can answer all 7.**

### 🧠 Day‑7 MCQs
1. The Runtime is a singleton because:
   a) to save one byte  b) one process = one deployment config = one discovery view shared by all  c) C++ needs it  d) for speed only
2. A "Meyers singleton" is:
   a) a global variable  b) a mutex  c) a function‑local static instance, lazily and safely initialised  d) a template
3. `getInstance()` can return a mock because:
   a) it always returns real  b) if a mock was injected it returns that instead of the real singleton  c) it copies config  d) it forwards to disk
4. The 4 things Runtime owns are roughly:
   a) threads, sockets, files, logs  b) events, fields, methods, proxies  c) only config  d) configuration, service discovery, per‑binding runtimes, tracing runtime
5. `Initialize()` uses a mutex because:
   a) it's slow  b) to guard one‑time config setup against concurrent init  c) to lock shared memory  d) for tracing
6. A test swaps the real runtime by:
   a) editing config  b) deleting the binary  c) using threads  d) injecting an `IRuntime` mock that `getInstance()` returns
7. The **mock binding** lets tests:
   a) use real SHM  b) run proxy/skeleton logic WITHOUT shared memory  c) skip compilation  d) use the network
8. Having `IRuntime` (an interface) matters because:
   a) it's faster  b) it saves memory  c) it enables substituting a mock for the concrete Runtime in tests  d) it's required by Bazel
9. Why do BOTH proxy and skeleton reach discovery through `Runtime::getInstance()`?
   a) coincidence  b) so they share the SAME configuration and discovery state  c) to reduce code  d) for logging
10. If two parts call `getInstance()`:
    a) two runtimes exist  b) it errors  c) config reloads  d) they get the exact same instance

<details><summary>Day‑7 answer key</summary>
1‑b · 2‑c · 3‑b · 4‑d · 5‑b · 6‑d · 7‑b · 8‑c · 9‑b · 10‑d
</details>

---

# DAY 8 — Zero‑copy shared memory (HARD — go slow) · 5h

`🏁 Day 8 of 10  [■■■■■■■■□□]  The heart of LoLa — how it's actually fast.`

> 📖 **Story so far:** You've said "zero-copy" for a week. Today you finally SEE the actual
> shared-memory machinery that makes it true. This is the heart of LoLa.
>
> 🚗 **Where this fits:** A camera app streams huge video frames to 3 other apps at 60fps.
> Copying each frame 3× would melt the CPU. Instead, the frame sits ONCE in a shared slot and all
> 3 readers look at the same bytes. Today you learn the slot + reference-count trick that makes
> that safe. This is *the* reason LoLa exists.
>
> 💡 **Aha you're chasing today:** "Nothing big ever moves between programs — they just pass a
> tiny counter around. 'Sharing' a 4MB frame costs almost nothing."

> 🔑 **New words today:** *Slot* = one numbered spot in shared memory that holds one data item.
> *Reference count* = a tally of how many readers currently hold a slot; the slot can't be reused
> until it hits zero. *Control array* = the tiny status/tally list; *storage array* = the big
> actual‑data list; they're linked by slot number.
>
> 🧠 One‑line goal: see that "zero‑copy" just means **we move a small number (the count), never
> the data itself.**

Read [API_DEEP_DIVE.md](API_DEEP_DIVE.md) 1.3, 1.4, 2.4 first (40m).

### Block A (90m) — the shared control + storage
**Step 8.1** — 📄 [../../impl/bindings/lola/event_data_control.h](../../impl/bindings/lola/event_data_control.h) lines **23–48**
👀 doc comment: two index‑linked arrays (control status vs data); `state_slots_` in shared memory.
✅ Control (status/refcount) is separate from data, linked by slot index. Understand WHY (perf).
➡️ Next: Step 8.2.

**Step 8.2** — 📄 [../../impl/bindings/lola/event_slot_status.h](../../impl/bindings/lola/event_slot_status.h) then [../../impl/bindings/lola/event_data_storage.h](../../impl/bindings/lola/event_data_storage.h)
👀 a slot's status lifecycle; the storage array holding actual payloads.
✅ How a slot is marked free / in‑writing / ready.
➡️ Next: Block B.

### Block B (120m) — allocate, send, receive
**Step 8.3** — 📄 [../../impl/bindings/lola/skeleton_event.h](../../impl/bindings/lola/skeleton_event.h) lines **73–115**
👀 `Send` (85), `Allocate` (87), `GetLatestSample` (89), `Notify` (114).
✅ The send side entry points.
➡️ Next: Step 8.4.

**Step 8.4** — 📄 [../../impl/bindings/lola/slot_collector.h](../../impl/bindings/lola/slot_collector.h) then [../../impl/bindings/lola/slot_decrementer.h](../../impl/bindings/lola/slot_decrementer.h)
👀 collector finds new slots + increments refcount; decrementer drops it when a `SamplePtr` dies.
✅ The receive side + why slots aren't reused too early.
➡️ Next: Step 8.5.

**Step 8.5** — 📄 [../../impl/plumbing/sample_ptr.h](../../impl/plumbing/sample_ptr.h) + [../../impl/sample_reference_tracker.h](../../impl/sample_reference_tracker.h)
👀 `SamplePtr` = pointer into SHM (no copy); tracker = thread‑safe refcount.
✅ "Zero‑copy" = move a refcount, not the data.
➡️ Next: Block C.

### Block C (60m) — Chapters 6 & 13
**Step 8.6** — Read [../tutorial/chapter_6/](../tutorial/chapter_6/) (sizing slots — ties to `numberOfSampleSlots`) and [../tutorial/chapter_13/](../tutorial/chapter_13/) (heap‑free ASIL‑B).
✅ You can reason about slot counts and no‑heap usage.

### Block D (30m) — Recall
**❓ Ask yourself (Day 8):**
1. Walk `Allocate → Send → GetNewSamples` in terms of slot status and reference count.
2. What actually makes this "zero‑copy"? What is the small thing that moves instead of the data?
3. Why are there **two** arrays (control vs storage) instead of one? What links them?
4. What are the possible states of a slot? (free / in‑writing / ready‑in‑use)
5. What stops the provider from reusing a slot that a consumer is still reading? (nonzero refcount)
6. What happens to a slot's refcount when a consumer's `SamplePtr` is destroyed? Who does it? (slot decrementer)
7. Where does `numberOfSampleSlots` from the config show up in this machinery?

**✅ DAY 8 DONE when you can answer all 7 (2nd hardest day — go slow).**

### 🧠 Day‑8 MCQs (deep — reason about the shared memory)
1. "Zero‑copy" means the thing that moves between processes is:
   a) the whole payload copied twice  b) a network packet  c) essentially a reference/refcount, not the data  d) a file
2. `EventDataControl` and `EventDataStorage` are two arrays linked by:
   a) pointers only  b) the slot index  c) timestamps  d) event name
3. They are kept separate because:
   a) tradition  b) to save disk  c) for tracing  d) tiny status/refcount is touched constantly; keeping it apart avoids dereferencing big data buffers
4. A slot's lifecycle is roughly:
   a) ready → free → ready  b) free → in‑writing → ready/in‑use → (refcount 0) → free  c) always ready  d) locked forever
5. A producer can NOT reuse a slot when:
   a) it's free  b) its reference count is nonzero (a consumer still holds it)  c) it's in‑writing by nobody  d) never
6. When a consumer's `SamplePtr` is destroyed:
   a) nothing  b) the slot is deleted from disk  c) a slot decrementer drops that slot's refcount  d) the provider crashes
7. `numberOfSampleSlots: 10` from config controls:
   a) subscribers  b) threads  c) bytes per sample only  d) how many slots the `EventDataControl`/storage arrays hold
8. `Allocate()` returns a `SampleAllocateePtr`; if you never `Send()` it:
   a) it's published anyway  b) a guard frees/returns the slot so it isn't leaked  c) it stays in‑writing forever  d) it crashes
9. The `SlotCollector` on the consumer:
   a) writes data  b) offers the service  c) parses config  d) finds slots newer than last‑seen and increments their refcount
10. If `maxSubscribers: 3` and a 4th subscriber joins:
    a) always fine  b) it may be rejected — the control structure is sized for the configured max  c) doubles memory  d) silently shares slot 0
11. Two consumers reading the same slot simultaneously is safe because:
    a) copies are made  b) only one can read  c) it isn't safe  d) the per‑slot refcount tracks BOTH; slot recycles only at count 0

<details><summary>Day‑8 answer key</summary>
1‑c · 2‑b · 3‑d · 4‑b · 5‑b · 6‑c · 7‑d · 8‑b · 9‑d · 10‑b · 11‑d
</details>

---

# DAY 9 — Robustness & crash recovery · 5h

`🏁 Day 9 of 10  [■■■■■■■■■□]  Almost there — what makes it a *safety* system.`

> 📖 **Story so far:** You know how data flows when everything works. Today: what happens when a
> program **crashes** mid-read — the difference between a toy and a safety system.
>
> 🚗 **Where this fits:** A passenger app crashes while holding 3 video slots. In a car that can't
> mean the camera service leaks memory until it dies too. Today's transaction log is what lets the
> system clean up a dead app's mess and keep driving — the stuff that earns the "ASIL-B" badge.
>
> 💡 **Aha you're chasing today:** "LoLa keeps a little written ledger of 'who holds what', so if
> an app dies, someone else can walk in, read the ledger, and undo its leftovers. That's why a
> crash doesn't poison the whole system."

> 🔑 **New words today:** *state machine* = something that can only be in one of a few named
> states and moves between them by rules (e.g. not‑subscribed → pending → subscribed).
> *Transaction log* = a written record of "who is holding what," so if a program crashes we can
> clean up its leftovers. 🧠 Goal: understand how LoLa survives a consumer crashing mid‑read.

### Block A (90m) — subscription as a state machine
**Step 9.1** — 📄 [../../impl/bindings/lola/subscription_state_machine.h](../../impl/bindings/lola/subscription_state_machine.h)
👀 states: not‑subscribed / subscription‑pending / subscribed.
✅ Subscription is a lifecycle, not a boolean.
➡️ Next: Step 9.2.

**Step 9.2** — 📄 [../../impl/bindings/lola/subscription_state_machine_states.h](../../impl/bindings/lola/subscription_state_machine_states.h)
👀 the per‑state transition handlers.
✅ How restart/unsubscribe move between states.
➡️ Next: Block B.

### Block B (120m) — the transaction log (the safety core)
**Step 9.3** — 📄 [../../impl/bindings/lola/transaction_log.h](../../impl/bindings/lola/transaction_log.h)
👀 a crash‑safe record of which slots a consumer holds.
✅ Why references survive a crash.
➡️ Next: Step 9.4.

**Step 9.4** — 📄 [../../impl/bindings/lola/transaction_log_set.h](../../impl/bindings/lola/transaction_log_set.h) then [../../impl/bindings/lola/transaction_log_rollback_executor.h](../../impl/bindings/lola/transaction_log_rollback_executor.h)
👀 how a dead consumer's slot references get rolled back and reclaimed.
✅ The mechanism that stops permanent slot leaks.
➡️ Next: Step 9.5.

**Step 9.5** — 📄 [../../design/partial_restart/](../../design/partial_restart/)
👀 the design rationale for restart handling.
✅ The big‑picture safety argument.
➡️ Next: Block C.

### Block C (60m) — re‑examine Chapter 4
**Step 9.6** — Re‑read [../tutorial/chapter_4/](../tutorial/chapter_4/) subscription states with the state‑machine files open side by side.
✅ Tutorial ↔ implementation connected.

### Block D (30m) — Recall
**❓ Ask yourself (Day 9):**
1. A consumer holding 3 slots suddenly crashes. What reclaims those slots? How does the provider find out?
2. What is a *state machine*? Name the subscription states and how you move between them.
3. What is the *transaction log*, and what problem does it solve?
4. Why is subscription NOT just a single true/false boolean?
5. Which component actually performs the cleanup after a crash? (rollback executor)
6. Why does a *safety‑grade* (ASIL) system especially need this crash‑recovery machinery?
7. Connect it back: how does Chapter 4's "subscription state" relate to the state‑machine files?

**✅ DAY 9 DONE when you can answer all 7.**

### 🧠 Day‑9 MCQs
1. A consumer crashes holding 3 slots. Those slots are reclaimed via:
   a) OS garbage collector  b) the transaction log + rollback executor  c) a timeout only  d) nothing — they leak
2. Subscription is modelled as a state machine because:
   a) it's fashionable  b) to save memory  c) for logging  d) it has a lifecycle (not‑subscribed → pending → subscribed) with restart/unsubscribe transitions
3. The transaction log records:
   a) log messages  b) which slots a consumer currently references (crash‑safe)  c) config  d) timings
4. Why can't subscription be a single bool?
   a) bools are slow  b) C++ lacks bool  c) tracing needs it  d) the provider may be absent/restarting, so intermediate states (pending) are needed
5. The component that performs cleanup after a crash is:
   a) the parser  b) the transaction_log_rollback_executor  c) the slot collector  d) the config
6. Partial restart matters for a safety system because:
   a) it looks nice  b) it's faster  c) it reduces code  d) a crashed/restarted participant must not permanently corrupt or leak shared state
7. The subscription state machine files relate to Chapter 4 by:
   a) unrelated  b) Chapter 4's "subscription state" behavior is implemented by that machine  c) Chapter 4 replaces it  d) both are tests
8. If rollback did NOT exist, a repeated crash‑loop would:
   a) self‑heal  b) speed up  c) reset config  d) progressively leak slots until the event is unusable
9. The transaction_log_set holds:
   a) one log  b) the set of per‑consumer transaction logs for an event/service  c) config entries  d) slots
10. Rollback is triggered when:
    a) every send  b) on subscribe  c) a dead/stale participant is detected and its references must be reclaimed  d) never

<details><summary>Day‑9 answer key</summary>
1‑b · 2‑d · 3‑b · 4‑d · 5‑b · 6‑d · 7‑b · 8‑d · 9‑b · 10‑c
</details>

---

# DAY 10 — Generic API, methods, fields, tracing + capstone · 5h

`🏁 Day 10 of 10  [■■■■■■■■■■]  Final stretch — fill the gaps and BUILD your own!`

> 📖 **Story so far:** You've mastered events (one-way streams). Today you fill in the last
> shapes — request/reply **methods**, read/write **fields** — and then BUILD something yourself.
>
> 🚗 **Where this fits:** "Set the cabin temperature to 21°" is a **field** (a value you set and
> others watch). "Calculate route to home" is a **method** (ask a question, get an answer). A
> **generic** proxy is what a logging/gateway tool uses to record every service without knowing
> its type. Today you see all three — then you extend the tutorial like a real engineer would.
>
> 💡 **Aha you're chasing today:** "Methods and fields aren't new magic — they're just events
> plus a little extra, built on everything I already learned. And I can now add my own!"

> 🔑 **New words today:** *type‑erased / generic* = working with data **without** knowing its
> exact C++ type at compile time (useful for tools/gateways). *Method* = a request‑and‑reply call
> (like asking a question and getting an answer). *Field* = a value you can read/write/watch
> (= an event plus optional get/set). *Tracing* = optional recording of what happened, for debugging.
> 🧠 Goal: fill the last gaps, then prove mastery by extending the tutorial yourself.

### Block A (90m) — type‑erased + fields/methods
**Step 10.1** — 📄 [../../impl/generic_proxy.h](../../impl/generic_proxy.h) + [../../impl/generic_skeleton.h](../../impl/generic_skeleton.h) + [../../impl/data_type_meta_info.h](../../impl/data_type_meta_info.h)
👀 offering/consuming without knowing the C++ type (used by the gateway).
✅ When/why to use the generic API.
➡️ Next: Step 10.2.

**Step 10.2** — 📄 [../../impl/methods/](../../impl/methods/)
👀 `skeleton_method_base.h`, `proxy_method_with_in_args_and_return.h` and the other 3 variants; `callable_traits.h`.
✅ Methods = request/response; one class per signature shape.
➡️ Next: Step 10.3.

**Step 10.3** — 📄 [../../impl/field_tags.h](../../impl/field_tags.h) + [../../impl/skeleton_field.h](../../impl/skeleton_field.h) + [../../impl/proxy_field.h](../../impl/proxy_field.h)
👀 a field = event (value) + optional get/set methods, selected by `WithGetter/WithSetter/WithNotifier`.
✅ Fields reuse events + methods you already learned.
➡️ Next: Step 10.4.

**Step 10.4** — 📄 [../../impl/tracing/i_tracing_runtime.h](../../impl/tracing/i_tracing_runtime.h)
👀 `RegisterServiceElement`, `RegisterShmObject`, `Trace`.
✅ Tracing is an optional observer wired in at OfferService (recall Day‑4 step 4.3).
➡️ Next: Block B.

### Block B (120m) — Chapters 7, 8, 9
**Step 10.5** — Read & run [../tutorial/chapter_7/](../tutorial/chapter_7/) (fields), [../tutorial/chapter_8/](../tutorial/chapter_8/) (methods), [../tutorial/chapter_9/](../tutorial/chapter_9/) (field Get/Set).
✅ You've exercised every service‑element kind.

### Block C (60m) — CAPSTONE
**Step 10.6** — Add a **second event** to the tutorial:
1. 📄 edit [../tutorial/chapter_1/hello_world_service.h](../tutorial/chapter_1/hello_world_service.h) — add another `Event<...> status{*this, "status"};`.
2. 📄 edit [../tutorial/chapter_1/mw_com_config.json](../tutorial/chapter_1/mw_com_config.json) — add the `status` event to both `serviceTypes` and `serviceInstances`.
3. 📄 edit provider.cpp / consumer.cpp — allocate/send and subscribe/receive it.
4. Build + run both, confirm the new event flows.
✅ You can extend a LoLa service end‑to‑end unaided.

### Block D (30m) — Final recall
**❓ Ask yourself (Day 10):**
1. What does *type‑erased / generic* mean, and when would you use `GenericProxy`/`GenericSkeleton` instead of the typed ones?
2. What is a *method* in LoLa, and how is it different from an *event*? (request/reply vs one‑way stream)
3. What is a *field*? What two things is it built from? (an event + optional get/set methods)
4. Which field tag enables `Get()`? Which enables `Set()`? Which makes it notify on change?
5. What is *tracing* for, and on which earlier day did you already see it wired in? (Day 4, `OfferService`)
6. **Capstone check:** you added a second event — list every file you had to touch and why.
7. Now take the 5‑question test in [MENTAL_MODEL.md](MENTAL_MODEL.md#part-e--60-second-self-test-can-you-answer-these) and answer each with a proof link.

### 🧠 Day‑10 MCQs
1. `GenericProxy`/`GenericSkeleton` are used when:
   a) you always know the type  b) for speed  c) only in tests  d) you must handle data WITHOUT knowing its C++ type at compile time (e.g. gateway)
2. A LoLa **method** differs from an **event** because:
   a) methods are one‑way  b) methods are request/response (call+reply); events are one‑way publish/subscribe  c) they're identical  d) events return values
3. A **field** is built from:
   a) two methods only  b) a slot only  c) a config entry  d) an event (the value) + optional get/set methods
4. `WithGetter` / `WithSetter` / `WithNotifier` respectively enable:
   a) log/trace/debug  b) `Get()` / `Set()` / change‑notification on the proxy  c) send/receive/offer  d) nothing
5. There are multiple `proxy_method_with_*` classes because:
   a) copy‑paste  b) for testing  c) versioning  d) one per method signature shape (in‑args, return, both, neither)
6. Tracing is wired in during:
   a) Subscribe  b) `OfferService` (the register‑SHM‑object callback from Day 4)  c) Create  d) never
7. `DataTypeMetaInfo` is needed by the generic API to:
   a) log  b) pick a binding  c) build config  d) describe size/layout of a type it doesn't know at compile time
8. In the capstone, adding a second event required editing:
   a) only provider.cpp  b) the interface header + config JSON (both sections) + provider + consumer  c) only config  d) the binding
9. A field with ONLY `WithSetter` (no getter/notifier) would be:
   a) fully usable  b) fastest  c) illegal to compile always  d) problematic — consumers couldn't read/observe it (at least one of getter/notifier is expected)
10. Type‑erased APIs trade:
    a) nothing  b) compile‑time type safety for runtime flexibility  c) speed for memory only  d) safety for logging

<details><summary>Day‑10 answer key</summary>
1‑d · 2‑b · 3‑d · 4‑b · 5‑d · 6‑b · 7‑d · 8‑b · 9‑d · 10‑b
</details>

**✅ DAY 10 DONE — you know LoLa root to tip.**

---

## Master progress checklist

- [ ] Day 1 — Public API (`types.h`, error model)
- [ ] Day 2 — Tutorial end‑to‑end (Ch.1)
- [ ] Day 3 — Traits + wrapper (traits.h)
- [ ] Day 4 — `OfferService` internals (skeleton_base.cpp)
- [ ] Day 5 — Proxy find/subscribe/receive (proxy_base.cpp)
- [ ] Day 6 — Discovery/identity/config (Ch.2,10,11)
- [ ] Day 7 — Runtime + mocks (Ch.12)
- [ ] Day 8 — Zero‑copy shared memory (Ch.6,13)
- [ ] Day 9 — Crash recovery (transaction log)
- [ ] Day 10 — Generic/methods/fields/tracing + capstone (Ch.7–9)

**When all 10 are ticked, you've read every architecturally important line in `mw::com`.**
