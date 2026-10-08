# CPU Cache Simulator

An interactive visual simulator for understanding how a CPU memory access is resolved by a cache.

Instead of treating a cache as a black box, this simulator exposes the complete decision path of every access:

```text
ADDRESS
   ↓
CACHE SET
   ↓
TAG COMPARISON
   ↓
HIT / MISS
   ↓
MEMORY
   ↓
CACHE LINE FILL
   ↓
CPU
```

You provide a memory address, the simulator decodes it into **tag, index, and offset**, selects the corresponding cache set, compares the tag against the available ways, and then visualizes exactly what happens next.

> **Teaching model — not a cycle-accurate model of a commercial CPU.**

---

## Why this exists

Cache behavior is usually explained using diagrams and formulas.

That works until you have to answer questions like:

- Why did this address map to this set?
- Which bits are the tag?
- Why was this access a miss?
- Which cache line was compared?
- Why was a particular line evicted?
- What actually moves between memory and cache?
- What changes after a line fill?
- Why does a repeated access become a hit?

This simulator makes those operations observable.

Every access can be stepped through one stage at a time, while the corresponding cache set, cache line, address decoder, transfer state, and statistics update with it.

---

## Core interaction

The central interaction is deliberately physical:

```text
┌─────────┐
│   CPU   │
└────┬────┘
     │ Address
     ▼
┌─────────────┐
│    CACHE    │
│             │
│ Set select  │
│ Tag compare │
└─────┬───────┘
      │
      ├──────── HIT ──────────► CPU
      │
      │ MISS
      ▼
┌─────────────┐
│ Main Memory │
└──────┬──────┘
       │
       │ Cache line
       ▼
┌─────────────┐
│    CACHE    │
│ Line fill   │
└──────┬──────┘
       │
       ▼
      CPU
```

The UI reflects this process directly rather than presenting cache behavior as a collection of disconnected numbers.

---

## What you can explore

### Address decomposition

For every address, the simulator calculates:

```text
┌────────────── TAG ──────────────┬──── INDEX ────┬── OFFSET ──┐
│                                 │               │            │
└─────────────────────────────────┴───────────────┴────────────┘
```

The decoder displays:

- Tag
- Index
- Offset
- Address in binary
- Selected set
- Memory block
- Cache-line address range

The number of bits in each field is derived from the active cache configuration.

---

### Cache geometry

Configure the simulator with different:

- Address sizes
- Cache sizes
- Cache-line sizes
- Associativities
- Replacement policies

Supported associativities:

```text
1-way
2-way
4-way
8-way
Fully associative
```

The simulator automatically derives:

```text
Number of lines
Number of sets
Offset bits
Index bits
Tag bits
Number of ways
```

Invalid configurations are rejected rather than silently producing misleading results.

---

## Replacement policies

The simulator supports:

### LRU

Least Recently Used.

The line that has been unused for the longest time becomes the replacement candidate.

### FIFO

First In, First Out.

The oldest inserted cache line is selected.

### Random

A deterministic pseudo-random replacement sequence is used so that the same simulation produces reproducible behavior.

The implementation uses a seeded random generator rather than relying directly on non-deterministic browser randomness.

---

## Step-by-step execution

A memory access can be executed incrementally.

Typical sequence:

```text
01  ADDRESS
02  CACHE SET
03  TAG COMPARISON
04  HIT / MISS
```

For a hit:

```text
ADDRESS
   ↓
CACHE SET
   ↓
TAG COMPARISON
   ↓
HIT
   ↓
CPU RETURN
```

For a miss:

```text
ADDRESS
   ↓
CACHE SET
   ↓
TAG COMPARISON
   ↓
MISS
   ↓
MEMORY ACCESS
   ↓
CACHE LINE TRANSFER
   ↓
INSTALL
   ↓
CPU RETURN
```

If the selected set is full, replacement and eviction become explicit stages:

```text
MISS
  ↓
REPLACEMENT
  ↓
EVICT
  ↓
MEMORY ACCESS
  ↓
CACHE LINE TRANSFER
  ↓
INSTALL
  ↓
CPU RETURN
```

The simulator internally models these stages as an actual state machine rather than merely changing labels on the screen.

---

## Access patterns

Built-in access patterns let you observe common cache behavior without manually constructing every address.

### Spatial locality

Access nearby addresses within the same region.

