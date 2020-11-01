# Caches

Caches store small parts of memory that may be frequently accessed.

## Direct Mapped Caches
A direct mapped cache corresponds to a specific chunk of memory.
- Data is stored as "blocks".
- Indices distinguish different blocks.
- Multiplying the indices of a cache by its block size
returns the total amount of storage that a cache has.

Each index in a cache may correspond to multiple memory addresses.
To determine which block an address corresponds to, we can mod the address by the size of the cache and divide it by the block size.
This amounts to only isolating specific bits in the memory address.

To determine which memory address a block currently corresponds to, we use **tags**.
- Every repeated "group" of memory is assigned a number
- The tag stores the number indicating which section of memory the data is from

Memory addresses are divided into three fields, all read as unsigned integers:
- *Tag*: Distinguishes between memory addresses that map to the same location
    - Uses the remaining bits after the index and offset length have been determined
- *Index*: Specifies the index of the block in the cache
    - Bit length must be able to represent the number of blocks
- *Offset*: Specifies the desired byte in the block
    - Bit length must be able to represent the block size

## Fully Associative Caches
Uses the same tags and offsets as direct mapped caches, but no indices. Any block can go anywhere in the cache, and *all* tags must
be compared when finding data. This can be done in parallel.

Fully associative caches *eliminate conflict misses*, since data can be stored anywhere. However, a comparator is needed for every single entry.

## Set Associative Caches
Set associative caches are a mixture of direct mapped caches and fully associative caches. Once again, the tag and offset of the memory addresses
keep their original functionality; however, the index now points to the correct "set".

Hence, the cache is direct-mapped with respect to sets.
Each set contains multiple blocks, each of which are fully associative with their respective memory chunk; to identify the correct block, we compare
all tags.

This solves the issue of hardware cost with respect to comparators.

## Cache Terminology
When reading memory:
- Cache hit: The cache block is valid with the correct memory address, so we read the data
- Cache miss: The respective cache block is empty, so we fetch and copy from memory
- Cache miss, block replacement: The wrong data is at the cache block, so we fetch and recopy from memory

The state and usage of the cache:
- Cold: Empty.
- Warming: Beginning to fill with values, often at the beginning of a program.
- Warm: The cache is being hit fairly often.
- Hot: The cache is performing very well with a high percentage of hits.

Performance of a cache:
- Hit rate: The fraction of accesses that hit the cache.
- Miss rate: The fraction of accesses that miss the cache.
- Hit time: The time needed to access cache memory.
- Miss penalty: The time needed to replace a block after a miss.

The *valid bit* indicates whether a block in the cache is storing valid data or garbage.

Types of misses:
- Compulsory misses: Occur when a program is first started and all blocks are invalid
    - Every block in memory must have at least one compulsory miss
- Conflict misses: Occur because two distinct memory addresses map to the same cache location; tag is incorrect
- Capacity misses: Occur when the cache no longer has space to copy data from memory.
    - The primary type of miss for fully associative caches

## Writing
There are two different options for handling writes to memory.
- Write-through:
    - Updates both cache and memory
    - No need to worry about inconsistencies
- Write-back:
    - Update only the cache block
    - Inconsistent memory is now *stale*.
    - Add a *dirty* bit to the block to indicate that memory should be updated when the block is replaced.

When writing into a set associative and fully associative caches, we must choose which blocks to write to.
- If an invalid block exists, we write into the first invalid block
- If all blocks are valid, we rely on one of the following *replacement policies*:
    - Least Recently Used (LRU): The most common policy, in which the least accessed block is cached out.
        - Takes advantage of **temporal locality**; more commonly used blocks will likely be used again in the future.
        - Harder to keep track of with higher associativity
    - First-In, First-Out (FIFO): Ignores accesses and tracks the order in which blocks are cached. The "oldest" block is replaced first.
        - Ignores temporal locality but is much easier to track
    - Random: Randomly selects a block to replace.
        - Can be effective in processes with low temporal locality

## Block Size
Having a larger block size grants the following benefits:
- **Spacial locality**: If we access a given word, we are likely to access nearby words as well
- Works well for sequential array accesses

However, a larger block size induces a larger *miss penality*.
Furthermore, if the block size is too big relative to cache size,
then there are fewer blocks and the *miss rate* increases.

## Average Memory Access time
AMAT = Hit Time + (Miss Rate) * (Miss Penality)

Multiple levels of caches may be added to reduce the miss penality.

Then, AMAT = Hit Time + (L1 Miss Rate) * (L1 Miss Penality) where:
- L1 Miss Penality = L2 Hit Time + (L2 Miss Rate) * (L2 Miss Penality)