---
layout: default
title: Session 4 – Memory Basics
parent: "01 – Digital Logic Fundamentals"
nav_order: 3
---

# Session 4 – Memory Basics

In the previous sessions, we built up combinational and sequential logic—circuits that compute and circuits that remember. Now we take a step back to see the bigger picture: how do we store vast amounts of data efficiently, access it quickly, and manage it in a way that makes sense for both hardware and software?

Memory is one of the fundamental pillars of computer architecture. Without it, our processors would be nothing more than expensive calculators that forget everything the moment you turn them off. But memory isn't just a single monolithic thing—it's a carefully orchestrated hierarchy of different technologies, each with its own trade‑offs between speed, size, and cost.

This session will walk you through the foundations of memory systems. We will start with the basic vocabulary—bits, bytes, words—and build up to understand why your computer has multiple levels of cache, what virtual memory does for you, and how all of these pieces fit together to create the illusion of a fast, large, and reliable memory system.

If this feels like a lot, don't worry. Take your time, work through the examples, and remember: every expert was once a beginner struggling with these same concepts.

By the end of this session, you should be comfortable with:

- the basic units (bit, byte, word) and how addressing works
- alignment constraints and why they matter
- endianness and how different systems store multi‑byte values
- the difference between SRAM, DRAM, and non‑volatile storage
- the memory hierarchy and the twin principles of locality
- cache organization: lines, sets, tags, and how addresses are decoded
- cache mapping schemes and replacement policies
- write policies and their performance implications
- a simple performance model (Average Memory Access Time)
- virtual memory, page tables, and the TLB
- how to model simple memories in Verilog

---

## 1. Data units and addressing: the foundation

Before we can talk about memory systems, we need a shared vocabulary. Let's start at the very bottom.

### 1.1 Bits, bytes, and words

- **Bit**: the smallest unit of information, representing a single binary digit: 0 or 1. Everything in a digital computer reduces to bits.

- **Byte**: a group of 8 bits. This is the fundamental unit of addressable storage in nearly all modern computers. One byte can represent 256 different values (from 0x00 to 0xFF in hexadecimal, or 0 to 255 in decimal).

- **Word**: the "natural" data size of a particular machine. On a 32‑bit processor, a word is typically 32 bits (4 bytes). On a 64‑bit machine, a word is 64 bits (8 bytes). You'll also hear:
  - **Half‑word**: half the size of a word (e.g., 16 bits on a 32‑bit machine)
  - **Double‑word**: twice the size (e.g., 64 bits on a 32‑bit machine, or 128 bits on a 64‑bit machine)

These terms are architecture‑dependent, so always check the documentation for the specific processor you're working with.

### 1.2 Address spaces and addressing modes

An **address** is simply a number that identifies a specific location in memory. Think of it like a house number on a street—every location has a unique identifier.

When we say an architecture has an **N‑bit address space**, we mean that addresses are N bits wide. This determines the maximum amount of memory the processor can directly address:

$$
\text{Maximum addressable memory} = 2^N \text{ bytes}
$$

For example:
- **16‑bit addresses**: $2^{16} = 65{,}536$ bytes = 64 KiB
- **32‑bit addresses**: $2^{32} = 4{,}294{,}967{,}296$ bytes = 4 GiB
- **64‑bit addresses**: $2^{64} \approx 18.4$ exabytes (far more than any current system actually has!)

In practice, most 64‑bit systems use only 48 bits of address space (256 TiB), which is still vastly more than you'll ever fill.

#### Byte addressing vs. word addressing

Most modern processors use **byte addressing**: each address points to a single byte, and accessing a word requires multiple consecutive byte addresses. Some older or specialized architectures used **word addressing**, where each address pointed to a whole word, and you couldn't directly address individual bytes. Byte addressing is more flexible and is now universal.

### 1.3 Example: address arithmetic

Suppose you have a 32‑bit machine with byte addressing. You allocate an array of 10 integers, each 4 bytes, starting at address 0x1000. Where does each element live?

```
Element    Address      Calculation
arr[0]     0x1000       base + 0*4
arr[1]     0x1004       base + 1*4
arr[2]     0x1008       base + 2*4
...
arr[9]     0x1024       base + 9*4
```

This is exactly what compilers do when you write `arr[i]` in C: they compute `base_address + i * sizeof(element)`.

---

## 2. Alignment: why it matters

### 2.1 What is alignment?

A data object is **aligned** if its address is a multiple of its size. For example:

- A 4‑byte integer is aligned if its address is divisible by 4 (e.g., 0x1000, 0x1004, 0x1008, …)
- An 8‑byte double is aligned if its address is divisible by 8 (e.g., 0x2000, 0x2008, 0x2010, …)
- A 1‑byte char is always aligned (any address is a multiple of 1)

If an object's address is *not* a multiple of its size, we say it is **misaligned**.

### 2.2 Why does alignment matter?

Many processors have restrictions on how they can access memory:

1. **Performance**: Even on processors that support misaligned access, it can be significantly slower. A misaligned 4‑byte load that spans two cache lines might require two separate memory transactions instead of one.

2. **Hardware simplicity**: Aligned accesses are easier to implement in hardware. The memory system can be designed assuming that a 4‑byte load will always access bytes within a single naturally aligned block.

3. **Atomicity**: On some architectures, only naturally aligned accesses are atomic. This matters for multithreaded programs where you want reads and writes to be indivisible operations.

