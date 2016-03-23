# ch02b
PROCESSES & THREADS II: INTERPROCESS COMMUNICATION & SYNCHRONIZATION

## WHY DO WE NEED IPC?
- Each process operates sequentially
- All is fine until processes want to share data
    - Exchange data between multiple processes
    - Allow processes to navigate critical regions
    - Maintain proper sequencing of actions in multiple processes
- These issues apply to threads as well
    - Threads can share data easily (same address space)
    - Other two issues apply to threads


## EX: BOUNDED BUFFER PROBLEM
![fig01](ch02b/ch02b-fig01.png)


## PROBLEM: RACE CONDITIONS
![fig02](ch02b/ch02b-fig02.png)
- Cooperating processes share storage (memory)
- Both may read and write the shared memory
- Problem: can’t guarantee that read followed by write is atomic
    - Ordering matters!
- This can result in erroneous results!
- We need to eliminate race conditions...


## CRITICAL REGIONS
![fig03](ch02b/ch02b-fig03.png)￼
- Use critical regions to provide mutual exclusion and help fix race conditions
- Four conditions to provide mutual exclusion
    - No two processes simultaneously in critical region
    - No assumptions made about speeds or number of CPUs
    - No process running outside its critical region may block another process
    - A process may not wait forever to enter its critical region


## BUSY WAITING: STRICT ALTERNATION
![fig04](ch02b/ch02b-fig04.png)￼
- Use a shared variable (turn) to keep track of whose turn it is
- Waiting process continually reads the variable to see if it can proceed
    - This is called a spin lock because the waiting process “spins” in a tight loop reading the variable
- Avoids race conditions, but doesn’t satisfy criterion 3 for critical regions


## BUSY WAITING: WORKING SOLUTION
```c
#define FALSE 0
#define TRUE 1
#define N 2			// # of processes
int turn;			// Whose turn is it?
int interested[N];	// Set to 1 if process j is interested

void enter_region(int process) {
	int other = 1 - process;	// # of the other process
	interested[process] = TRUE;
	turn = process;				// show interest
	while(turn==process && interested[other]==TRUE)
		;	// Wait wile the other process runs
}

void leave_region (int process) {
	interested[process] = FALSE;	// I'm no longer interested
}
```
-only you could deal with two processes


