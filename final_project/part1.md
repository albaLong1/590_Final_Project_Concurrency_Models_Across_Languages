# Part 1: The Model Matrix

---

## Cell 1: Actor Model in Java

> **Model being expressed:** Actor Model (Erlang-style)
> **Language used:** Java

### 1. What maps naturally

_In Java, threads map pretty well to actors because each thread can be assigned its own private state and can indefinitely loop, processing work, which is similar to how Erlang's actors function. A rough mailbox analog is implemented in Java's BlockingQueue, since one thread puts messages in while another takes them out. Java's OO structure maps loosely to actors in that objects encapsulate state. It is similar to how Erlang's actors maintain their private state such that no actors can access it._

### 2. What requires simulation or workaround

_Mailbox mechanism must be built manually because, in Erlang, every process gets a mailbox automatically. Messages are implemented in FIFO structure, so pattern-matched selective receive is done by receive block. This process isn't in-built in Java. We need to use a BlockingQueue<Object> to simulate message receipt, then write dispatch loop with instanceof with checks to match pattern matching. In Erlang, processes can skip messages that aren't ready for and leave it for later, while in Java, every local exception is caught. In Erlang, process will die and be restarted with clean state. Simulating OTP supervision in Java means to write a thread with Thread.UncaughtExceptionHandler, restart logic, and state reinitialization. OTP does this all automatically._

### 3. What cannot be represented well or at all

_Java doesn't have Erlang's selective receive concept implemented at all. In Java, we cannot skip a message and return to it without removing it and manually putting it back in line, since Java's BlockingQueue is strictly FIFO. This is against Erlang's actor model nature, which has been done in Java to implement the same feature. Also, Java threads share heap—the isolation isn't enforced through runtime. "Actors" implemented in Java can accidentally share state through common object references, which violates the foundational guarantee of the actor model that there is no memory shared between actors. The BEAM enforces this at the runtime level; Java cannot._

---

## Cell 2: CSP / Channels in Java

> **Model being expressed:** CSP / Channels (Go-style)
> **Language used:** Java

### 1. What maps naturally

_Java's BlockingQueue from java.util.concurrent approximates a buffered Go channel reasonably well. It is typed through generics, thread-safe, and blocks the consumer when empty and the producer when full, which mirrors Go's buffered channel behavior. Java's threading model also supports the general shape of CSP-style programming since we can spawn multiple threads that communicate through shared queues, which is structurally similar to goroutines passing data through channels._

### 2. What requires simulation or workaround

_It is hard to implement Go's select. In GO, select lets a goroutine on whichever of N channel operations becomes ready, with random selction among concurrent cases. Java has no direct equivalent. Polling in a loop or building a shared multiplexing queue that accumulate signals from various sources are required for simulating. Both sender and receiver block until both are ready which means uncached channels in Go provide prearranged synchronization. Java's SynchronousQueue can only approximate this process._

### 3. What cannot be represented well or at all

_Go's select cannot be reporduced in Java without significant infrstructure. While Go channels are first-class typed values that can be passed between goroutines, stored in structs, and directionally restricted (chan<-, <-chan). Java queues are objects that can be passed around, but there is no language-level enforcement. Goroutine can accidentallu read from Java queue with no compile error while it only is supposed to read._

---

## Cell 3: Shared Memory in Elixir

> **Model being expressed:** Shared Memory (Java-style)
> **Language used:** Elixir

### 1. What maps naturally

_There is almost no maps. This is the biggest mismatch in the whole matrix. The BEAM implement process isolation at runtime level so no two processes can share memory. This is not a missing feature instead it is a hard runtime constraint. Directly reading and writing a shared memory location does not exist on the BEAM._

### 2. What requires simulation or workaround

_Elixir's Agent is the closest simulation. An Agent is a process holding a value and other processes, reading or updating it through messages. Message passing occur under the hood even though it looks like shared state from outside. ETS is another option that works like an in-memory table multiple processes can use, but is managed by the runtime. In Java, this is managed by direct memory access._

### 3. What cannot be represented well or at all

