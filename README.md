# 🖥️ Parallel Computer Architecture (PCA)
## UNIT 3 — Multi-threaded Architectures
### *Tera Teacher Ka Sabse Bada Episode — 10 Hours Ka Material! 🔥*

---

> **📌 Unit 3 ke baad tu:**
> - ILP vs TLP ka fark bol sakta hai
> - Cache coherence protocols (MSI, MESI, MOESI, MOSI) explain kar sakta hai
> - Snoopy bus kaise kaam karta hai — janega
> - Memory consistency models — SC, TSO, etc. — samjhega
> - Intel, IBM, Sun ke real case studies likhega
> - Exam mein sabse zyada marks is unit se laayega! 💪

---

## 📚 UNIT 3 Syllabus Checklist

- [x] Parallel Computers (recap + TLP focus)
- [x] ILP vs TLP
- [x] Cache Hierarchy & Communication Latency
- [x] Shared Memory Multiprocessors
- [x] Cache Coherence Problem
- [x] Synchronization Primitives — Atomic, TTS, Ticket, Array Locks
- [x] Barriers — Central and Tree
- [x] Performance Implications in Shared Memory Programs
- [x] Chip Multiprocessors (CMP) — Why CMP?
- [x] Shared L2 vs Tiled CMP
- [x] Core Complexity, Power/Performance
- [x] Snoopy Coherence — Invalidate vs Update
- [x] MSI, MESI, MOESI, MOSI Protocols
- [x] Pipelined Snoopy Bus Design
- [x] Memory Consistency Models — SC, PC, TSO, PSO, WO/WC, RC
- [x] Case Studies — Intel Montecito, Dual-core, Pentium4, IBM Power4, Sun Niagara

---

# TOPIC 1: Parallel Computers & ILP vs TLP

## Parallelism Ke Types — Big Picture

```
PARALLELISM
    |
    |-- Bit-level Parallelism     (32-bit -> 64-bit processors)
    |
    |-- Instruction-level (ILP)   (Within a single thread)
    |
    |-- Thread-level (TLP)        (Multiple threads simultaneously)
    |
    `-- Data-level (DLP)          (SIMD, Vector processors)
```

---

## ILP — Instruction Level Parallelism

**Kya hai:** Ek thread ke andar hi kai instructions simultaneously execute karna

> **Analogy:** Assembly line — ek car banate waqt, alag-alag workers alag-alag stages pe kaam karte hain ek saath (pipelining). Ek hi car/thread, lekin kaam parallel!

**Kaise achieve hota hai:**
- **Pipelining** — Instruction ke stages overlap karo
- **Superscalar** — Multiple execution units
- **Out-of-Order Execution** — Ready instructions pehle execute karo
- **Branch Prediction** — Guess karo aur aage badho
- **Speculative Execution** — Result assume karke kaam shuru karo

**Limitation:** ILP wall — ek thread mein kitna bhi parallel dhundho, limit aa jaati hai (4-6 IPC practical limit)

---

## TLP — Thread Level Parallelism

**Kya hai:** Multiple threads simultaneously alag-alag cores pe execute karna

> **Analogy:** Multiple assembly lines — 4 alag factories mein 4 alag cars ek saath ban rahi hain. Truly parallel!

**Kaise achieve hota hai:**
- **Multi-core processors** — Alag physical cores
- **Simultaneous Multi-Threading (SMT/HyperThreading)** — Ek core pe multiple threads
- **Multi-threading** — OS level thread switching

---

## ILP vs TLP — Comparison Table

| Feature | ILP | TLP |
|---------|-----|-----|
| Unit | Instructions within 1 thread | Multiple threads |
| Hardware | Superscalar, Out-of-Order | Multi-core, SMT |
| Granularity | Fine (instruction level) | Coarse (thread level) |
| Programmer effort | Less (compiler/hardware does it) | More (need parallel code) |
| Limit | ILP wall (~4-6 instructions) | Memory/bandwidth wall |
| Example | Intel Core pipeline | Intel Core i7 quad-core |

## Trick

> **"ILP = Ek Banda, Kai Haath"** (ek thread, kai instructions parallel)
> **"TLP = Kai Bande, Ek Kaam Mil Ke"** (kai threads, ek problem solve)

---

## EXAM TEMPLATE — ILP vs TLP

```
[Definition of both]
ILP (Instruction Level Parallelism): Executing multiple instructions 
from a single thread simultaneously using pipelining, superscalar, 
and out-of-order execution.

TLP (Thread Level Parallelism): Executing multiple threads simultaneously 
on different processing cores or hardware threads.

[Comparison Table]
Feature    | ILP                  | TLP
-----------|----------------------|---------------------------
Scope      | Within one thread    | Across multiple threads
Hardware   | Superscalar, OOO     | Multi-core, SMT
Granularity| Fine (instruction)   | Coarse (thread)
Limit      | ~4-6 IPC wall        | Memory bandwidth wall
Example    | Single core pipeline | Multi-core processor

[Why TLP is needed]
ILP has diminishing returns due to data dependencies and limited 
parallelism within a single thread. TLP overcomes these by using 
multiple threads, each with its own independent instruction stream.

[Conclusion]
Modern processors exploit both ILP (within each core) and TLP (across 
multiple cores) to maximize performance.
```

---

# TOPIC 2: Cache Hierarchy & Communication Latency

## Latency Chart

```
Level         | Access Time  | Size     
Registers     | < 1 ns       | ~100 B   
L1 Cache      | ~1-4 ns      | 32-64 KB 
L2 Cache      | ~5-12 ns     | 256KB-1MB
L3 Cache      | ~20-40 ns    | 4-32 MB  
Main RAM      | ~60-100 ns   | GBs      
Remote Memory | ~100-1000 ns | ---      
Disk (SSD)    | ~100,000 ns  | TBs      
```

> **Analogy of Latency:**
> - L1 Cache = Apni pocket se cheez nikalna (1 second)
> - RAM = Doosre room se laana (1 minute)
> - Disk = Doosre sheher se mangana (1 day)
> - Remote Memory = Doosre desh se mangana (1 week)

**Key insight:** Parallel system mein — Processor A ne data update kiya, Processor B ko woh data chahiye — network/bus se laana padega → **communication latency** = performance killer!

---

# TOPIC 3: Shared Memory Multiprocessors

## Architecture Overview

```
+------+  +------+  +------+  +------+
|  P1  |  |  P2  |  |  P3  |  |  P4  |   <- Processors
|  $1  |  |  $2  |  |  $3  |  |  $4  |   <- Private Caches
+--+---+  +--+---+  +--+---+  +--+---+
   `----------+----------+-----------'
                    |
           +--------v--------+
           |  Shared Memory  |   <- One memory for all
           |   (RAM/DRAM)    |
           +-----------------+
```