## BAKERY ALGORITHM FOR MANY PROCESSES
- Notation used
    - <<< is lexicographical order on (ticket#, process ID)
    - (a,b) <<< (c,d) if (a <<< c) or ((a==c) and (b<d))
    - Max(a0,a1,…,an-1) is a number k such that k ≥ ai for all i
- Shared data
    - choosing initialized to 0
    - number initialized to 0
```c
int n;				// # of processes
int choosing[n];
int number[n];
```


## BAKERY ALGORITHM: CODE
```c
while(1) {  // i is the number of the current process
	choosing[i] = 1;
	number[i] = max(number[0], number[1], ..., number[n-1]) + 1;
	choosing[i] = 0;
	for (j=0; j<n; j++) {
		while(  choosing[j]  ) 	// wait while j is choosing a
			;	// number
		// wait while j wants to enter and j <<< i
		while(  (number[j] != 0) &&
				((number[j] < number[i]) ||
				((number[j]== number[i]) && (j<i))))
			;
	}
	// critical section
	number[i]=0;
	// rest of code
}
```


## HARDWARE FOR SYNCHRONIZATION
- Prior methods work, but...
    - May be somewhat complex
    - Require busy waiting: process spins in a loop waiting for something to happen, wasting CPU time
- Solution: use hardware
- Several hardware methods
    - Test & set: test a variable and set it in one instruction
    - Atomic swap: switch register & memory in one instruction
    - Turn off interrupts: process won’t be switched out unless it asks to be suspended


## MUTUAL EXCLUSION USING HARDWARE
![fig05](ch02b/ch02b-fig05.png)￼￼
- Single shared variable lock
- Still requires busy waiting, but code is much simpler
- Two versions
    - Test and set
    - Swap
- Works for any number of processes
- Possible problem with requirements
    - Non-concurrent code can lead to unbounded waiting


## ELIMINATING BUSY WAITING
- Problem: previous solutions waste CPU time
    - Both hardware and software solutions require spin locks
    - Allow processes to sleep while they wait to execute their critical sections
- Problem: priority inversion (higher priority process waits for lower priority process)
- Solution: use semaphores
    - Synchronization mechanism that doesn’t require busy waiting during entire critical section
- Implementation
    - Semaphore S accessed by two atomic operations
        - Down(S): while (S<=0) {}; S-= 1;
        - Up(S): S+=1;
    - Down() is another name for P()
    - Up() is another name for V()
    - Modify implementation to eliminate busy wait from Down()


## CRITICAL SECTIONS USING SEMAPHORES
![fig06](ch02b/ch02b-fig06.png)￼
- Define a class called Semaphore
    - Class allows more complex implementations for semaphores
    - Details hidden from processes
- Code for individual process is simple


## IMPLEMENTING SEMAPHORES WITH BLOCKING
- Assume two operations:
    - Sleep(): suspends current process
    - Wakeup(P): allows process P to resume execution
- Semaphore is a class
    - Track value of semaphore
    - Keep a list of processes waiting for the semaphore
- Operations still atomic



## SEMAPHORES FOR GENERAL SYNCHRONIZATION
![fig08](ch02b/ch02b-fig08.png)￼
- We want to execute $$B$$ in $$P1$$ only after $$A$$ executes in $$P0$$
- Use a semaphore initialized to $$0$$
- Use `up()` to notify $$P1$$ at the appropriate time
￼


## TYPES OF SEMAPHORES
- Two different types of semaphores
    - Counting semaphores
    - Binary semaphores
- Counting semaphore
    - Value can range over an unrestricted range
- Binary semaphore
    - Only two values possible
        - $$1$$ means the semaphore is available
        - $$0$$ means a process has acquired the semaphore
    - May be simpler to implement
- Possible to implement one type using the other


## MONITORS
- A monitor is another kind of high-level synchronization primitive
    - One monitor has multiple entry points
    - Only one process may be in the monitor at any time
    - Enforces mutual exclusion—less chance for programming errors
- Monitors provided by high-level language
    - Variables belonging to monitor are protected from simultaneous access
    - Procedures in monitor are guaranteed to have mutual exclusion
- Monitor implementation
    - Language / compiler handles implementation
    - Can be implemented using semaphores


## MONITOR USAGE
![fig07](ch02b/ch02b-fig07.png)￼
- This looks like C++ code, but it’s not supported by C++
- Provides the following features:
    - Variables foo, bar, and arr are accessible only by proc1 & proc2
    - Only one process can be executing in either proc1 or proc2 at any time
￼

## CONDITION VARIABLES IN MONITORS
- Problem: how can a process wait inside a monitor?
    - Can’t simply sleep: there’s no way for anyone else to enter
    - Solution: use a condition variable
- Condition variables support two operations
    - `Wait()`: suspend this process until signaled
    - `Signal()`: wake up exactly one process waiting on this condition variable
        - If no process is waiting, signal has no effect
        - Signals on condition variables aren’t “saved up”
- Condition variables are only usable within monitors
    - Process must be in monitor to signal on a condition variable
    - Question: which process gets the monitor after `Signal()`?


## MONITOR SEMANTICS
- Problem: P signals on condition variable X, waking Q
    - Both can’t be active in the monitor at the same time
    - Which one continues first?
- Mesa semantics
    - Signaling process (P) continues first
    - Q resumes when P leaves the monitor
    - Seems more logical: why suspend P when it signals?
- Hoare semantics
    - Awakened process (Q) continues first
    - P resumes when Q leaves the monitor
    - May be better: condition that Q wanted may no longer hold when P leaves the monitor


## LOCKS & CONDITION VARIABLES
- Monitors require native language support
- Provide monitor support using special data types and procedures
    - Locks (Acquire(), Release())
    - Condition variables (Wait(), Signal())
- Lock usage
    - Acquiring a lock == entering a monitor\
    - Releasing a lock == leaving a monitor
- Condition variable usage
    - Each condition variable is associated with exactly one lock
    - Lock must be held to use condition variable
    - Waiting on a condition variable releases the lock implicitly
    - Returning from Wait() on a condition variable reacquires the lock


## IMPLEMENTING LOCKS WITH SEMAPHORES
![fig08](ch02b/ch02b-fig08.png)￼￼
- Use mutex to ensure exclusion within the lock bounds
- Use next to give lock to processes with a higher priority (why?)
- nextCount indicates whether there are any higher priority waiters


## IMPLEMENTING CONDITION VARIABLES
![fig09](ch02b/ch02b-fig09.png)￼￼￼
- Are these Hoare or Mesa semantics?
- Can there be multiple condition variables for a single Lock?


## MESSAGE PASSING
- Synchronize by exchanging messages
- Two primitives:
    - Send: send a message
    - Receive: receive a message
    - Both may specify a “channel” to use
- Issue: how does the sender know the receiver got the message?
- Issue: authentication


## BARRIER
![fig10](ch02b/ch02b-fig10.png)￼￼￼
- Used for synchronizing multiple processes
- Processes wait at a “barrier” until all in the group arrive
- After all have arrived, all processes can proceed
- May be implemented using locks and condition variables



## IMPLEMENTING BARRIERS USING SEMAPHORES
```c
Barrier b;
b.bsem.value = 0;
b.mutex.value = 1;
b.waiting = 0;
b.maxproc = n;

HitBarrier(Barrier *b) {
	SemDown (&b->mutex);
	if (++b->waiting >= b->maxproc) {
		while(--b->waiting > 0) {
			SemUp(&b->bsem);
		}
		SemUp(&b->mutex);
	} else {
		SemUp(&b->mutex);
		SemDown(&b->bsem);
	}
}
```￼


## DEADLOCK AND STARVATION
![fig11](ch02b/ch02b-fig11.png)￼￼￼
- Deadlock: two or more processes are waiting indefinitely for an event that can only by caused by a waiting process
    - P0 gets A, needs B
    - P1 gets B, needs A
    - Each process waiting for the other to signal
- Starvation: indefinite blocking
    - Process is never removed from the semaphore queue in which its suspended
    - May be caused by ordering in queues (priority)



## CLASSICAL SYNCHRONIZATION PROBLEMS
- #### Bounded Buffer
    - Multiple producers and consumers
    - Synchronize access to shared buffer
- #### Readers & Writers
    - Many processes that may read and/or write
    - Only one writer allowed at any time
    - Many readers allowed, but not while a process is writing
- #### Dining Philosophers
    - Resource allocation problem
    - N processes and limited resources to perform sequence of tasks
- Goal: use semaphores to implement solutions to these problems


## BOUNDED BUFFER PROBLEM
![fig12](ch02b/ch02b-fig12.png)￼￼
Goal: implement producer-consumer without busy waiting
￼


## READERS-WRITERS PROBLEM
![fig13](ch02b/ch02b-fig13.png)


## DINING PHILOSOPHERS
![fig14](ch02b/ch02b-fig14.png)￼￼
- $$N$$ philosophers around a table
    - All are hungry
    - All like to think
- $$N$$ chopsticks available
    - $$1$$ between each pair of philosophers
- Philosophers need two chopsticks to eat
- Philosophers alternate between eating and thinking
- Goal: coordinate use of chopsticks


## DINING PHILOSOPHERS: SOLUTION 1
![fig15](ch02b/ch02b-fig15.png)￼￼￼
- Use a semaphore for each chopstick
- A hungry philosopher
    - Gets the chopstick to his right
    - Gets the chopstick to his left
    - Eats
    - Puts down the chopsticks
- Potential problems?
    - Deadlock
    - Fairness



## DINING PHILOSOPHERS: SOLUTION 2
![fig16](ch02b/ch02b-fig16.png)￼￼￼
- Use a semaphore for each chopstick
- A hungry philosopher
    - Gets lower, then higher numbered chopstick
    - Eats
    - Puts down the chopsticks
- Potential problems?
    - Deadlock
    - Fairness


## DINING PHILOSOPHERS WITH LOCKS
![fig17](ch02b/ch02b-fig17.png)￼￼￼


## THE SLEEPY BARBER PROBLEM
- Barber wants to sleep all day
    - Wakes up to cut hair
- Customers wait in chairs until barber chair is free
    - Limited space in the waiting room
    - Leave if no space free
- Write the synchronization code for this problem


## CODE FOR THE SLEEPY BARBER PROBLEM
![fig18](ch02b/ch02b-fig18.png)

￼