4. **Architecture requirements**: Some architectures (like early MIPS and Alpha) simply **fault** on misaligned accesses—your program crashes with an alignment exception.

### 2.3 Example: aligned vs. misaligned

Consider accessing a 32‑bit integer at different addresses:

| Address  | Aligned? | Why                                   |
|----------|----------|---------------------------------------|
| 0x1000   | Yes      | 0x1000 ÷ 4 = 0x400 (exact multiple)  |
| 0x1004   | Yes      | 0x1004 ÷ 4 = 0x401 (exact multiple)  |
| 0x1002   | No       | 0x1002 ÷ 4 = 0x400.5 (not a multiple)|
| 0x1007   | No       | 0x1007 ÷ 4 = 0x401.75 (not a multiple)|

On an x86 processor, both aligned and misaligned accesses will work, but the misaligned ones will be slower. On a strict RISC architecture, the misaligned accesses might cause a trap that the operating system has to handle in software (very slow!) or might simply crash your program.

### 2.4 Practical advice

- Modern compilers automatically align data structures for you.
- Use `#pragma pack` or `__attribute__((packed))` with caution—packing structs to save space can introduce misalignment.
- When writing low‑level code (device drivers, embedded systems), always be conscious of alignment requirements.

---

## 3. Endianness: byte order in multi‑byte values

### 3.1 The problem

When you store a multi‑byte value like the 32‑bit integer `0x12345678` in memory, which byte goes where? There are two main conventions.

### 3.2 Little‑endian vs. big‑endian

- **Little‑endian**: the **least significant byte** (LSB) is stored at the lowest address.
  - Think "little end first"
  - Used by x86, x86‑64, ARM in little‑endian mode, RISC‑V

- **Big‑endian**: the **most significant byte** (MSB) is stored at the lowest address.
  - Think "big end first"
  - Used by network protocols (hence "network byte order"), some older architectures (68000, SPARC), ARM in big‑endian mode

### 3.3 Detailed example

Let's store `0x12345678` starting at address `0x1000`:

**Little‑endian layout:**

```
Address    Byte value
0x1000     0x78    (LSB)
0x1001     0x56
0x1002     0x34
0x1003     0x12    (MSB)
```

Reading from lowest to highest address, you see: 78 56 34 12

**Big‑endian layout:**

```
Address    Byte value
0x1000     0x12    (MSB)
0x1001     0x34
0x1002     0x56
0x1003     0x78    (LSB)
```

Reading from lowest to highest address, you see: 12 34 56 78

### 3.4 Why does this matter?

1. **File formats and network protocols**: Data sent over a network or stored in a file must have a defined byte order, or different machines will interpret it differently.

2. **Binary data interchange**: If you write a binary file on a little‑endian machine and read it on a big‑endian machine (or vice versa), you'll get garbage unless you explicitly handle the conversion.

3. **Debugging**: When you look at a memory dump in a debugger, you need to know the endianness to interpret multi‑byte values correctly.

### 3.5 Practical advice

- Use standard libraries for network byte order conversion: `htons()`, `htonl()`, `ntohs()`, `ntohl()` in C.
- Be aware of your target platform's endianness, especially when dealing with binary file I/O or hardware registers.
- Modern high‑level languages often hide this from you, but it's critical knowledge for systems programming.

---

## 4. Memory technologies: SRAM, DRAM, and beyond

Now that we understand how data is represented and addressed, let's talk about the physical hardware that stores it.

### 4.1 SRAM (Static RAM)

**Structure**: Each bit is stored in a small circuit made from flip‑flops (typically 6 transistors per bit in a classic 6T SRAM cell).

**Characteristics**:
- **Fast**: Access times in the nanosecond range or even sub‑nanosecond for on‑chip caches.
- **No refresh needed**: As long as power is on, the data remains stable (hence "static").
- **Expensive**: More transistors per bit means larger area and higher cost.
- **Low density**: You can't pack as many bits into a given chip area.

**Uses**: 
- On‑chip CPU caches (L1, L2, sometimes L3)
- Register files
- Small embedded memories in FPGAs and ASICs

Think of SRAM as the fast, expensive memory that sits close to the processor.

### 4.2 DRAM (Dynamic RAM)

**Structure**: Each bit is stored as a charge on a tiny capacitor, with one transistor acting as an access switch (a 1T‑1C cell).

**Characteristics**:
- **Slower than SRAM**: Access latencies are tens to hundreds of nanoseconds.
- **Needs refresh**: The capacitor charge leaks away, so DRAM must be periodically "refreshed" (rewritten) every few milliseconds.
- **Cheaper and denser**: Far fewer transistors per bit means you can fit gigabytes on a single chip.

**Uses**:
- Main system memory (RAM sticks in your computer)
- Graphics card memory (GDDR variants)

Think of DRAM as the large, slower, but more affordable memory that holds your programs and data while they're running.

### 4.3 Non‑volatile storage

**Volatile** means the memory loses its contents when power is removed. SRAM and DRAM are both volatile.

**Non‑volatile** storage retains data without power:

- **Flash memory (NAND Flash)**: Used in SSDs, USB drives, SD cards. Much slower than DRAM for random access, but persistent and fairly dense.
- **Hard disk drives (HDD)**: Mechanical spinning disks with magnetic storage. Even slower than SSDs, but cheap per gigabyte.
- **Emerging technologies**: ReRAM, MRAM, Phase-Change Memory (PCM)—faster than Flash, closer to DRAM speeds, but still being developed.