_In Java threads reading the same counter are literally touching the same memory spot. In Elixir every access goes through a process or managed table which adds overhead and changes how it actually works. Race conditions in the Java sense cannot happen on the BEAM because the runtime stops it. But that also means the main benefit of shared memory is lost completely._

---

## Cell 4: CSP / Channels in Elixir

> **Model being expressed:** CSP / Channels (Go-style)
> **Language used:** Elixir

### 1. What maps naturally

_A synchronous channel can be simulated in Elixir using the call/reply pattern shown in GenServer slides. One process sends a message with its PID and blocks in a receive waiting for a reply. The other process handles it and sends back. This is similar to Go's unbuffered channel rendezvous where both sides must be ready before anything happens._

### 2. What requires simulation or workaround

_In Go a channel is its own entity, separate from any goroutine. In Elixir there is no such concept. To simulate a channel a dedicated process must be spawned just to act as the channel, which takes a lot of extra work. The call/reply pattern also sends two messages for every one exchange, which is double the traffic of a real Go channel. Go's select also cannot be faked cleanly since Elixir's receive does pattern matching but cannot natively wait on whichever of multiple processes sends first._

### 3. What cannot be represented well or at all

_Chan int only carries integers and compiler checks it, so Go chanenels are typed. Elixir mailboxes take anything and types are only checked when receive tries to match at runtime. So there is no compile time safety. Channel's direction has no equivalency because Elixir processes can always both send and receive with no way to restrict that._

---

## Cell 5: Actor Model in Go

> **Model being expressed:** Actor Model (Erlang-style)
> **Language used:** Go

### 1. What maps naturally

_With basic structure, goroutines map well to actors. Whereas a goroutine is spawned with go f(), runs independently, and stops when its code is done, Erlang spawns in the very similar way. Each goroutine can be given its own channel to act as a rough mailbox. Although using chan interface{} allows different types of messages be passed like in Erlang, type checking is lost when doing that._

### 2. What requires simulation or workaround

_Mailbox behavior must be built from scratch. In Erlang every process gets a mailbox automatically (FIFO, supporting pattern matching, and ability to skip messages, even though they are not ready.) Go channels are typed and strictly FIFO with no skipping. To fake selective receive, messages must be pulled off, checked, and non-matching ones put back, which gets slow as queue grows and breaks ordering._

### 3. What cannot be represented well or at all

_OTP's supervisor trees cannot be done naturally in Go. In Erlang (spawnlink) and monitor are built into the language - when a process dies its supervisor gets notified and restarts it. In Go a goroutine that panics just crashes the program unless recover() is placed in a deferred function, but that is local recovery not supervision from another goroutine. All the monitoring, restarting, and state setup must be written by hand. Erlang also has a process registry through register/2 and whereis/1 - Go has nothing like that, so extra code is needed just to let the system find a restarted goroutine._ 

---

## Cell 6: Shared Memory in Go

> **Model being expressed:** Shared Memory (Java-style)
> **Language used:** Go

### 1. What maps naturally

_Go fully supports shared memory. Goroutines run in the same address space so any global variable or struct field is directly accessible to all of them, same as Java threads sharing a heap. The slides show this with the ccc counter example where goroutines increment and decrement the same integer. sync.Mutex works like Java's synchronized, and sync.WaitGroup works like thread.join(). Out of all six cells this one fits the best._

### 2. What requires simulation or workaround

_Nothing needs to be simulated — Go has all the tools for shared memory. The issue is Go pushes programmers toward channels instead. Using sync.Mutex works fine but goes against how Go is meant to be written. The model works, but idiomatic Go style has to be abandoned to use it._

### 3. What cannot be represented well or at all

_There is nothing in the shared memory model that Go cannot do. It has sync.Mutex, sync.RWMutex, sync/atomic, and sync.Cond — all of it is there. This is the one cell where nothing is lost. The only downside is stylistic — Go programmers are not supposed to write this way and the Go race detector exists because shared memory is seen as risky even in Go. But technically everything is representable._

---
