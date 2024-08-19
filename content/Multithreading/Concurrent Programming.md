---
draft: true
---

The scope of this document is to go through the different aspects of what concurrent programming means and some key concepts. 

When talking about concurrent or parallel programming it's important to note that the number of cores that the host machine's CPU has will play a crucial role. We can have single core CPUs and multi-core CPUs and the way concurrent programming is handled is completely different.
**Concurrency** is the art or process of doing multiple things *at the same time*.

The concept of *same time* differs based on the CPU type:
- For a **single core CPU** all the processes happen one after the other but **fast enough** as to not be perceived as different processes. So for single cores there is no *at the same time* as all the processes happen at different points in time.
- For **multi-core CPU** the processes can be ran by different cores and, as such they are actually happening at the same point in time.
 
#### Thread
A **thread** is defined at the OS level and it can be considered a set of instructions to be run by CPU. An application can be composed of multiple threads that can run *at the same time*.
The JVM works with several threads:
- Garbage Collector (**GC**)
- Just in time compiler (**JIT**)