### 4.4 Comparison table

| Type     | Speed         | Density | Cost/bit | Volatile? | Typical use            |
|----------|---------------|---------|----------|-----------|------------------------|
| SRAM     | Fastest (~ns) | Low     | High     | Yes       | Caches, registers      |
| DRAM     | Fast (~10s ns)| Medium  | Medium   | Yes       | Main memory (RAM)      |
| Flash    | Slow (~μs)    | High    | Low      | No        | SSDs, USB drives       |
| HDD      | Very slow (ms)| Highest | Lowest   | No        | Bulk storage           |

The key insight: **there is no single "perfect" memory**. Fast memory is expensive and small; cheap memory is slow and large. The solution? Use a **hierarchy** of memories, which brings us to the next section.

---

## 5. The memory hierarchy and locality

### 5.1 Why a hierarchy?

Ideally, we'd like memory that is:
- As fast as SRAM (nanosecond access)
- As large as a hard drive (terabytes)
- As cheap as DRAM per bit

Unfortunately, physics and economics make this impossible. The solution is the **memory hierarchy**: we build a pyramid of memories, each level larger but slower than the one above.

```
        ┌──────────────┐
        │  Registers   │ ← Fastest, smallest (tens of bytes)
        ├──────────────┤
        │  L1 cache    │ ← Very fast (tens of KB)
        ├──────────────┤
        │  L2 cache    │ ← Fast (hundreds of KB)
        ├──────────────┤
        │  L3 cache    │ ← Moderate (MBs, shared)
        ├──────────────┤
        │  Main memory │ ← Large (GBs), slower
        │  (DRAM)      │
        ├──────────────┤
        │  SSD/HDD     │ ← Huge (TBs), much slower
        └──────────────┘
```

The idea: keep frequently accessed data in small, fast levels; use large, slow levels for everything else.

### 5.2 Principle of locality

Why does this hierarchy work? Because programs don't access memory randomly; they exhibit **locality**:

#### Temporal locality
If you access a memory location now, you're likely to access it again soon.

Example: loop counters, frequently called functions, stack frames.

```c
for (int i = 0; i < 1000; i++) {
    // The variable 'i' is read repeatedly → temporal locality
    sum += array[i];
}
```

#### Spatial locality
If you access a memory location, you're likely to soon access nearby locations.

Example: sequential array traversal, instruction fetch (programs execute sequentially most of the time).

```c
for (int i = 0; i < 1000; i++) {
    sum += array[i];   // Access array[0], array[1], array[2]... in order
}
```

By exploiting locality, caches can keep the "working set" of your program close to the CPU, dramatically reducing average access time.

### 5.3 The memory hierarchy in practice

Let's see how a load instruction flows through the hierarchy:

1. **CPU executes** `lw x1, 0(x2)` (load word from address in x2)
2. **L1 cache check**: Is the data in L1? 
   - **Hit**: Return data in ~2–4 cycles. Done.
   - **Miss**: Check L2.
3. **L2 cache check**: Is it in L2?
   - **Hit**: Return data in ~10–20 cycles. Fill L1. Done.
   - **Miss**: Check L3.
4. **L3 cache check**: Is it in L3?
   - **Hit**: Return data in ~30–50 cycles. Fill L2 and L1. Done.
   - **Miss**: Go to main memory.
5. **DRAM access**: Fetch data from main memory (~200 cycles). Fill L3, L2, L1. Done.

If the data isn't in DRAM either (a **page fault**), the OS must load it from disk—an operation that can take *millions* of cycles.

This hierarchy is invisible to your software (mostly), but profoundly affects performance. A program with good locality runs fast; a program that thrashes the cache runs slow.

---

## 6. Cache basics: how caches work

A cache is a small, fast memory that stores copies of frequently accessed data from a larger, slower memory. Let's dissect how caches operate.

### 6.1 Cache lines (blocks)

Caches don't store individual bytes; they store **cache lines** (also called blocks), which are contiguous chunks of memory, typically 64 bytes in modern processors.

Why lines and not bytes?
- **Spatial locality**: If you access one byte, you'll probably access nearby bytes soon, so fetch them all at once.
- **Efficiency**: Transferring a whole line amortizes the cost of the memory transaction.

### 6.2 Cache organization: sets, ways, lines

Think of a cache as a table with rows and columns:

- **Line / Block**: The data storage unit (e.g., 64 bytes)
- **Set**: A row in the table (identified by the **index** bits of the address)
- **Way**: A column in the table (how many lines per set)

A cache with $S$ sets and $W$ ways has $S \times W$ total lines.

### 6.3 Address breakdown

When the CPU generates an address, the cache hardware splits it into three fields:

```
┌────────────┬──────────────┬────────────────┐
│    Tag     │    Index     │     Offset     │
└────────────┴──────────────┴────────────────┘
 High bits    Middle bits     Low bits
```

- **Offset** ($b$ bits): Which byte within the cache line? For a 64‑byte line, $b = \log_2(64) = 6$ bits.
- **Index** ($s$ bits): Which set to look in? If there are $S = 2^s$ sets, we need $s$ bits.
- **Tag** (remaining bits): Which memory block is stored here? The tag disambiguates different addresses that map to the same set.

### 6.4 Cache lookup: step by step

