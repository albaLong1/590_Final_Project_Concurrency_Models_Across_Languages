# Part 3: Synthesis
 
---
 
_Write 300-500 words here. Do not summarize the matrix — use it as evidence for a new higher-level claim._
 
_Consider addressing:_
- _What does the difficulty of expressing each model in foreign languages reveal about the models themselves?_
- _Are some models more "portable" than others?_
- _Does portability correlate with how much a model relies on runtime support vs language-level constructs?_
- _Does portability correlate with a model's theory of where errors come from and how they should be handled?_
---
 
_The matrix reveals one clear pattern - the harder a model is to port, the more it depends on its runtime to enforce its guarantees, not just its syntax or libraries._
_Shared memory is the most portable model across all three languages. It makes minimal demands on the runtime - it only needs a shared address space and locking primitives. Go represents it fully with sync.Mutex. Java has had it natively for decades. Even Elixir can partially fake it with Agent and ETS. The model does not care what language it runs in as long as threads and locks exist._
_CSP sits in the middle. The basic channel mechanic - send, receive, block - can be approximated in both Java and Elixir. BlockingQueue in Java gets close. The call/reply pattern in Elixir gets close. But Go's select kept coming up in the matrix as the feature that cannot be cleanly faked anywhere. select requires the runtime to monitor multiple channels simultaneously and pick one fairly. Without it the simulation either wastes CPU polling or loses directionality entirely. CSP is portable in parts but not fully._
_The actor model is the least portable. Not because message passing is hard to fake - that can be built anywhere with queues. The problem is everything the BEAM provides on top of it. Selective receive, supervisor trees, process linking, named registries - all enforced by the BEAM runtime, not by library code. In Java OTP supervision has to be written entirely by hand. In Go the same problem exists - goroutines can recover locally but nothing watches over them the way a supervisor process does. The "let it crash" philosophy only works because the BEAM is built around it from the ground up. Porting the actor model means porting its entire theory of failure, not just its message passing. That theory does not travel._
_The more a model relies on its runtime for correctness - not convenience, but actual correctness - the less portable it is. Shared memory relies on the programmer. CSP relies partly on the runtime. The actor model relies on the runtime almost entirely. That is exactly the order of portability the matrix shows._
 
---
 






























