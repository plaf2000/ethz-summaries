## Java Threads: some key points

- Every Java program has at least one execution thread
  - First execution thread calls `main()`
- Each call to `start()` method of a Thread object creates an actual execution thread
- Program ends when all threads (non-daemon threads) finish.
- Threads **can continue** to run even if `main()` returns
- Creating a Thread object **does not start** a thread
- Calling `run()` **doesn’t start thread either** (need to call `start()`!)

## (Some) Useful Thread attributes and methods

ID: this attribute denotes the unique identifier for each Thread.

```java
Thread t = Thread.currentThread(); // get the current thread
System.out.println(“Thread ID” + t.getId()); // prints the current ID.
```

Name: this attribute denotes the name of Thread.

```java
t.setName(“PP“ + 2019); // can be modified like this
```

Priority: denotes the priority of the thread. Threads can have a priority between 1 and 10: JVM uses the priority of threads to select the one that uses the CPU at each moment

```javascript
t.setPriority(Thread.MAX_PRIORITY); // updates the thread’s priority
```

Status: denotes the status the thread is in: one of new, runnable, blocked, waiting, time waiting, or terminated (we will discuss the different statuses in more detail later):

```java
if (t.getState() == State.TERMINATED) //check if thread’s status is terminated
```

## Threads' exceptions

What if a worker thread throws an
exception?

- Exception is (usually) shown on console
- Behaviour of `thread.join()` is unaffected
- → Main thread may not be aware of an exception inside a worker thread

Implementing `UncaughtExceptionHandler` interface allows us to
handle unchecked exceptions
Three options:

- Register exception handler with `Thread` object
- Register exception handler with `ThreadGroup` object
- Use `setDefaultUncaughtExceptionHandler()` to register handler for
  all threads

Handler can then record which threads terminated exceptionally, or restart them, or ...

## Preview: Threads Safety Hazard

**Thread safety**

- implies program safety
- typically refers to “nothing bad ever happens”, in any possible interleaving (a safety property)

Examples of **safety properties we** will encounter in this course include:

- absence of data races
- mutual exclusion
- linearizability
- atomicity
- schedule-deterministic
- absence of deadlock
- custom invariants (e.g., age > 15)

**Thread safety**: “nothing bad happens”

**Liveness**: “eventually something good happens”, progress *will* be made at some point.

## Locks: preview

In Java, all objects have an internal lock, called intrinsic lock or monitor lock, which are used to implement synchronized.

Java also offers external locks (e.g. in package `java.util.concurrent.locks` )

- Less easy to use
- But support more sophisticated locking idioms, e.g. for reader-writer
  scenarios

## Synchronized Methods

```java
// synchronized method: locks on "this" object
public synchronized type name(parameters) { ... }
// synchronized static method: locks on the given class
public static synchronized type name(parameters) { ... }
```

### Examples: Synchronization with different locks

Allow more concurrency:

```java
public class TwoCounters {
    private long c1 = 0, c2 = 0;
    private Object lock1 = new Object();
    private Object lock2 = new Object();
    public void inc1() {
        synchronized(lock1) {
        	c1++;
        }
    }
    public void inc2() {
        synchronized(lock2) {
        	c2++;
        }
    }
}
```

### Examples: Synchronization with static methods

```java
public class Screen {
    private static Screen theScreen;
    private Screen(){...}
    public static synchronized getScreen() { // locks the entire class Screen!
        if (theScreen == null) {
            theScreen = new Screen();
        }
        return theScreen;
    }
}
```



**`wait()`** releases object lock, thread waits on internal queue

**`notify()`** wakes the highest-priority thread closest to front of object’s internal queue

**`notifyAll()`** wakes up all waiting threads:

- Threads non-deterministically compete for access to object
- May not be fair (low-priority threads may never get access)

May only be called when object is locked (e.g. inside `synchronize`)

## Thread States: Summary

Thread is created when an object derived from the Thread class is created. At this point, the thread is not
executable, it is in a **new** state.

Once the `start` method is called, the thread becomes eligible for execution by the scheduler.

If the thread calls the `wait` method in an Object, or calls the `join` method in another thread object, the
thread becomes **not runnable** and no longer eligible for execution.

It becomes executable as a result of an associated `notify` method being called by another thread, or if the
thread with which it has requested a join, becomes **terminated**.

A thread enters the **terminated** state, either as a result of the run method exiting (normally, or as a result of
an unhandled exception) or because its destroy method has been called.

In the latter case, the thread is abruptly moved to the **terminated** state and does not have the opportunity to
execute any finally clauses associated with its execution; it may leave other objects locked.

## Parallel performance 

Sequential execution time: $T 1$
Execution time $T_p$ on p CPUs