1. **Extract the index** from the address and select the corresponding set.
2. **Check all ways** in that set:
   - For each way, compare the stored tag with the address tag.
   - Check the **valid bit** (is this line actually holding data?).
3. If tag matches and valid bit is set: **Cache hit**. Use the offset to select the byte/word within the line.
4. If no match in any way: **Cache miss**. Fetch the line from the next level.

### 6.5 Example: 1 KB direct‑mapped cache, 64-byte lines, 32-bit addresses

- **Line size**: 64 bytes → offset = 6 bits
- **Total cache size**: 1 KB = 1024 bytes
- **Number of lines**: 1024 / 64 = 16 lines
- **Direct‑mapped**: 1 way, so 16 sets
- **Index bits**: $\log_2(16) = 4$ bits
- **Tag bits**: $32 - 6 - 4 = 22$ bits

Address breakdown:

```
┌────────── 22 bits ──────────┬─ 4 bits ─┬─ 6 bits ─┐
│         Tag                  │  Index   │  Offset  │
└──────────────────────────────┴──────────┴──────────┘
```

When you access address `0x00401A3C`:
- Offset = `0x3C & 0x3F = 0x3C` (byte 60 within the line)
- Index = `(0x00401A3C >> 6) & 0xF = 0x6` (set 6)
- Tag = `0x00401A3C >> 10 = 0x1006`

The cache checks set 6. If the stored tag is `0x1006` and the valid bit is 1, it's a hit. Otherwise, it's a miss.

---

## 7. Cache mapping schemes

How do we decide which memory block can go in which cache location? There are three main strategies.

### 7.1 Direct‑mapped cache

**Rule**: Each memory block maps to exactly one cache set.

- Set index = (block address) mod (number of sets)
- Only 1 way, so there's only one possible location.

**Pros**:
- Simple and fast: no need to search multiple locations; just compute the index and check one tag.

**Cons**:
- **Conflict misses**: If two frequently accessed blocks map to the same set, they'll keep evicting each other, even if the rest of the cache is empty. This is called "thrashing."

**Use case**: Simple designs, small caches.

### 7.2 Fully associative cache

**Rule**: Any memory block can go in any cache line.

- No index bits—the entire address (minus offset) is the tag.
- Must search **all** cache lines in parallel to find a match.

**Pros**:
- No conflict misses: blocks can go anywhere, so you won't thrash unless the cache is truly full.

**Cons**:
- Expensive: requires parallel comparison of all tags (need $N$ comparators for $N$ lines).
- Doesn't scale to large caches.

**Use case**: Very small caches (e.g., TLBs with ~64 entries).

### 7.3 Set‑associative cache (N‑way)

**Rule**: Compromise between direct‑mapped and fully associative.

- The cache is divided into $S$ sets.
- Each set has $N$ ways (lines).
- A block maps to a specific set (via index bits), but can occupy any of the $N$ ways within that set.

**Pros**:
- Reduces conflict misses compared to direct‑mapped.
- More practical than fully associative: only need to search $N$ ways (typically 2, 4, 8, or 16), not all lines.

**Cons**:
- More complex than direct‑mapped: need $N$ comparators and replacement logic.

**Use case**: Most modern caches (L1, L2, L3 are typically 4‑way to 16‑way set‑associative).

### 7.4 Comparison table

| Type              | Ways per set | Conflict misses | Hardware complexity | Typical use           |
|-------------------|--------------|-----------------|---------------------|-----------------------|
| Direct‑mapped     | 1            | High            | Low                 | Simple systems        |
| 2‑way set‑assoc.  | 2            | Medium          | Medium              | Small caches          |
| 4–8‑way set‑assoc.| 4–8          | Low             | Medium              | Modern L1/L2/L3       |
| Fully associative | All          | Very low        | High                | TLBs, tiny caches     |

---

## 8. Replacement policies

When a cache miss occurs and all ways in the target set are occupied, which line do you evict? This is the **replacement policy**.

### 8.1 Random

**Idea**: Pick a victim at random.

**Pros**: Simple to implement, no bookkeeping.

**Cons**: Might evict a frequently used line by chance.

### 8.2 FIFO (First‑In, First‑Out)

**Idea**: Evict the line that has been in the cache the longest.

**Pros**: Fair, simple to track (just a timestamp or queue).

**Cons**: Doesn't consider how often a line is used—might evict something you just accessed many times.

### 8.3 LRU (Least Recently Used)

**Idea**: Evict the line that hasn't been accessed for the longest time.

**Pros**: Works well in practice because it respects temporal locality—lines you haven't used recently are less likely to be needed soon.

**Cons**: Requires tracking access order, which gets expensive for high associativity (e.g., 8‑way LRU needs to track 8! possible orderings).

### 8.4 Pseudo‑LRU

**Idea**: Approximate LRU with simpler hardware (e.g., using a binary tree of bits to narrow down which line to evict).

**Pros**: Much cheaper than true LRU, still performs well.

**Cons**: Not perfect LRU, but good enough for most workloads.

### 8.5 In practice

Most modern processors use pseudo‑LRU or other approximations. For small caches or TLBs, true LRU is feasible. For very simple designs, random or FIFO suffices.

---

## 9. Write policies: handling stores

Loads are straightforward: fetch the data from the cache (or lower levels on a miss). But what happens when you write to memory? We need policies for **write hits** and **write misses**.