**UMA:** Sab processors ko memory tak equal time lagta hai (Symmetric Multiprocessor / SMP)

**NUMA:** Kuch memory paas mein (local — fast), kuch door (remote — slow)

---

# TOPIC 4: Cache Coherence Problem (MOST IMPORTANT!)

## Problem Kya Hai?

> Multiple processors, apne-apne caches mein ek hi memory location ki alag-alag copies rakhte hain. Ek ne update kiya → baaki outdated ho gaye → STALE DATA!

> **Analogy:** 3 log ek shared Google Doc par kaam kar rahe hain — offline mode mein:
> - Person A ka copy: Pizza = Rs.200
> - Person B ka copy: Pizza = Rs.200
> - Person A update karta hai: Pizza = Rs.250
> - Person B abhi bhi purana Rs.200 dekh raha hai → Cache Coherence Problem!

## Formal Example:

```
Initially: Memory[X] = 100

Step 1: CPU1 reads X  -> CPU1 cache: X=100  OK
Step 2: CPU2 reads X  -> CPU2 cache: X=100  OK
Step 3: CPU1 writes X=200 -> CPU1 cache: X=200  OK
                             Memory: X=200  OK
                             CPU2 cache: X=100  STALE! WRONG!

Step 4: CPU2 reads X  -> Gets 100 (wrong!)
```

## Two Solutions:

**Write-Invalidate:**
> Koi likhne wala ho → sab doosron ki copies INVALID kar do!

```
CPU1 writes X=200:
-> Send "INVALIDATE X" to all caches
-> CPU2 cache: X = INVALID
-> CPU2 reads X -> cache miss -> gets fresh value 200 from memory
```

**Write-Update:**
> Koi likhne wala ho → sab doosron ki copies UPDATE kar do!

```
CPU1 writes X=200:
-> Send "UPDATE X=200" to all caches
-> CPU2 cache: X = 200  (updated immediately)
```

## Invalidate vs Update:

| Feature | Invalidate | Update |
|---------|------------|--------|
| Action on write | Invalidate others | Update others |
| Bus traffic | Less (just a signal) | More (full data) |
| Next read after write | Cache miss | Cache hit |
| Used in practice | More common | Less common |

> **Trick:** "Invalid = Announce Purana Hai, Hatao!" vs "Update = Announce Naya Hua, Aao Lo!"

---

## EXAM TEMPLATE — Cache Coherence

```
[Definition]
Cache coherence is the problem that arises in shared memory 
multiprocessors when multiple processors cache the same memory location 
and one processor modifies its copy, making other copies stale/invalid.

[Example]
If CPU1 and CPU2 both cache X=100, and CPU1 writes X=200, 
CPU2 still sees X=100 — cache coherence violation.

[Two Solutions]
1. Write-Invalidate: Writer sends invalidation to all other caches.
   Others fetch updated value on next access. Less bus traffic.

2. Write-Update: Writer broadcasts new value to all other caches.
   Immediate update but higher bus traffic.

[Conclusion]
Hardware protocols (like MESI) automatically maintain coherence 
without programmer intervention.
```

---

# TOPIC 5: Snoopy Cache Coherence Protocols (MOST IMPORTANT!)

## Snoopy Protocol Kya Hai?

> **Snooping** = Sab caches bus pe "nazar rakhte hain" — jab koi write kare, sab sun lete hain aur apni copies update karte hain

> **Analogy:** Colony mein ek loudspeaker — koi bhi announce karega toh sab sun lenge. "Ghar 5 pe paani aaya!" → Sab react kar lete hain!

## Cache States Meaning:

| State Letter | Matlab |
|-------------|--------|
| M — Modified | Cache mein hai, memory se alag (dirty) — sirf is cache pe |
| E — Exclusive | Cache mein hai, memory same hai — sirf is cache pe |
| S — Shared | Cache mein hai, doosre caches mein bhi hoga, memory valid |
| I — Invalid | Cache mein nahi / outdated |
| O — Owned | Modified hai, cache-to-cache transfer ki zimmedaari |

---

## Protocol 1: MSI (3 States — Simple)

**States:** Modified (M), Shared (S), Invalid (I)

**Why MSI has a problem:**
> Sirf ek processor ne data read kiya (no one else has it), par woh S (Shared) state mein rehta tha. Agar write karna chahta hai → Unnecessary invalidation broadcast!

**State Transitions Table:**

| Current State | Event | Action | Next State |
|--------------|-------|--------|------------|
| Invalid | Processor Read | Fetch from memory | Shared |
| Invalid | Processor Write | Fetch + Invalidate others | Modified |
| Shared | Processor Read | Cache hit | Shared |
| Shared | Processor Write | Invalidate others | Modified |
| Shared | Other writes | Receive invalidation | Invalid |
| Modified | Processor Read/Write | Cache hit | Modified |
| Modified | Other reads | Write back to memory | Shared |
| Modified | Other writes | Write back, invalidated | Invalid |

**MSI Walkthrough Example:**
```
Initially: All caches INVALID, Memory[X] = 10

1. CPU1 reads X:   CPU1: I -> S, gets X=10
2. CPU2 reads X:   CPU2: I -> S, gets X=10, CPU1 stays S
3. CPU1 writes 20: CPU1: S -> M, broadcast INVALIDATE
                   CPU2: S -> I (receives invalidation)
4. CPU2 reads X:   CPU2: I -> S (cache miss! CPU1 writes back)
                   CPU1: M -> S (write back X=20 to memory)
                   Both now see X=20
```

