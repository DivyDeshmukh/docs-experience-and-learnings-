# Variable (or Dynamic) Partitioning in Operating System

Variable (Dynamic) Partitioning is a **contiguous memory allocation technique** where memory partitions are created **at run-time based on the size of the process requesting memory**. Unlike fixed partitioning, partitions are **not predefined during system configuration**.

This method was introduced to overcome the limitations of **fixed partitioning**, especially **internal fragmentation**.

---

## How Variable Partitioning Works

1. Initially, the **entire RAM is free and unpartitioned**.  
2. When a process arrives, the **operating system allocates exactly the amount of memory required by that process**.  
3. Each partition is **created dynamically and may differ in size**.  
4. When a process finishes execution, its allocated memory is **released and becomes a free hole**.  
5. The **number of partitions in memory changes dynamically** based on process arrival and termination.

---

## Key Features of Variable Partitioning

- Partitions are **created during run-time**, not in advance.  
- The **partition size equals the process size**, avoiding wasted memory.  
- The **number of partitions is not fixed**.  
- **Memory utilization is more efficient** compared to fixed partitioning.  
- Processes are loaded **until the memory is fully utilized**.

---

## Advantages

**No Internal Fragmentation**  
Memory is allocated exactly as per the process size, so **no space is wasted inside a partition**.

**Better Memory Utilization**  
Since internal wastage is eliminated, **available RAM is used more efficiently**.

**No Fixed Partition Size Limitation**  
A process of **any size can be loaded** as long as sufficient contiguous memory is available.

**Higher Degree of Multiprogramming**  
More processes can be accommodated in memory due to **flexible partition sizes**.

---

## Disadvantages

**External Fragmentation**  
Free memory gets divided into **small non-contiguous blocks**, making it difficult to allocate memory for large processes.

**Complex Implementation**  
Memory allocation and deallocation are done **at run-time**, making management more complicated than fixed partitioning.

**Compaction Overhead**  
To reduce external fragmentation, **compaction is required**, which consumes CPU time and system resources.

**Slower Allocation Process**  
Searching for a suitable free block (**first fit, best fit, worst fit**) increases allocation time.

---

## Important Note

Even though **internal fragmentation does not occur**, a process may still fail to get memory due to **external fragmentation**.

For example:

If **P5 requires 3 MB** and the total free memory available is **3 MB**, but it is split into **multiple small holes**, the process **cannot be allocated memory** because **contiguous allocation does not allow spanning across holes**.

---

## Key Points on Variable (Dynamic) Partitioning in Operating Systems

- Memory partitions are **created and resized dynamically**.  
- The OS **maintains a list of free memory holes**.  
- **Allocation algorithms** are used to find suitable memory blocks.  
- **Internal fragmentation is eliminated**, but **external fragmentation remains**.  
- **Compaction may be required** to reduce fragmentation.  
- Widely studied for understanding **modern memory management concepts**.