- $T_p = \frac{T_1}{p}$ (perfection)
- $T_p > \frac{T_1}{p}$ (performance loss, what normally happens)
- $T_p < \frac{T_1}{p}$ (sorcery!)

(parallel) speedup S p on p CPUs: $S p = T_1 / T_p$

- $S_p = p$ → linear speedup (perfection)
- $S_p < p$ → sub-linear speedup (performance loss)
- $S_p > p$ → super-linear speedup (sorcery!)

Efficiency: $\frac{S_1}{p}$

### Amdahl's Law

> ...the effort expended on achieving high parallel processing rates is
> wasted unless it is accompanied by achievements in sequential
> processing rates of very nearly the same magnitude.
>
> — Gene Amdahl

Execution time $T_1$ of a program falls into two categories:

- Time spent doing non-parallelizable serial work
- Time spent doing parallelizable work

Call these $W_{ser}$ and $W_{par}$ respectively.

Sequential:  $T_1 = W_{ser}+W_{par}$ 

Parallel:  $T_p \geq W_{ser}+\frac{W_{par}}{p}$ 

$\implies S_p\leq\frac{W_{ser}+W_{par}}{W_{ser}+\frac{W_{par}}{p}}$

Which gives the following corollary:

Assuming $f$ is the non-parallelizable serial fractions of the total work, then:
$$
W_{ser} = fT_1\\
W_{par}=(1-f)T_1\\
\implies S_p\leq\frac{1}{f+\frac{1-f}{p}}
$$
Infinite workers: $S_\infty \leq \frac{1}{f}$

### Gustafson's Law

consider problem size

- run-time, not problem size, is constant
- more processors allows to solve larger problems in the same time
- parallel part of a program scales with the problem size

$$
W = p(1-f)T_{wall} + fT_{wall}\\
\begin{align}\\
S_p&=f+p(1-f)\\
&=p-f(p-1)
\end{align}
$$

$T_{wall}$ constant (available time)

## Java's executor service

<img src="/home/plaf2000/.config/Typora/typora-user-images/image-20210618022824331.png" alt="image-20210618022824331" style="zoom:33%;" />

Future will contain the result of the task once it will be completed.

Interface `Runnable`:
→ `void run() `Does not return result
Interface` Callable<T>`:
→ `T call()` Returns result

## Task parallelism: performance model

$T_\infty$ is the span, critical path

- Time it takes on infinite processors

- longest path from root to sink

  <img src="/home/plaf2000/.config/Typora/typora-user-images/image-20210618025450581.png" alt="image-20210618025450581" style="zoom:33%;" />

  

$\frac{T_1}{T_\infty}$ → parallelism

- “wider” is better

Lower bounds:

- $T_p\geq \frac{T_1}{p}$
- $T_p\geq T_\infty$

### Scheduling of task graphs

Scheduler is an algorithm for assigning tasks to processors

- $T_p$ depends on scheduler
- $\frac{T_1}{p}$ and $T_\infty$ are fixed

a bound on how fast you can get on p processors
with a greedy scheduler: $T_p\leq \frac{T_1}{p}+ T_\infty$

### Work stealing scheduler

Every processor has a queue of works. If a processor has no work, then it steals it from other processor (if available).

Provably: $T_p=\frac{T_1}{P}+\mathcal{O}(T_\infty)$

Empirically: $T_p=\frac{T_1}{P}+\mathcal{O}(T_\infty)$

## ForkJoin Java Framework

<img src="/home/plaf2000/.config/Typora/typora-user-images/image-20210618170903371.png" alt="image-20210618170903371" style="zoom:70%;" />

<img src="/home/plaf2000/.config/Typora/typora-user-images/image-20210618171442631.png" alt="image-20210618171442631" style="zoom:70%;" />

An implementation for summing up all the numbers in two arrays:

```java
class VecAdd extends RecursiveAction  {

	public static final int SEQUENTIAL_THRESHOLD = 1;
	private int lo, hi;
	private int[] arr1, arr2, res;

	VecAdd(int[] arr1, int[] arr2, int lo, int hi, int[] res) {
		this.lo = lo;
		this.hi = hi;
		this.arr1 = arr1;
		this.arr2 = arr2;
		this.res = res;
	 }

	 public void compute() {
			if (hi - lo <= SEQUENTIAL_THRESHOLD) {
				for (int i = lo; i < hi; i++) {
					res[i] = arr1[i] + arr2[i];
				}
			} 
			else {
				int mid = (lo + hi) / 2;
				VecAdd left = new VecAdd(arr1, arr2, lo, mid, res);
				VecAdd right = new VecAdd(arr1, arr2, mid, hi, res);
				
				left.fork();
				right.compute();
				left.join();
                
               	// ^ better version (instead of forking both left and right)
			}
		}
}

public class Main {

	private static final ForkJoinPool fjPool = new ForkJoinPool();
	
	public static void main(String[] args) throws Exception {
	  	  
	  int[] myIntArray1 = new int[8];
	  int[] myIntArray2 = new int[8];
	  
	  assert (myIntArray1.length == myIntArray2.length);
	  
	  int[] ans = new int[myIntArray1.length];
	    
	  for(int i=0; i<myIntArray1.length; i++)
		  myIntArray1[i] = myIntArray2[i] = 1;
	  
	  fjPool.invoke(new VecAdd(myIntArray1, myIntArray2, 0, ans.length, ans));
	  
	  System.out.print("answer array: ");
	  for(int i=0; i<ans.length; i++)
		  System.out.print(ans[i] + " ");
	}
	
}
```