---

## Protocol 2: MESI (4 States — Most Important!)

**States:** Modified (M), Exclusive (E), Shared (S), Invalid (I)

**MESI adds E (Exclusive) — Why?**

> MSI mein agar sirf ek processor ne data read kiya, phir bhi S state mein aata tha. Write karne pe → broadcast zaruri tha (wasteful!)
>
> MESI Solution: Exclusive state — "Sirf mujhpe hai, memory bhi valid hai" → Write karo toh directly M, NO broadcast needed!

**E State Meaning:**
- Sirf is cache ke paas ye data hai
- Memory bhi valid (clean copy)
- Write karo → M state (no bus broadcast!)

**MESI Benefit over MSI:**
```
With MSI:
  CPU1 reads X alone -> S state
  CPU1 writes X      -> Must broadcast INVALIDATE (wasteful! no one else has it)

With MESI:
  CPU1 reads X alone -> E state (Exclusive!)
  CPU1 writes X      -> E -> M (NO broadcast needed!)
  BUS TRAFFIC SAVED!
```

**MESI State Transitions:**

| Current | Event | Next State | Action |
|---------|-------|-----------|--------|
| I | Read (only reader) | E | Fetch from memory |
| I | Read (others also have) | S | Fetch from memory |
| I | Write | M | Fetch + Invalidate others |
| E | Processor Read | E | Cache hit |
| E | Processor Write | M | No broadcast! |
| E | Another processor reads | S | Transition to shared |
| S | Processor Read | S | Cache hit |
| S | Processor Write | M | Broadcast invalidate others |
| S | Another processor writes | I | Receive invalidation |
| M | Processor Read/Write | M | Cache hit |
| M | Another processor reads | S | Write back to memory first |
| M | Another processor writes | I | Write back, then invalidated |

**MESI State Diagram (Draw in exam!):**
```
        Processor Read (only one)
              |
              v
        +---------+  Processor Write  +-----------+
        | INVALID |<------------------| MODIFIED  |
        +---------+  (writeback+inv)  +-----------+
             |   ^                         ^  |
    Proc Read|   |Other write (inv)        |  | Proc Write
  (shared)   |   |                         |  | (broadcast inv)
             v   |           No broadcast! |  |
        +---------+  Proc Write   +-----------+
        | SHARED  |-------------->| EXCLUSIVE |
        +---------+               +-----------+
             |                         |
             `------- Another reads ---'
                      (both go S)
```

**Trick:** "Mera Exclusive Secret Invalid hua" = M, E, S, I

---

## Protocol 3: MOESI (5 States)

**States:** Modified (M), Owned (O), Exclusive (E), Shared (S), Invalid (I)

**MOESI adds O (Owned) — Why?**

> MESI problem: CPU1 has M state. CPU2 reads → CPU1 writes BACK to memory (slow!) → CPU2 reads from memory. Two slow operations!
>
> MOESI Solution: O (Owned) state — CPU1 directly sends data to CPU2 WITHOUT writing to memory first! Memory update baad mein!

**O State Meaning:**
- Cache has modified data
- This cache is RESPONSIBLE for supplying data to others
- Memory is stale (hasn't been updated yet)
- Multiple caches can be in S, but one "owns" the dirty copy

```
MESI (without Owned):
  CPU1 (M) -> CPU2 reads -> CPU1 writes back to memory -> CPU2 reads memory
  TWO bus transactions!

MOESI (with Owned):
  CPU1 (M) -> CPU2 reads -> CPU1 directly gives data to CPU2 (O state)
  CPU1: M -> O, CPU2: I -> S
  ONE bus transaction! Memory updated lazily later.
```

**MOESI States Summary:**

| State | Only Copy? | Memory Valid? | Dirty? |
|-------|-----------|--------------|--------|
| M | Yes | No | Yes |
| O | No (others in S) | No | Yes (owns dirty copy) |
| E | Yes | Yes | No |
| S | No | Yes | No |
| I | — | — | — |

---

## Protocol 4: MOSI (4 States)

**States:** Modified (M), Owned (O), Shared (S), Invalid (I)

> MOSI = MOESI without E (Exclusive) state
> Has cache-to-cache transfer (O) but no exclusive optimization (E)
> Less efficient than MESI for exclusive reads

---

## Protocol Comparison Table

| Protocol | States | Key Feature | Efficiency |
|---------|--------|-------------|------------|
| MSI | 3 | Basic, always broadcast on write | Low |
| MESI | 4 | E state saves broadcast for solo reader | Medium-High |
| MOESI | 5 | O state enables cache-to-cache transfer | High |
| MOSI | 4 | O state, no E | Medium |

**Trick:** "More States = Generally More Efficient"
- MSI (3) < MESI (4) = MOSI (4) < MOESI (5)
- Sab mein common: M, S, I
- MESI adds: E (save broadcast)
- MOESI adds: O (save writeback)

---

## EXAM TEMPLATE — MESI Protocol

```
[Introduction]
MESI is a cache coherence protocol maintaining coherence in shared 
memory multiprocessors using four states: Modified, Exclusive, 
Shared, and Invalid.

[Four States]
1. Modified (M): Present only in this cache, modified (dirty).
   Memory copy is stale. Must write back before eviction or sharing.

2. Exclusive (E): Present only in this cache, matches memory (clean).
   Write: transitions to M without bus broadcast (key advantage!)

3. Shared (S): May be present in multiple caches. Memory is valid.

4. Invalid (I): Cache line is not valid in this cache.

[Key Advantage over MSI]
MESI adds Exclusive state. When only one processor has a line (E state),
it can write without broadcasting invalidation, saving bus bandwidth.

[State Transitions on Key Events]
- Read miss, only reader: I -> E
- Read miss, others also have: I -> S
- Write in E: E -> M (no bus transaction needed!)
- Write in S: S -> M (broadcast invalidate)
- Another reads M: M -> S (write back to memory first)

