## 2a. Best Overall Fit
 
**My benchmark:**
 
_How much of the original model can be represented without building extra infrastructure by hand and without losing core semantic guarantees._
 
**Which cell and why:**
 
_Go has sync.Mutex, sync.RWMutex, sync.WaitGroup, sync/atomic, and sync.Cond all built in natively. Nothing needs to be simulated or worked around. The slides show this directly with the ccc counter example where goroutines increment and decrement a shared integer using sync.Mutex — no extra infrastructure needed at all. Every core feature of the shared memory model is present. Threads map to goroutines, locks map to sync.Mutex, and join maps to sync.WaitGroup. The semantics match one to one._
 
**Evidence from the matrix:**
 
_Compared to every other cell in the matrix, this one loses nothing functionally. Actor model in Java loses selective receive and OTP supervision. CSP in Java loses select entirely. Shared memory in Elixir is blocked at the runtime level. Actor model in Go loses supervisor trees. CSP in Elixir loses typed channels and directionality. Shared memory in Go only loses idiomatic style — not actual functionality._
 
**Counterargument and rebuttal:**
 
_A counterargument could be made that CSP in Elixir fits well because the call/reply pattern in GenServer approximates synchronous channel behavior. But call/reply sends two messages for every one exchange and has no equivalent of select. Core semantic features are still missing. In shared memory in Go nothing is missing — the only downside is that Go discourages the pattern stylistically, which is a much smaller loss than losing actual features._
 
---
 
## 2b. Worst Overall Fit
 
**My benchmark:**
 
_How deeply the host language or runtime blocks the target model at an architectural level — not just missing an API but making the core concept impossible to represent._
 
**Which cell and why:**
 
_The BEAM runtime does not allow processes to share memory. This is not a missing library or a missing API — it is a hard constraint built into how the BEAM works. The defining characteristic of shared memory is direct low-latency access to a common address space. On the BEAM that is simply not possible. Agent and ETS are the closest simulations available but both route every access through message passing or a managed runtime table. In Java two threads reading the same counter are literally touching the same memory location. In Elixir that cannot happen no matter how much extra code is written._
 
**Evidence from the matrix:**
 
_Every other cell in the matrix at least partially represents the target model. Java can fake actor mailboxes with BlockingQueue. Go can fake actor supervision loosely with channels and recover(). Elixir can fake synchronous channels with call/reply. But shared memory in Elixir cannot fake the one thing that defines shared memory — direct shared access. The simulation changes the fundamental semantics, not just the syntax._
 
**Counterargument and rebuttal:**
 
_A counterargument could be made that actor model in Go is worse because OTP supervision is also impossible to build naturally. That is a fair point - spawnlink, monitor, and named process registries are all missing in Go. But in Go we can at least build a rough supervisor by hand using goroutines and error channels. The constraint is at the library level. In Elixir the constraint is at the runtime level - no amount of hand-written code can produce actual shared memory on the BEAM. That makes Elixir's block deeper and more fundamental than Go's._
 
---