### Getting good results in practice

Sequential threshold

- Library documentation recommends doing approximately 100-5000 basic operations in each “piece” of your algorithm

Library needs to “warm up”

- May see slow results before the Java virtual machine re-optimizes the library
  internals
- Put your computations in a loop to see the “long-term benefit”

Wait until your computer has more processors :smile: 

- Seriously, overhead may dominate at 4 processors, but parallel programming is likely to become much more important

Beware memory-hierarchy issues

- E.g. locality.

### Managing state

**Main challenge for parallel programs**

Approaches:

- immutability
  - data do not change
  - best option, should be used when possible
- isolated mutability
  - data can change, but only one thread/task can access them
- mutable/shared data
  - data can change / all tasks/threads can potentially access them

## Mutual exclusion

Mutual exclusion: One thread using a resource means another thread must wait

- a.k.a. critical sections, which technically have other requirements

## Re-entrant lock

A re-entrant lock (a.k.a. recursive lock) 

- “remembers” the thread (if any) that currently holds it
- a count

## Races

Roughly: a **race condition** occurs when the computation result depends on the scheduling (how
threads are interleaved)

This lecture makes a big distinction between **data races and** bad **interleavings**, both kinds
of race-condition bugs

- Confusion often results from not distinguishing these or using the ambiguous “race condition” to
  mean only one

### The distinction

**Data Race** [aka Low Level Race Condition, low semantic level]

Erroneous program behavior caused by insufficiently synchronized accesses of a shared resource by multiple threads, e.g. Simultaneous read/write or write/write of the same memory location

- (for mortals) always an error, due to compiler & HW
- Original `peek` example has no data races

**Bad Interleaving** [aka High Level Race Condition, high semantic level]

Erroneous program behavior caused by an unfavorable execution order of a multithreaded algorithm that makes use of otherwise well synchronized resources.
	“Bad” depends on your specification
	Original `peek` had several

## Interleavings

Possible interleaving with k statements and 2 threads:
$$
\binom{2k}{k} = \mathcal{O}\binom{4^n}{\sqrt{2n}}
$$

### Memory reording

**Rule of thumb**: Compiler and hardware allowed to make changes that do not affect the *semantics* of a ***sequentially*** executed program.

## Memory model

A memory model (e.g., of a programming language like Java) provides (often minimal) guarantees for the
effects of memory operations.

- leaving open optimization possibilities for hardware and compiler
- but including guidelines for writing correct multithreaded programs

### Program Order (PO)

Program order is a total order of intra-thread actions

- Program statements are NOT a total order across threads!

Program order does not provide an ordering guarantee for memory accesses!

- The only reason it exists is to provide the link between possible executions and the original program.

Intra-thread consistency: Per thread, the PO order is consistent with the threads isolated execution

### Synchronization Actions (SA)

Synchronization actions are:

- Read/write of a volatile variable
- Lock monitor, unlock monitor
- First/last action of a thread (synthetic)
- Actions which start a thread
- Actions which determine if a thread has terminated

### Synchronization Actions form the Synchronization Order (SO)

- SO is a total order
- All threads see SA in the same order
- SA within a thread are in PO
- SO is consistent: all reads in SO see the last writes in SO

### Synchronizes-With (SW)

- SW only pairs the specific actions which "see" each other
  - For example, two threads that read and write to the same volatile variable :arrow_down:
- A volatile write to x synchronizes with subsequent read of x (subsequent in SO)

### Happens-Before (HB) orders

- The transitive closure of PO and SW forms HB
- HB consistency: When reading a variable, we see either the last write (in HB) or any other unordered write.
  - This means races are allowed!

## Critical sections

Pieces of code with the following conditions
1. Mutual exclusion: statements from critical sections of two or more processes must not be interleaved
2. Freedom from deadlock: if some processes are trying to enter a critical section then one of them must
eventually succeed
3. Freedom from starvation: if any process tries to enter its critical section, then that process must
  eventually succeed