Useful for observing how one cache-line fill can satisfy subsequent nearby accesses.

### Temporal locality

Repeatedly access previously used addresses.

Useful for seeing previously loaded lines become hits.

### Stride access

Access memory with a fixed 16-byte stride.

Useful for examining how regular address patterns interact with cache-line boundaries and sets.

### Conflict miss

Generate addresses that compete for the same set.

Useful for understanding why increasing associativity can reduce conflict misses.

### Capacity miss

Generate an access sequence larger than the available cache capacity.

Useful for observing lines being displaced because the working set exceeds the cache.

These sequences are generated from the current cache geometry rather than being hard-coded for a single configuration.

---

## Cache visualization

The cache is rendered as a set/way structure:

```text
             Way 0        Way 1
          ┌──────────┐  ┌──────────┐
Set 0     │ V Tag    │  │ V Tag    │
          │ Data     │  │ Data     │
          └──────────┘  └──────────┘

          ┌──────────┐  ┌──────────┐
Set 1     │ V Tag    │  │ V Tag    │
          │ Data     │  │ Data     │
          └──────────┘  └──────────┘
```

Each line exposes:

- Valid bit
- Tag
- Memory block
- Data preview
- Set
- Way

During an access, the relevant set and cache line are visually highlighted.

After an access completes, individual cache lines can be selected to inspect their state.

---

## Hit and miss statistics

The simulator maintains live statistics for:

| Metric | Description |
|---|---|
| Accesses | Total completed memory accesses |
| Hits | Accesses resolved directly from cache |
| Misses | Accesses requiring a cache miss path |
| Hit rate | Hits / total accesses |
| Evictions | Misses that required replacing a valid line |

All statistics are derived from the simulator's internal cache state and access history.

---

## Access history

Every completed access is recorded with information including:

```text
#01   0x1000   HIT    set 0 · way 1
#02   0x1010   MISS   set 1 · way 0
#03   0x2000   MISS   set 0 · way 1 · evict 0x1000
```

This makes it possible to connect individual accesses with the resulting cache state and overall hit/miss statistics.

---

## Example

Consider a configuration such as:

```text
Address size:       16 bits
Cache size:         256 B
Line size:           16 B
Associativity:       2-way
Replacement:          LRU
```

The cache contains:

```text
256 / 16 = 16 cache lines
```

With 2-way associativity:

```text
16 / 2 = 8 sets
```

Therefore the address is divided into:

```text
Offset = log₂(16) = 4 bits
Index  = log₂(8)  = 3 bits
Tag    = 16 - 4 - 3
       = 9 bits
```

The simulator performs this calculation automatically and displays the resulting geometry.

---

## Getting started

There is no build system or dependency installation required.

The simulator is implemented as a self-contained HTML document containing:

```text
HTML
CSS
JavaScript
```

Open the HTML file in a modern browser.

```bash
# Clone the repository
git clone <your-repository-url>

# Enter the project
cd <repository-name>

# Open the HTML file
```

You can also serve it locally with any static HTTP server.

For example:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

---

## How to use it

### 1. Configure the cache

Choose:

- Address size
- Cache size
- Line size
- Associativity
- Replacement policy

### 2. Enter an address

Examples:

```text
0x1000
```

or:

```text
4096
```

Both hexadecimal and decimal addresses are supported.

### 3. Inspect the decoder

Before executing the access, examine how the address is divided into:

```text
TAG
INDEX
OFFSET
```

### 4. Access memory

Click:

```text
Access Memory
```

### 5. Step through the decision

Use:

```text
Step
```

to advance one stage at a time.

Or use:

```text
Run
```

to automatically execute the sequence.

### 6. Inspect the resulting cache

Observe:

- Selected set
- Compared ways
- Hit/miss result
- Evicted line
- Filled cache line
- Updated statistics
- Access history

---

## Design principles

The simulator intentionally follows a few principles.

### 1. Make the address visible

The requested address should never disappear into an abstraction.

The user should always be able to answer:

> What address am I accessing?

### 2. Make the mapping visible

The user should be able to see:

```text
Address
   ↓
Block
   ↓
Set
```

### 3. Make tag comparison visible

A cache hit should not simply appear as:

```text
HIT
```

The simulator shows the set and ways involved in the comparison.

### 4. Make misses physical

A miss results in a visible sequence:

```text
MISS
 ↓
Memory
 ↓
Cache-line transfer
 ↓
Installation
 ↓
CPU return
```

### 5. Preserve state

The cache is not a fake animation.

Accesses modify an actual simulated cache state, which affects subsequent accesses.

### 6. Keep the model honest

The simulator explicitly avoids claiming to reproduce the microarchitecture of a particular commercial processor.

---

## What this simulator models

The simulator models:

- Address decoding
- Cache lines
- Sets
- Ways
- Valid bits
- Tags
- Offsets
- Indexing
- Tag comparison
- Hits
- Misses
- Cache-line fills
- Eviction
- LRU replacement
- FIFO replacement
- Deterministic random replacement
- Access sequences
- Hit-rate statistics
- Cache state transitions

---

## What it intentionally does not model

This is **not** a cycle-accurate CPU simulator.

Real processors can include substantially more machinery, including:

- Hardware prefetching
- Cache coherence protocols
- Speculative execution
- Write buffers
- Miss Status Holding Registers (MSHRs)
- Multiple outstanding memory requests
- Hardware-specific replacement mechanisms
- Memory-level parallelism
- Detailed bus/interconnect behavior
- DRAM timing
- Store/load ordering behavior

The project deliberately keeps these out of scope so that the fundamental cache-access mechanism remains inspectable.

---

## Architecture

The application is intentionally small and dependency-free.

```text
┌──────────────────────────────────────┐
│               UI                     │
│                                      │
│  Configuration                       │
│  Address Input                       │
│  Presets                             │
│  Cache Visualization                 │
│  Decoder                             │
│  Decision Panel                      │
│  Statistics                          │
│  History                             │
└────────────────┬─────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────┐
│          Simulation State            │
│                                      │
│  Configuration                       │
│  Cache                               │
│  History                             │
│  Access Queue                        │
│  Pending Access                      │
│  Deterministic RNG                   │
└────────────────┬─────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────┐
│          Cache Model                 │
│                                      │
│  Geometry                            │
│  Address decoding                    │
│  Tag comparison                      │
│  Hit/miss resolution                 │
│  Victim selection                    │
│  Line installation                   │
│  LRU/FIFO/Random                     │
└──────────────────────────────────────┘
```

The simulation state is maintained separately from rendering, allowing the UI to reflect the underlying cache state rather than independently faking the visualization.

---

## Validation

The simulator validates cache configurations before running them.

Examples of invalid configurations include:

- Cache size that is not a power of two
- Line size that is not a power of two
- Line size larger than the cache
- Cache configurations without whole cache lines
- Invalid associativity
- Cache geometry exceeding the configured address space
- Excessively large cache configurations

Address input is also validated against the configured address width.

---

## Accessibility & interaction details

The interface includes:

- Keyboard-focusable controls
- Visible focus states
- ARIA status updates
- Address validation feedback
- Responsive layouts
- Reduced-motion support

The simulator also adapts its layout for smaller screens rather than assuming a desktop-only viewport.

---

## Project structure

The current implementation is intentionally contained in a single HTML file:

```text
CPU Cache Simulator
└── index.html
    ├── HTML structure
    ├── CSS
    └── JavaScript simulation engine
```

This makes the simulator easy to inspect, fork, modify, and deploy as a static page.

---

## Educational use

This project is useful for learning and teaching:

- Computer Organization & Architecture
- CPU caches
- Memory hierarchy
- Cache mapping
- Direct-mapped caches
- Set-associative caches
- Fully associative caches
- Locality
- Cache misses
- Replacement policies

It is particularly useful when learning concepts that are normally presented only through static diagrams.

---

## Limitations

The simulator intentionally uses a simplified teaching model.

For example:

- Data values are deterministic generated previews rather than real DRAM contents.
- Memory access has no physical latency model.
- Cache-line fills are represented as logical transfers.
- Replacement behavior is simplified to the selected policy.
- There is no write path.
- There is no coherence model.
- There is no multi-level cache hierarchy.
- There is no CPU pipeline or out-of-order execution model.

These limitations are intentional. The goal is to make the **cache decision process observable**, not to reproduce an actual processor.

---

## License

Add your preferred license here.

For example:

```text
MIT License
```

---

## Author

**Nithyananda Chalukya**

Built as an interactive low-level systems visualization project focused on making computer architecture concepts easier to inspect rather than simply memorize.