### 9.1 Write‑hit policies

#### Write‑through

**Idea**: On a write hit, update **both** the cache **and** the next level (e.g., main memory) immediately.

**Pros**:
- Simple: cache and memory are always consistent.
- No "dirty" state to track.

**Cons**:
- Every write generates traffic to the next level → higher bandwidth usage, slower.

#### Write‑back

**Idea**: On a write hit, update **only** the cache and mark the line as "dirty." Write the line back to memory only when it's evicted.

**Pros**:
- Multiple writes to the same line don't generate multiple memory transactions → saves bandwidth.
- Faster average write time.

**Cons**:
- More complex: need a dirty bit per line, and a write‑back buffer.
- Cache and memory temporarily inconsistent.

### 9.2 Write‑miss policies

#### Write allocate (fetch on write)

**Idea**: On a write miss, load the entire line into the cache (just like a read miss), then perform the write in the cache.

**Pros**:
- Subsequent writes to nearby locations (spatial locality) hit in the cache.
- Works well with write‑back.

**Cons**:
- Fetching a whole line just to write a few bytes can waste bandwidth.

#### No‑write allocate (write around)

**Idea**: On a write miss, write directly to the next level; don't bring the line into the cache.

**Pros**:
- Avoids fetching data you might not read.
- Works well with write‑through.

**Cons**:
- If you write to the same address again soon, it's still a miss.

### 9.3 Common combinations

- **Write‑through + No‑write allocate**: Simple and consistent; used in some embedded systems.
- **Write‑back + Write allocate**: Maximizes performance by exploiting locality; used in most high‑performance processors.

---

## 10. Cache performance: AMAT

How do we quantify cache performance? One useful metric is **Average Memory Access Time (AMAT)**.

### 10.1 The formula

For a single‑level cache:

$$
\text{AMAT} = \text{Hit time} + \text{Miss rate} \times \text{Miss penalty}
$$

Where:
- **Hit time**: Time to access the cache when the data is present (e.g., 2 cycles for L1).
- **Miss rate**: Fraction of accesses that miss (e.g., 0.05 = 5%).
- **Miss penalty**: Additional time to fetch from the next level on a miss (e.g., 100 cycles to go to DRAM).

### 10.2 Example: single-level cache

- L1 hit time = 2 cycles
- L1 miss rate = 5% = 0.05
- DRAM access time = 200 cycles

$$
\text{AMAT} = 2 + 0.05 \times 200 = 2 + 10 = 12 \text{ cycles}
$$

Even though only 5% of accesses miss, the high penalty makes the average access time 6× slower than a pure hit.

### 10.3 Multi‑level caches

For a two‑level hierarchy (L1 and L2):

$$
\text{AMAT} = T_{\text{L1}} + MR_{\text{L1}} \times \left( T_{\text{L2}} + MR_{\text{L2}} \times T_{\text{mem}} \right)
$$

Where:
- $T_{\text{L1}}$ = L1 hit time
- $MR_{\text{L1}}$ = L1 miss rate
- $T_{\text{L2}}$ = L2 access time (from L1 miss)
- $MR_{\text{L2}}$ = L2 local miss rate (misses in L2 relative to L2 accesses)
- $T_{\text{mem}}$ = Main memory access time

### 10.4 Example: two-level cache

- L1 hit time = 2 cycles, L1 miss rate = 5%
- L2 access time = 10 cycles, L2 local miss rate = 20%
- DRAM access time = 200 cycles

$$
\begin{align*}
\text{AMAT} &= 2 + 0.05 \times (10 + 0.20 \times 200) \\
            &= 2 + 0.05 \times (10 + 40) \\
            &= 2 + 0.05 \times 50 \\
            &= 2 + 2.5 \\
            &= 4.5 \text{ cycles}
\end{align*}
$$

Adding an L2 cache reduced AMAT from 12 cycles (no L2) to 4.5 cycles—a significant improvement!

### 10.5 Takeaway

- **Hit rate** (1 − miss rate) is critical: even a small improvement can have a big impact.
- **Miss penalty** matters: faster lower-level memory or prefetching can reduce AMAT.
- **Multi‑level caches** smooth out the performance gap between fast registers and slow DRAM.

---

## 11. Virtual memory: abstraction and protection

So far, we've assumed the CPU directly accesses **physical memory** using physical addresses. But modern systems use **virtual memory** to provide abstraction, protection, and flexibility.

### 11.1 What is virtual memory?

Every process sees its own **virtual address space**—a large, contiguous range of addresses (e.g., 0 to $2^{64} - 1$ on a 64‑bit system). The hardware and operating system transparently map these virtual addresses to **physical addresses** in RAM.

This gives us several benefits:

1. **Isolation**: Each process has its own address space. One process can't accidentally (or maliciously) access another's memory.
2. **Simplicity**: Programmers don't need to worry about where in physical RAM their code/data lives—it all looks contiguous.
3. **Overcommitment**: The total virtual memory of all processes can exceed physical RAM, with the OS paging unused data to disk.

### 11.2 Pages and frames

Virtual memory is divided into fixed‑size chunks called **pages** (commonly 4 KiB, but larger pages like 2 MiB or 1 GiB are also used).

Physical memory is divided into **frames** (same size as pages).

The OS maintains a **page table** for each process, mapping **virtual page numbers (VPN)** to **physical frame numbers (PFN)**. The VPN is simply the high-order bits of a virtual address that identify which page you're accessing, while the PFN identifies the corresponding physical frame in RAM.