[Conclusion]
MESI efficiently maintains cache coherence with reduced bus traffic 
by leveraging the Exclusive state.
```

---

# TOPIC 6: Pipelined Snoopy Bus Design

## Snoopy Bus Kya Hai?

> Shared bus jisme sab processors "snoop" karte hain (dekhte rehte hain). Har processor bus pe har transaction dekh sakta hai aur react kar sakta hai.

> **Analogy:** Colony mein shared loudspeaker — sab sun lete hain, sab react karte hain!

## Without Pipelining (Problem):

```
Cycle 1-5:  Txn1 [Address][Data][Response]
Cycle 6-10:                                 Txn2 [Address][Data][Response]
SLOW — Ek transaction khatam hone ka wait!
```

## With Pipelining (Better!):

```
Cycle 1: Txn1 [Address Phase]
Cycle 2: Txn1 [Data Phase]    | Txn2 [Address Phase]
Cycle 3: Txn1 [Response]      | Txn2 [Data Phase]    | Txn3 [Address Phase]
Multiple transactions overlap = FASTER!
```

## 4 Pipeline Stages of Snoopy Bus:

**Stage 1 — Address Phase:**
- Processor bus pe address dalti hai
- Sab caches is address ko "snoop" karte hain
- Check: Kya mere paas ye data hai? (Hit/Miss)

**Stage 2 — Arbitration Phase:**
- Agar kai processors ek saath bus use karna chahte hain → arbitration
- Ek ko priority milti hai

**Stage 3 — Data Phase:**
- Data transfer (memory se processor, ya processor-to-processor)
- Cache-to-cache transfer bhi possible

**Stage 4 — Response/Acknowledgment:**
- Transaction complete — all caches update state accordingly

## Snoopy Bus Limitations:

- Bus bandwidth bottleneck — sab share karte hain ek hi bus
- Scalability limited — 32-64 processors ke baad bus saturate ho jaata hai
- Large scale ke liye: Directory-based protocols use hote hain

> **Trick:** "Snoopy = Gossip Network — Sab sunta hai, sab jaanta hai, sab react karta hai!"

---

## EXAM TEMPLATE — Pipelined Snoopy Bus

```
[Definition]
A snoopy bus is a shared bus where all processors monitor (snoop) 
all bus transactions to maintain cache coherence. Pipelining overlaps 
multiple bus transactions for efficiency.

[How Snooping Works]
When a processor performs a read or write, it broadcasts the request 
on the shared bus. All other processors' caches snoop this transaction 
and update their state (invalidate or update).

[Pipeline Stages]
1. Address Phase: Processor places address; others snoop.
2. Arbitration: Multiple requests resolved; one gets bus.
3. Data Phase: Data transferred between memory/caches.
4. Acknowledgment: Transaction completed; cache states updated.

[Advantage of Pipelining]
Next transaction's address phase overlaps with current transaction's 
data phase, increasing bus utilization and throughput.

[Limitations]
Bus becomes bottleneck as processor count grows. Scalability limited 
to ~32-64 processors. Large systems use directory protocols instead.
```

---

# TOPIC 7: Synchronization Primitives (Advanced)

## Atomic Operations

> **Atomic** = Operation jo interrupt nahi ho sakta — ek hi "step" mein complete hoti hai

**Common Atomic Operations:**

```
Test-and-Set (TAS):   Read value, set to 1 atomically
                      Returns 0 -> Lock acquired; Returns 1 -> Locked, wait

Compare-and-Swap:     If memory[addr] == expected, set = new_value
                      Used in lock-free data structures

Fetch-and-Add:        old = *addr; *addr += value; return old;
                      Used in ticket locks
```

---

## Lock Type 1: Simple TAS (Bad for Multiprocessor!)

```c
// Every spin iteration does a write (TAS) -> floods bus with invalidations!
while (test_and_set(&lock) == 1) {
    // Keep spinning — NOISY, generates heavy bus traffic!
}
```

**Problem:** Every spinning thread does a write → bus ka bahot traffic!

---

## Lock Type 2: TTS — Test and Test-and-Set (Better!)

> Pehle sirf READ karo (test), phir agar free lage tab TAS (test-and-set) karo!

```c
// TTS Lock:
while (true) {
    while (lock == 1) {
        // Just READ — uses local cache, no bus write!
    }
    // Lock might be free — now try TAS:
    if (test_and_set(&lock) == 0) {
        break;  // Got the lock!
    }
    // Failed? Go back to reading.
}
```

**Why TTS > TAS:**
- TAS: Har iteration mein write → bus mein invalidation flood!
- TTS: Read karo locally (cache hit), sirf ek baar TAS karo
- Bus traffic bahut kam!

> **Analogy:**
> TAS = Traffic light pe baar baar poochhte rehna "Abhi green hua?" (bahut annoying!)
> TTS = Light dekho passively, jab green dikhne lage tab proceed karo!

---

## Lock Type 3: Ticket Lock (Fair!)

> Token number system — Queue mein apna number lo, apna number aane ka wait karo!

```c
struct ticket_lock {
    int next_ticket;   // Agla ticket number
    int now_serving;   // Abhi kaun serve ho raha hai
};

void lock(struct ticket_lock *L) {
    int my_ticket = fetch_and_add(&L->next_ticket, 1);  // Token lo
    while (L->now_serving != my_ticket) {
        // Apna number aane ka wait karo (spin locally)
    }
    // Lock acquired!
}

void unlock(struct ticket_lock *L) {
    L->now_serving++;   // Next person ka number announce karo
}
```

**Ticket Lock Advantage:**
- **Fair** — FIFO order guaranteed (pehle aaya, pehle serve!)
- TAS/TTS mein koi bhi jump kar sakta tha — unfair!
- Starvation nahi hoti

> **Analogy:** Hospital ka token system — Token lo, wait karo, apna number aane pe andar jao. Fair, orderly!

---

## Lock Type 4: Array-Based Lock (Best Scalability!)

> Har processor ka apna array slot — apne slot mein spin karo (no sharing!)

```c
int slots[NUM_PROCESSORS];   // Each processor has own slot

