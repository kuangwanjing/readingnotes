## Locks

### Background

*Problem to solve: attain instruction atomicity while having concurrency.*

Solution: lock up the **critical sections**.   

```C
lock_t mutex;
...
lock(&mutex);
// do something critically
unlock(&mutex);
```

Lock is either unoccupied or acquired. Lock provides control over the scheduling of threads for programmers.

Lock — Mutal Exclusion 

Coarse-grained and fine-grained: fine-grained lock enable more concurrency. 

---

### Build a Lock

With the help of hardware primitives and OS. 

**Evaluation of Locks**

1. Correctness: does the lock provide mutal exclusion, preventing multiple threads entering the critical section concurrently. 
2. Fairness: contend for the lock fairly and no starvation(extreme case). 
3. Performance: time overhead added by using the lock. Cases: single thread; threads on single CPU; threads on multiple CPU.

#### Implementations

**Interruption:** 

```C
void lock() {
  DisableInterrupts();
}
void unlock() {
  EnableInterrupts();
}
```

Analysis:

1. Correctness: works on single-processor system but controlling interrupt is a privileged operation which enable process to monopolize the processor. OS would never regain the control by malicious software.  not work on multiple-processor system: even if the interrupt is disabled on a processor, other threads can still enter the critical section with other processor. This also lead to the delayed response of OS to some critical system event(like I/O).
2. Fairness: — 
3. Performance: too much overhead. 

**Loads and Stores:** *(Failed)*

Basic idea: use a single variable and accessing it via loads and stores. 

```C
typedef struct __lock_t { int flag; } lock_t;
void init(lock_t *mutex) {
  mutex->flag = 0; // just one single flag 
}
void lock(lock_t *mutex) {
  while (mutex->flag == 1) // test the flag
    ; // spin-wait and do nothing 
  mutex->flag = 1; // then set the flag
}
void unlock(lock_t *mutex) {
  mutex->flag = 0;
}
```

Analysis:

1. Correctness: not correct due to interrupts 
2. Fairness:
3. Performance: the spin-waiting wastes the time slice of the thread because it does nothing. 

*The following solutions try to solve the above two problems.* 

---

**Spin Locks with Test-And-Set**

Use atomic instruction test-and-set to achieve atomic exchange. 

```C
int TestAndSet(int *old_ptr, int new) { // hardware instruction
  int old = old_ptr;
  *old_ptr = new; // exchange value
  return old; // this tells whether the exchange takes place. 
}
```

```C
typedef struct __lock_t { int flag } lock_t;
void init(lock_t *mutex) {
  mutex->flag = 0;
}
void lock(lock_t *mutex) {
  while (TestAndSet(&mutext->flag, 1) == 1)
    ; // it requires a preemptive scheduler on single CPU otherwise its performance is bad.  
  // since exchange take places in the loop, it is not neccessary to set flag be 1 like the last implementation.
}
void unlock(lock_t *mutex) {
  mutex->flag = 0;
}
```

Analysis:

1. Correctness: yes
2. Fairness: spin lock doesn't guarantee fairness! 
3. Performance: bad on single CPU but works effectively on multiple CPU if the critical section is short enough. 

**Compare-And-Swap**

Another hardware primitive:

```c
int CompareAndSwap(int *ptr, int expected, int new) {
  int actual = *ptr;
  if (actual == expected) { // TestAndSet directly set the value without examining the value
    *ptr = new;
  }
  return actual;
}
```

```c
typedef struct __lock_t { int flag } lock_t;
void init(lock_t *mutex) {
  mutex->flag = 0;
}
void lock(lock_t *mutex) {
  while (CompareAndSwap(&mutext->flag, 0, 1) == 1)
    ;   
  // pretty similar to TestAndSet
}
void unlock(lock_t *mutex) {
  mutex->flag = 0;
}
```

Analysis: (similar to TestAndSet)

**Load-Linked and Store-Conditional**

**Fetch-And-Add**

---

**Yield** 

Thread relinquishes the CPU once it fails to enter the critical section.  Primitive: yield(); 

```C
// based on the work of TestAndSet
typedef struct __lock_t { int flag } lock_t;
void init(lock_t *mutex) {
  mutex->flag = 0;
}
void lock(lock_t *mutex) {
  while (TestAndSet(&mutext->flag, 1) == 1)
    yield();
}
void unlock(lock_t *mutex) {
  mutex->flag = 0;
}
```

yield() puts the thread into ready state and the thread would be rescheduled later. 

Analysis:

1. 

2. 
3. Performance: if the scheduler is in Round-Robin style, and the scheduling order may leads to many context switches on thread level. This overhead could be very substantial. 

**Sleeping instead of Spinning with Queues**

Primitives: park() and unpark(ThreadID)

```C
typedef struct __lock_t {
  int flag; // the state of a lock.
  int guard; // why use guard? Suppose we use flag to test and set, the solution degraded to spin-lock. Therefore, use a guard value to guard the operation to the lock. Since the critical section for the lock operations are tiny, the spin wait won't take long. 
  queue_t *q; // queue of threads
}
void lock_init(lock_t *m) {
  m->flag = 0;
  m->guard = 0;
  queue_init(m->q);
}
void lock(lock_t *m) {
  while (TestAndSet(m->guard, 1) == 1) 
    ;
  if (m->flag == 0) { // the lock is not held.
    m->flag = 1;
    m->guard = 0;
  } else { // otherwise the lock is held by other thread, put the current thread into queue and give up the CPU. 
    queue_add(m->q, gettid());
    set_park(); // !!! avoid race condition
    m->guard = 0; // this line can't be moved to the position next to park() otherwise deadlock to the lock. 
    park(); // <- when the thread is waken up, PC is at this point. 
  }
}
void unlock(lock_t *m) {
  while (TestAndSet(m->guard, 1) == 1) 
    ;
  if (queue_empty(m->q)) // this can be a race condition of park() in lock() leading the other thread fall into infinite sleep. so setpark() before releasing the guard. 
    m->flag = 0;
  else
    unpark(queue_remove(m->q)); 
  // the m->flag is not released because the waken thread does not hold the guard, so it is not safe for it to update the state of lock. Remember where the PC is. 
  m->guard = 0;
}
```