### 11.3 Address translation: Virtual Page Number (VPN) to Physical Frame Number (PFN)

A virtual address is split into:

```
┌──────────────────┬──────────────────┐
│   Page Number    │   Page Offset    │
│      (VPN)       │                  │
└──────────────────┴──────────────────┘
```

For a 4 KiB page, the offset is 12 bits (since $2^{12} = 4096$). The remaining bits identify the virtual page.

**Translation process**:
1. Use the VPN to index into the page table.
2. Look up the corresponding PFN (and check permissions).
3. Replace the VPN with the PFN to form the physical address.
4. Append the page offset (unchanged).

```
Virtual:  ┌────── VPN ──────┬── Offset ──┐
                   │
                   ▼
          [Page Table Lookup]
                   │
                   ▼
Physical: ┌────── PFN ──────┬── Offset ──┐
```

### 11.4 Page table entries (PTEs)

Each entry in the page table contains:

- **PFN**: The physical frame number.
- **Valid bit**: Is this page currently in RAM?
- **Permissions**: Read, write, execute bits.
- **Dirty bit**: Has the page been written to?
- **Accessed bit**: Has the page been recently used (for page replacement)?

If the valid bit is 0, accessing the page triggers a **page fault**, and the OS must load the page from disk.

### 11.5 The TLB (Translation Lookaside Buffer)

Walking the page table on every memory access would be too slow (often requires multiple memory accesses for multi‑level page tables).

Solution: the **TLB** (Translation Lookaside Buffer), a small, fast cache of recent VPN → PFN translations (recent mappings from virtual page numbers to physical frame numbers).

**TLB lookup**:
1. Check if the VPN (virtual page number) is in the TLB.
   - **TLB hit**: Use the cached PFN. Fast (typically <1 cycle).
   - **TLB miss**: Walk the page table to find the PFN, then fill the TLB.

The TLB exploits temporal and spatial locality: if you access page $P$ now, you're likely to access it (or nearby pages) again soon.

**Typical TLB**: 64–512 entries, fully associative or highly set‑associative, with a hit rate of 95–99% for most workloads.

### 11.6 Putting it all together: L1 cache + TLB

Modern processors perform address translation and cache lookup **in parallel** (for virtually indexed, physically tagged caches) or **in sequence**. Here's a simplified flow:

1. **Instruction fetch** or **load/store** generates a virtual address.
2. **TLB lookup**:
   - Hit: Get the PFN quickly.
   - Miss: Walk the page table (hardware or software), fill TLB.
3. **Form physical address** from PFN + offset.
4. **L1 cache lookup** using the physical address:
   - Hit: Return data.
   - Miss: Check L2, L3, DRAM as before.

If the page is not in RAM (valid bit = 0), a **page fault** occurs:
- The OS selects a victim page, writes it to disk if dirty, loads the requested page from disk, updates the page table, and resumes the instruction.

Page faults are extremely expensive (millions of cycles), so the OS tries to keep the working set in RAM.

---

## 12. Modeling memory in Verilog

Let's see how to model simple memory structures in RTL.

### 12.1 A basic synchronous RAM

Here's a 256‑byte memory with synchronous read and write:

```verilog
module byte_ram (
    input  wire        clk,
    input  wire        we,       // write enable
    input  wire [7:0]  addr,     // 8-bit address → 256 locations
    input  wire [7:0]  din,      // data in
    output reg  [7:0]  dout      // data out
);
    // Storage array
    reg [7:0] mem [0:255];

    always @(posedge clk) begin
        if (we)
            mem[addr] <= din;   // Write
        dout <= mem[addr];      // Read (always happens)
    end
endmodule
```

**Note**: The read occurs on every cycle, regardless of `we`. Some designs use a separate read enable. Also, the read‑during‑write behavior (what happens if you read and write the same address?) depends on your synthesis tool and target.

### 12.2 A word‑addressed RAM

For a 32‑bit word RAM:

```verilog
module word_ram #(
    parameter ADDR_WIDTH = 10,  // 1024 words
    parameter DATA_WIDTH = 32
)(
    input  wire                      clk,
    input  wire                      we,
    input  wire [ADDR_WIDTH-1:0]     addr,
    input  wire [DATA_WIDTH-1:0]     din,
    output reg  [DATA_WIDTH-1:0]     dout
);
    reg [DATA_WIDTH-1:0] mem [0:(1 << ADDR_WIDTH)-1];

    always @(posedge clk) begin
        if (we)
            mem[addr] <= din;
        dout <= mem[addr];
    end
endmodule
```

### 12.3 Byte‑enable support

Real memories often support writing only certain bytes of a word:

```verilog
module byte_enable_ram (
    input  wire        clk,
    input  wire [3:0]  we,      // 4-bit write enable (one per byte)
    input  wire [9:0]  addr,
    input  wire [31:0] din,
    output reg  [31:0] dout
);
    reg [31:0] mem [0:1023];

    always @(posedge clk) begin
        if (we[0]) mem[addr][ 7: 0] <= din[ 7: 0];
        if (we[1]) mem[addr][15: 8] <= din[15: 8];
        if (we[2]) mem[addr][23:16] <= din[23:16];
        if (we[3]) mem[addr][31:24] <= din[31:24];
        dout <= mem[addr];
    end
endmodule
```