// Each processor spins on ITS OWN slot only!
// -> local cache, zero bus traffic during spinning!
```

**Array Lock Advantage:**
- Har processor apne private cache location pe spin karta hai
- No false sharing, no bus traffic during spinning
- Best scalability among spin locks

**Disadvantage:** O(N) memory needed

## Comparison of Lock Types:

| Lock | Fairness | Bus Traffic | Memory | Scalability |
|------|---------|------------|--------|-------------|
| Simple TAS | No | Very High | O(1) | Poor |
| TTS | No | Medium | O(1) | Better |
| Ticket Lock | Yes (FIFO) | Medium | O(1) | Good |
| Array Lock | Yes | Low (local) | O(N) | Best |

**Tricks:**
- TAS = Noisy neighbour (keeps knocking!)
- TTS = Polite — checks first, knocks only when needed
- Ticket = Hospital token — fair queue
- Array = Own doorbell — spin on yours only!

---

## Barriers — Central and Tree

### What is a Barrier?

> Sab threads yahan ruko, jab tak sab na pahunch jayein — phir saath aage!

> **Analogy:** Race mein sab runners ek checkpoint pe milenge, aur phir saath race start!

---

### Central Barrier (Simple, but Bad Scalability)

```
Global counter: Total = N threads
Each thread: counter-- when it arrives
Last thread (count reaches 0): Reset + Release all!
Others: Spin waiting for release signal
```

**Problem:** Ek hi counter pe sab spin karte hain → heavy bus traffic! Scales poorly.

---

### Tree Barrier (Better Scalability — O(log N))

> Like a tournament — paired processors sync karte hain, phir winners next round!

```
8 Processors:
Round 1: [P0-P1] [P2-P3] [P4-P5] [P6-P7]   (4 pairs sync)
Round 2: [P0--P2] [P4--P6]                   (2 pairs sync)
Round 3: [P0------P4]                         (1 pair -- DONE!)

WAKEUP (broadcast back down the tree)
```

**O(log N) rounds instead of O(N) waiting → Better scalability!**

## Central vs Tree Barrier:

| Feature | Central | Tree |
|---------|---------|------|
| Implementation | Simple | Complex |
| Rounds to complete | O(1) | O(log N) |
| Bus traffic | High | Low (distributed) |
| Scalability | Poor | Good |

---

## EXAM TEMPLATE — Synchronization Primitives

```
[Introduction]
Synchronization primitives coordinate access to shared resources 
in parallel systems.

[Atomic Operations]
Atomic operations execute indivisibly: Test-and-Set (TAS), 
Compare-and-Swap (CAS), Fetch-and-Add.

[Lock Types]
1. Test-and-Set (TAS): Atomic read+set. 
   Problem: High bus traffic (every spin does a write).

2. Test-and-Test-and-Set (TTS): First reads lock (no bus write), 
   then attempts TAS only when lock appears free.
   Advantage: Reduced bus traffic.

3. Ticket Lock: Fetch-and-Add assigns ordered ticket numbers.
   Wait for your ticket number. 
   Advantage: Fair FIFO ordering, prevents starvation.

4. Array Lock: Each processor spins on its own array slot.
   Advantage: No false sharing, minimal bus traffic, best scalability.

[Barriers]
Central Barrier: Global counter, last thread releases all.
Simple but causes high bus traffic.

Tree Barrier: O(log N) rounds, hierarchical sync.
Better scalability with reduced bus traffic.
```

---

# TOPIC 8: Chip Multiprocessors (CMP) — Why CMP?

## CMP Kya Hai?

> CMP = Chip Multi-Processor = Multiple processor cores on a SINGLE chip

## Why CMP? — Two Main Reasons

### Reason 1: Moore's Law

> Moore's Law: Chip pe transistor count har ~2 saal mein double hota hai

**Problem:** Itne zyada transistors ka kya karein?
- Pehle: Clock speed badhao → Power Wall hit ho gayi (heat!)
- Aaj: **More cores daalo!**

> Moore's Law ka result → Transistors hain → Use them as cores!

### Reason 2: Wire Delay Problem

> Signal ko chip ke ek corner se doosre corner tak jaane mein time lagta hai!

```
Old days: Wire delay < Gate delay -> Speed limited by transistor
Now:      Wire delay > Gate delay -> Speed limited by signal travel time!
```

**CMP Solution:**
- Multiple SIMPLE cores (each core smaller → shorter internal wires → fast!)
- Communication between cores through structured interconnect

> **Analogy:** Ek huge factory ka communication problem (wire delay) → Kai chhote factories ek area mein (CMP tiles)!

---

## Shared L2 vs Tiled CMP

### Shared L2 CMP

```
+------------------------------------------+
|            CMP Chip                      |
| +------+ +------+ +------+ +------+      |
| |Core 1| |Core 2| |Core 3| |Core 4|      |
| | L1$  | | L1$  | | L1$  | | L1$  |      |
| +--+---+ +--+---+ +--+---+ +--+---+      |
|    `----------+----------+-----------'   |
|           Shared L2 Cache (All cores)    |
+------------------------------------------+
```

**Advantages:** Simple coherence, good for shared data workloads
**Disadvantages:** L2 bandwidth saturates, wire delay to L2 grows, limited scale

---

### Tiled CMP

```
+------------------------------------------+
| +----------+    +----------+             |
| |  Tile 1  |----| Tile 2   |             |
| |Core + L2 |    |Core + L2 |             |
| +----------+    +----------+             |
|      |                |                  |
| +----------+    +----------+             |
| |  Tile 3  |----| Tile 4   |             |
| |Core + L2 |    |Core + L2 |             |
| +----------+    +----------+             |
|  Each tile has its own L2 slice          |
+------------------------------------------+
```

**Advantages:** Better scalability, local L2 reduces wire delay, higher total cache bandwidth
**Disadvantages:** Complex coherence protocol, non-uniform cache access

## Shared L2 vs Tiled CMP Comparison:

| Feature | Shared L2 | Tiled CMP |
|---------|-----------|-----------|
| Cache | One shared L2 | Each core has own L2 |
| Coherence | Simple | Complex |
| Scalability | Limited | Better |
| Bandwidth | Limited | Higher (distributed) |
| Example | Intel Core 2 Duo | Many-core tiles |