This allows the CPU to write a single byte or halfword without a read‑modify‑write cycle.

### 12.4 Dual‑port RAM

Some designs need two simultaneous accesses (e.g., a register file with two read ports and one write port):

```verilog
module dual_port_ram (
    input  wire        clk,
    // Port A (read/write)
    input  wire        we_a,
    input  wire [7:0]  addr_a,
    input  wire [31:0] din_a,
    output reg  [31:0] dout_a,
    // Port B (read only)
    input  wire [7:0]  addr_b,
    output reg  [31:0] dout_b
);
    reg [31:0] mem [0:255];

    always @(posedge clk) begin
        // Port A
        if (we_a)
            mem[addr_a] <= din_a;
        dout_a <= mem[addr_a];
        // Port B
        dout_b <= mem[addr_b];
    end
endmodule
```

Dual‑port RAMs are common in FPGAs as BRAM primitives.

---

## 13. Orders of magnitude: latency and bandwidth

It's helpful to have a mental picture of how fast different parts of the memory hierarchy are.

### 13.1 Typical latencies (ballpark)

| Level           | Latency (cycles) | Latency (time)  | Analogy                      |
|-----------------|------------------|-----------------|------------------------------|
| L1 cache        | 1–4              | ~1 ns           | Grabbing a book from your desk |
| L2 cache        | 10–20            | ~5–10 ns        | Walking to a nearby bookshelf |
| L3 cache        | 30–70            | ~20–30 ns       | Going to another room        |
| DRAM (main mem) | 100–300          | ~50–100 ns      | Driving to the library       |
| SSD             | ~25,000          | ~100 μs         | Ordering a book online       |
| HDD (random)    | ~1,000,000       | ~10 ms          | Waiting weeks for delivery   |

These numbers vary by architecture, but the **orders of magnitude** are consistent: each level down the hierarchy is roughly **10× to 100×** slower.

### 13.2 Bandwidth

Latency is the time to start getting data; **bandwidth** is how much data you can transfer per second once you've started.

- **L1 → CPU**: Hundreds of GB/s (multiple loads/stores per cycle)
- **DRAM → CPU**: Tens of GB/s (limited by memory bus width and speed)
- **SSD**: Several GB/s (NVMe drives)
- **HDD**: Hundreds of MB/s

For sequential access, bandwidth matters more than latency. For random access, latency dominates.

---

## 14. Worked examples

Let's solidify our understanding with detailed examples.

### 14.1 Example 1: Cache address breakdown

**Problem**: A 32 KiB, 4‑way set‑associative cache with 64‑byte lines. Addresses are 32 bits. How many bits for tag, index, and offset?

**Solution**:
1. **Offset bits**: Line size = 64 bytes = $2^6$ → offset = 6 bits.
2. **Total lines**: Cache size / line size = 32 KiB / 64 B = $32{,}768 / 64 = 512$ lines.
3. **Number of sets**: Lines / associativity = $512 / 4 = 128$ sets = $2^7$ → index = 7 bits.
4. **Tag bits**: Remaining bits = $32 - 6 - 7 = 19$ bits.

**Answer**: 19‑bit tag, 7‑bit index, 6‑bit offset.

### 14.2 Example 2: AMAT calculation

**Problem**: 
- L1 hit time = 2 cycles, miss rate = 4%
- L2 access time = 12 cycles, local miss rate = 30%
- Main memory access time = 180 cycles

Compute AMAT.

**Solution**:

$$
\begin{align*}
\text{AMAT} &= T_{L1} + MR_{L1} \times (T_{L2} + MR_{L2} \times T_{mem}) \\
            &= 2 + 0.04 \times (12 + 0.30 \times 180) \\
            &= 2 + 0.04 \times (12 + 54) \\
            &= 2 + 0.04 \times 66 \\
            &= 2 + 2.64 \\
            &= 4.64 \text{ cycles}
\end{align*}
$$

**Answer**: 4.64 cycles on average.

### 14.3 Example 3: Endianness

**Problem**: On a little‑endian machine, the 32‑bit value `0xDEADBEEF` is stored starting at address `0x2000`. What byte is at address `0x2001`?

**Solution**: Little‑endian stores the LSB first.

```
Address    Byte
0x2000     0xEF  (LSB)
0x2001     0xBE
0x2002     0xAD
0x2003     0xDE  (MSB)
```

**Answer**: `0xBE`.

### 14.4 Example 4: Alignment check

**Problem**: Is a 4‑byte integer aligned at address `0x1004`? What about `0x1002`?

**Solution**: A 4‑byte value is aligned if its address is a multiple of 4.

- `0x1004 / 4 = 0x401` (exact) → **Aligned**.
- `0x1002 / 4 = 0x400.5` (not exact) → **Misaligned**.

**Answer**: `0x1004` is aligned, `0x1002` is not.

---

## 15. Practice exercises

Try these on your own before checking the answers.

### Exercise 1: Cache parameters

A 128 KiB, 8‑way set‑associative cache with 128‑byte lines. Addresses are 32 bits. Find:
(a) Offset bits  
(b) Number of sets  
(c) Index bits  
(d) Tag bits

### Exercise 2: AMAT with write‑back

You have an L1 cache with:
- Hit time = 1 cycle
- Miss rate = 3%
- Write‑back policy: 50% of evictions are dirty and require a write‑back (adds 50 cycles)
- Miss penalty (fetch from L2) = 20 cycles

Estimate the effective AMAT assuming reads and writes are equally likely.

### Exercise 3: Page table calculation

A system uses 4 KiB pages and 32‑bit virtual addresses. How many bits are used for the page offset? How many bits for the virtual page number?

### Exercise 4: Endianness conversion

You read the bytes `0x12 0x34 0x56 0x78` from addresses `0x100` to `0x103` (in increasing order). If the machine is big‑endian, what is the 32‑bit value? What if it's little‑endian?

### Exercise 5: Cache conflict misses

A direct‑mapped cache has 256 lines of 64 bytes each. Addresses are 32 bits. You repeatedly access addresses `0x1000`, `0x5000`, and `0x9000`. Will these addresses conflict (map to the same set)?

**Hint**: Compute the set index for each address.

---

## 16. Solutions to exercises

### Solution 1

(a) Offset = $\log_2(128) = 7$ bits  
(b) Total lines = $128{,}KiB / 128{,}B = 1024$; sets = $1024 / 8 = 128$  
(c) Index = $\log_2(128) = 7$ bits  
(d) Tag = $32 - 7 - 7 = 18$ bits

### Solution 2

Read miss penalty = 20 cycles  
Write miss penalty = 20 (fetch) + 0.5 × 50 (write‑back) = 20 + 25 = 45 cycles (on average)  
Combined miss penalty (assuming equal read/write) = (20 + 45)/2 = 32.5 cycles  
AMAT ≈ $1 + 0.03 \times 32.5 = 1 + 0.975 \approx 2$ cycles

(Note: In practice, you'd separate read and write traffic or use a more detailed model.)

### Solution 3

Page size = 4 KiB = $2^{12}$ → offset = 12 bits  
VPN bits = $32 - 12 = 20$ bits

### Solution 4

Big‑endian (MSB first): read `12 34 56 78` → value is `0x12345678`  
Little‑endian (LSB first): read `12 34 56 78`, but they represent `78 56 34 12` → value is `0x78563412`

### Solution 5

Index bits: offset = 6, total sets = 256 → index uses bits [6:13]

- `0x1000`: index = `(0x1000 >> 6) & 0xFF = 0x40 & 0xFF = 0x40`
- `0x5000`: index = `(0x5000 >> 6) & 0xFF = 0x140 & 0xFF = 0x40`
- `0x9000`: index = `(0x9000 >> 6) & 0xFF = 0x240 & 0xFF = 0x40`

All three map to set `0x40` → **Yes, they conflict**. You'll get thrashing if you access them repeatedly.

---

## 17. Common pitfalls and misconceptions

### Misconception 1: "Bigger cache is always better"

**Reality**: Beyond a certain size, caches have diminishing returns. A huge cache:
- Takes more die area and power
- Has longer access latency (harder to search/route signals)
- May not improve hit rate if the working set is small

It's about finding the sweet spot for your workload.

### Misconception 2: "The TLB is just another cache"

**Reality**: The TLB caches *translations*, not data. A TLB miss doesn't mean you have to go to DRAM—it means you have to walk the page table (which itself might be cached in L1/L2). Still, TLB misses are expensive, especially for workloads with large, scattered memory footprints.

### Misconception 3: "Alignment only matters for old architectures"

**Reality**: Even on x86, which tolerates misalignment, misaligned accesses can be slower. For ARM and RISC‑V, misaligned accesses might trap or require software emulation (very slow). And in multithreaded code, misaligned atomics might not even be atomic.

### Misconception 4: "Virtual memory is just for security"

**Reality**: Virtual memory also provides *convenience* (each process gets a simple, contiguous address space) and *flexibility* (the OS can move pages around in physical memory without updating pointers in your program).

---

## 18. Where this goes next

We've now covered the essentials of memory systems. In future sessions and modules, we'll see how these concepts connect to the rest of the processor:

- **ISA and instruction encoding**: How load/store instructions specify addresses and data sizes.
- **Pipeline hazards**: How cache misses stall the pipeline and impact performance.
- **Memory consistency models**: What guarantees the hardware provides for multithreaded programs.
- **Advanced cache coherence**: How multi‑core systems keep caches synchronized.
- **Memory controllers**: The interface between the CPU and DRAM, including burst modes and refresh.

For now, make sure you're comfortable with:
- Address breakdown (tag/index/offset)
- Cache mapping (direct, associative, set‑associative)
- Replacement and write policies
- AMAT calculations
- Virtual memory and the TLB

These are the building blocks you'll use again and again as we design and analyze real systems.

---

## 19. Further reading and resources

- **Computer Organization and Design** by Patterson & Hennessy (chapters on memory hierarchy)
- **Computer Architecture: A Quantitative Approach** by Hennessy & Patterson (more advanced treatment)
- **What Every Programmer Should Know About Memory** by Ulrich Drepper (deep dive into DRAM, caches, and performance)
- **Intel and AMD optimization manuals** (for real‑world cache sizes and latencies)
- **ARM Architecture Reference Manual** (for alignment and endianness details)

---

That's it for Session 4! Take your time digesting this material. Memory systems can feel abstract at first, but once you internalize the hierarchy and the principles of locality, everything else falls into place. Try the exercises, experiment with cache simulators if you can find them, and don't hesitate to revisit sections as needed.

Happy learning, and see you in the next session!