---

## Core Complexity vs Power/Performance

### The Power Problem

```
Power is proportional to C x V^2 x f
(C = capacitance, V = voltage, f = frequency)
```

**Problem:** Clock speed badhao → Voltage badhana padta → Power exponentially bada!

### Performance per Watt:

```
Simple Core x 4  vs  Complex Core x 1

Simple Core x 4:
- 4 simple cores at 2GHz, 1W each = 4W total
- Can handle 4 independent threads simultaneously

Complex Core x 1:
- 1 complex core at 4GHz, 10W
- Better single-thread performance
- But wastes power on speculation/OOO logic
```

**Conclusion:** Many simple cores often beat one complex core in throughput and power efficiency. But complex core wins in single-thread performance!

| Feature | Simple Core | Complex Core |
|---------|-------------|-------------|
| Power | Low | High |
| Single-thread perf | Lower | Higher |
| Throughput (parallel) | Higher (more cores) | Lower (fewer cores) |
| Example | ARM Cortex-A5 | Intel Core i9 |

---

# TOPIC 9: Memory Consistency Models (VERY IMPORTANT!)

## Problem Kya Hai?

> Cache coherence = ek variable ka value sab ke liye same ho.
> Memory Consistency = Multiple variables ke writes aur reads ka ORDER kya hona chahiye?

> **Analogy:** Do log ek saath diary likhte hain — Cache coherence ensures ek entry dono mein same hogi. Memory Consistency bolta hai: kaunsi entry pehle likhi, uski guarantee?

---

## Model 1: Sequential Consistency (SC) — Strictest

> "Har processor apni instructions ke order mein kaam kare, aur sab processors ek hi global order dekhen"

**Rules:**
1. Ek processor ke andar: Instruction order maintain karo
2. All processors: Ek hi global order of operations dikhe

```
P1: Write A=1, then Write B=1
P2: Reads B, then Reads A

SC Guarantees: If P2 reads B=1, then P2 MUST read A=1
(P1 wrote A before B, so if B is visible, A must also be visible)
```

**Advantage:** Easy to reason about
**Disadvantage:** Very slow — hardware must ensure strict ordering

> **Analogy:** Single-lane road — ek ke baad ek, strict order, no overtaking!

---

## Model 2: Processor Consistency (PC)

> SC se thoda relaxed — ek processor ke writes sab ko same order mein dikhenge, lekin different processors ke writes ka global order guaranteed nahi

---

## Model 3: TSO — Total Store Order

> Intel x86 processors yahi use karte hain!

**Key Relaxation:** Writes go into a write buffer (delay allowed). A processor can read while its write is still in buffer!

```
P1: Write X=1
P1: Read Y   <- Can read Y before X's write is visible to others!
(X=1 is in write buffer, not yet globally visible)
```

**Why TSO?** Write buffer allows processor to queue writes and continue. Performance gain!

---

## Model 4: PSO — Partial Store Order

> TSO se aur zyada relaxed — Writes to DIFFERENT addresses can also be reordered!

```
P1: Write X=1   <- These two writes can be reordered
P1: Write Y=1   <- (going to different addresses)
```

---

## Model 5: WO/WC — Weak Ordering / Weak Consistency

> Memory accesses between synchronization operations can be reordered freely!

```
SYNC_ACQUIRE
  // All operations here are strongly ordered
  critical_section_work
SYNC_RELEASE

// Between SYNCs: hardware can reorder anything freely
```

**Programmer responsibility:** Use sync operations when ordering matters!

---

## Model 6: RC — Release Consistency

> WO se refined — Acquire (for reads) and Release (for writes) are separate!

```
lock(mutex)     <- ACQUIRE — ensures reads/writes before this are complete
  Critical Section
unlock(mutex)   <- RELEASE — ensures reads/writes in section visible before release
```

---

## Memory Models Summary Table

```
Model     | Store-Load | Store-Store | Load-Load | Load-Store
----------|------------|-------------|-----------|------------
SC        | No relax   | No relax    | No relax  | No relax   <- Strictest
PC        | YES relax  | No          | No        | No
TSO       | YES relax  | No          | No        | No
PSO       | YES relax  | YES relax   | No        | No
WO/WC     | YES        | YES         | YES       | YES
RC        | YES        | YES         | YES       | YES        <- Most relaxed

(YES = This reordering IS ALLOWED)
```

**Relaxation Pyramid:**
```
SC (Strictest -- correct but slow)
    |
    v
TSO (x86 uses this)
    |
    v
PSO
    |
    v
WO/WC
    |
    v
RC (Most relaxed -- fastest but hardest to program)
```

**Trick:** "SC is Slow but Safe; TSO is Typical (x86); WO is Wild; RC is Release-Controlled!"
Or: "SC -> TSO -> PSO -> WO -> RC = More relaxed, Faster but harder!"

---

## EXAM TEMPLATE — Memory Consistency Models

```
[Introduction]
Memory consistency models define the rules for the order in which 
memory operations (reads and writes) are observed by different processors 
in a shared memory system.

[Models]
1. Sequential Consistency (SC): Strictest. All processors see the same 
   global order. Operations execute in program order. Easy to reason 
   about but slowest.

2. Total Store Order (TSO): Relaxes store-to-load ordering. Stores go 
   into a write buffer. Used by Intel x86 architecture.

3. Partial Store Order (PSO): Further relaxes by allowing stores to 
   different addresses to be reordered.

4. Weak Ordering (WO): Memory operations between sync points can 
   be reordered freely. Strong ordering only at sync boundaries.

5. Release Consistency (RC): Refines WO with separate Acquire and 
   Release synchronization operations.

[Relaxation Order]
SC (strictest) -> TSO -> PSO -> WO/WC -> RC (most relaxed)
More relaxed = Better performance but harder to program correctly.

[Programmer's Role]
Under relaxed models, programmers must use explicit synchronization 
(locks, barriers) to enforce required ordering.

[Conclusion]
The memory consistency model is a hardware design decision balancing 
correctness, programmability, and performance.
```

---

# TOPIC 10: Case Studies (Exam Mein Zaroor Aata Hai!)

## Case Study 1: Intel Montecito (Itanium 2)

**Type:** Dual-core Itanium processor (EPIC architecture)

```
Architecture:  EPIC (Explicitly Parallel Instruction Computing)
Cores:         2 cores on one chip
Threads/Core:  2 hardware threads (SMT) = 4 total
Cache:
  L1: 16KB I-cache + 16KB D-cache per core
  L2: 256KB per core
  L3: 24MB SHARED (huge! for server workloads)
Technology:    90nm
Transistors:   ~1.72 billion
Target:        Server / HPC workloads (databases, enterprise)
```

**Key Points:**
- Early example of multi-core + SMT combination
- Huge shared L3 cache for inter-core data sharing
- EPIC = Compiler does parallel scheduling (not hardware OOO)
- Showed that large shared L3 + multiple cores + SMT = server performance

---

## Case Study 2: Intel Core 2 Duo (Mainstream Dual-Core)

**Type:** First mainstream dual-core desktop/laptop processor

```
Architecture:  Modified P6 (Core microarchitecture)
Cores:         2 cores on one die
Cache:
  L1: 32KB per core (private)
  L2: 2MB or 4MB SHARED between both cores
Coherence:     Snoopy bus protocol (MESI)
Technology:    65nm then 45nm
Target:        Desktop, laptop mainstream users
```

```
+----------------------------------+
|         Core 2 Duo               |
|  +--------+     +--------+       |
|  | Core 0 |     | Core 1 |       |
|  | L1 I$  |     | L1 I$  |       |
|  | L1 D$  |     | L1 D$  |       |
|  +---+----+     +----+---+       |
|      `---------------'           |
|       Shared L2 Cache (4MB)      |
+----------------------------------+
```

**Key Points:**
- Brought dual-core to mainstream consumers
- Shared L2 = easy data sharing, simple coherence
- Much better performance-per-watt than Pentium 4

---

## Case Study 3: Intel Pentium 4 (Prescott)

**Type:** Single-core with Hyper-Threading (SMT)

```
Architecture:   NetBurst (VERY deep pipeline -- 31 stages!)
Cores:          1 physical core
Threads:        2 logical (via Hyper-Threading)
Clock:          3.0 - 3.8 GHz (very high)
Cache:
  L1: 12K micro-ops trace cache
  L2: 1MB
Power:          Up to 115W (VERY hot!)
Technology:     90nm (Prescott)
```

**Why Pentium 4 is Important (as a lesson!):**

```
Problem 1 - Pipeline too deep:
  31 pipeline stages -> branch misprediction = 31 cycles wasted!
  High GHz but very low IPC -> "GHz is not everything!"

Problem 2 - Power Wall:
  115W -> Extreme heat -> Nicknamed "Presshot"!
  Intel had to abandon NetBurst architecture

Good Part - HyperThreading:
  1 Physical Core -> 2 Logical Cores (SMT)
  When one thread stalls -> other thread uses execution units
  ~20-30% performance improvement (not 2x)
```

**Key Lesson:** Deep pipelines + high clock does NOT equal high performance. Power wall kills performance scaling.

---

## Case Study 4: IBM Power4 (World's FIRST Dual-Core!)

**Type:** First true commercial dual-core processor (2001!)

```
Architecture:  IBM POWER ISA
Cores:         2 cores per chip (FIRST EVER! Before Intel!)
Cache:
  L1: 32KB I$ + 32KB D$ per core
  L2: 1.5MB on-chip (shared, 3-way associative)
  L3: 32MB off-chip (connected via high-speed bus)
Processor:     4-way OOO, 8-issue superscalar per core
Clock:         1.0 - 1.9 GHz
Transistors:   174 million, 180nm
Target:        IBM pSeries servers (enterprise)
```

```
+------------------------------------------+
|              IBM Power4 Chip             |
| +---------------+   +---------------+   |
| |    Core A     |   |    Core B     |   |
| | 8-issue OOO   |   | 8-issue OOO   |   |
| | 32KB L1 I$    |   | 32KB L1 I$    |   |
| | 32KB L1 D$    |   | 32KB L1 D$    |   |
| +------+--------+   +-------+-------+   |
|        `------------------'             |
|         1.5MB Shared L2 Cache           |
|         Memory Interface                |
+------------------------------------------+
                   |
          32MB Off-Chip L3 Cache
```

**Why IBM Power4 is Important:**
- **World's first commercial dual-core processor (2001)** — Before Intel by years!
- 8-issue superscalar — very high-performance per core
- L3 off-chip allowed massive 32MB capacity
- Established dual-core concept that Intel later adopted for mainstream

---

## Case Study 5: Sun UltraSPARC T1 — Niagara (Throughput King!)

**Type:** Massively multi-threaded processor — opposite philosophy of Power4!

```
Architecture:  SPARC (RISC, simple in-order)
Cores:         8 cores per chip
Threads/Core:  4 hardware threads (SMT)
Total Threads: 8 x 4 = 32 hardware threads!
Cache:
  L1: 16KB I$ + 8KB D$ per core
  L2: 3MB SHARED (all 8 cores share!)
Clock:         1.2 GHz (deliberately LOW!)
Power:         ~72W (low for 32 threads!)
Technology:    90nm
Target:        Web servers, throughput computing
```

**Niagara vs Pentium 4 — Two Opposite Philosophies:**

```
Feature          | Pentium 4      | Sun Niagara
-----------------|----------------|------------------
Cores            | 1              | 8
Threads total    | 2 (HT)         | 32!
Core complexity  | Complex OOO    | Simple In-order
Pipeline depth   | 31 stages      | Short
Branch predictor | Yes            | NO!
Out-of-order     | Yes            | NO!
Clock            | 3.8 GHz        | 1.2 GHz
Power            | 115W           | 72W
Best for         | Single thread  | Server throughput
```

**Why Niagara is Brilliant:**

```
Web server workload = Thousands of simultaneous requests
Each request = one thread

Niagara: 32 threads -> Handle 32 requests simultaneously!
When one thread stalls on memory (cache miss) -> ANOTHER thread runs IMMEDIATELY!
No wasted cycles! This is called "Latency Hiding through Multi-threading"

Result: Excellent throughput-per-watt for web servers
```

**The Key Insight:** In-order simple core + many threads = better for server workloads than complex OOO + few threads!

---

## Case Studies Master Comparison Table

| Feature | Pentium 4 | Core 2 Duo | Montecito | IBM Power4 | Sun Niagara |
|---------|-----------|------------|-----------|-----------|------------|
| Cores | 1 | 2 | 2 | 2 | 8 |
| Total Threads | 2 (HT) | 2 | 4 | 2 | 32 |
| Core Type | Complex OOO | Complex OOO | EPIC | Complex OOO | Simple In-order |
| Clock | 3.8 GHz | 3.0 GHz | 1.7 GHz | 1.9 GHz | 1.2 GHz |
| L2 Cache | 1MB | 4MB shared | 256KB/core | 1.5MB shared | 3MB shared |
| Power | 115W | ~65W | ~100W | ~160W | ~72W |
| Target | Desktop | Desktop/Laptop | Server | Server | Web Server |
| Key Feature | SMT (HT) | Mainstream dual | EPIC+SMT | First dual-core EVER | 32-thread throughput |

---

## EXAM TEMPLATE — Case Studies

```
[For ANY case study, use this template:]

[Name + Category]
[Processor Name] is a [dual-core/multi-threaded/etc.] processor designed 
for [desktop/server/web server] workloads by [Intel/IBM/Sun].

[Key Architecture Features]
- Cores: [X]
- Threads: [Y] (via SMT/HyperThreading if applicable)
- Cache: L1: [size], L2: [shared/private, size], L3: [if any]
- Clock: [X GHz]
- Core design: [OOO/In-order/EPIC]
- Power: [XW]
- Technology: [Xnm]

[What Makes It Unique]
[2-3 lines on the key innovation or lesson]

[Best For / Limitation]
Best for: [workload type]
Limitation: [what it lacks]
```

### Complete Example — Sun Niagara:

```
Sun UltraSPARC T1 (Niagara) is a massively multi-threaded processor 
designed for web server and throughput computing workloads.

Key Features:
- 8 simple in-order SPARC cores
- 4 hardware threads per core = 32 total threads
- L1: 16KB per core; L2: 3MB shared among all cores
- Clock: 1.2 GHz (deliberately low)
- Simple in-order pipeline (no branch predictor, no OOO)
- Power: ~72W for 32 threads

Key Innovation:
Niagara rejected the "faster single core" approach. With 32 threads, 
when one thread stalls on a cache miss, another thread immediately 
executes ("latency hiding"). This gives excellent throughput for 
web servers handling thousands of simultaneous requests.

Best for: Web servers, databases, network processing
Advantage: Highest thread-level parallelism, best throughput-per-watt
Limitation: Poor single-thread performance compared to complex cores
```

---

# MASTER SUMMARY TABLE — Unit 3

| Topic | Key Point | Exam Trick |
|-------|-----------|------------|
| ILP | Parallelism within one thread | Pipelining, superscalar |
| TLP | Parallelism across threads | Multi-core, SMT |
| Cache Coherence | Stale data in multiprocessors | Google Doc offline analogy |
| Invalidate | Write -> Invalidate others | Announce "purana hai, hatao!" |
| Update | Write -> Update all copies | Announce "naya hua, update karo!" |
| MSI | 3 states: M, S, I | Basic, always broadcasts |
| MESI | +E: Exclusive solo reader | E saves bus traffic on write |
| MOESI | +O: Owned for cache-to-cache | O avoids writeback to memory |
| TAS | Simple atomic lock | Noisy (floods bus) |
| TTS | Read first, then TAS | Polite — check then knock |
| Ticket Lock | FIFO token-based fair lock | Hospital token system |
| Array Lock | Private slot spinning | Own doorbell — no sharing |
| Central Barrier | All wait on global counter | Simple but poor scale |
| Tree Barrier | Hierarchical O(log N) | Tournament style |
| Why CMP | Moore's Law + Wire Delay | More transistors -> more cores |
| Shared L2 | All cores share L2 | Simple, limited scale |
| Tiled CMP | Each core has own L2 | Scalable, complex |
| SC | Strictest memory model | Single-lane road, slow |
| TSO | x86 uses this, write buffers | Intel's choice |
| WO/RC | Most relaxed, sync points | Programmer must add sync |
| Pentium 4 | 31-stage pipeline, 115W | Lesson: deep != fast |
| IBM Power4 | World's first dual-core! 2001 | Before Intel! |
| Montecito | Dual Itanium + SMT + 24MB L3 | Server EPIC beast |
| Sun Niagara | 8 cores x 4 threads = 32 | Throughput king! Latency hiding |

---

# EXAM-DAY TOP QUESTIONS FOR UNIT 3

1. **ILP vs TLP** — Definition + Table
2. **Cache Coherence Problem** — Example + Two solutions
3. **MESI Protocol** — States + Transitions + Diagram (MUST!)
4. **MOESI vs MESI** — What does O state add?
5. **TTS vs TAS** — Why TTS is better?
6. **Ticket Lock** — How it works? Why fair?
7. **SC vs TSO** — Memory consistency models
8. **Why CMP?** — Moore's Law + Wire Delay reasons
9. **Shared L2 vs Tiled CMP** — Compare
10. **Sun Niagara case study** — Most commonly asked!
11. **IBM Power4** — World's first dual-core
12. **Pipelined Snoopy Bus** — How it works

## Diagram Banana = Bonus Marks:
- MESI 4-state diagram with arrows
- Snoopy bus pipeline stages
- NUMA architecture
- Tree barrier structure
- Cache hierarchy with latency numbers

---

*Prepared with love by Your Teacher | PCA Unit 3 | All 10 Contact Hours Covered!*
*"ILP samjha, TLP jaana, MESI yaad kiya -- ab EXAM READY HAI TU!" 🎉*
*Teen units complete -- Ab revision karo aur exam mein dhamaka karo! 💪*
