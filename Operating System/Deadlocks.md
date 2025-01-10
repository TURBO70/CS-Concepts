### Common Concurrency Problems

Concurrency introduces complexities in code, leading to two types of bugs:

1. **Non-Deadlock Bugs**
    - These include **atomicity violation bugs** and **order violation bugs**.

#### **Atomicity Violation Bugs**

- **Problem:** Occurs when multiple memory accesses intended to be atomic are interrupted.

Example :

```
// Thread 1
if (thd->proc_info) {
    fputs(thd->proc_info, ...);
}

// Thread 2
thd->proc_info = NULL;

```

- If Thread 1 checks the pointer and gets interrupted, Thread 2 can set it to `NULL`, causing Thread 1 to crash when it resumes.
    
- **Fix:** Use locks to enforce atomicity:

```
pthread_mutex_t proc_info_lock = PTHREAD_MUTEX_INITIALIZER;

// Thread 1
pthread_mutex_lock(&proc_info_lock);
if (thd->proc_info) {
    fputs(thd->proc_info, ...);
}
pthread_mutex_unlock(&proc_info_lock);

// Thread 2
pthread_mutex_lock(&proc_info_lock);
thd->proc_info = NULL;
pthread_mutex_unlock(&proc_info_lock);

```


#### **Order Violation Bugs**

- **Problem:** Occurs when the execution order of operations is not guaranteed.
    
- **Example:**
```
// Thread 1
void init() {
    mThread = PR_CreateThread(mMain, ...);
}

// Thread 2
void mMain(...) {
    mState = mThread->State; // Assumes mThread is initialized
}

```


- If Thread 2 runs before Thread 1 finishes, `mThread` may be `NULL`.
    
- **Fix:** Use condition variables to enforce order:

```
pthread_mutex_t mtLock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t mtCond = PTHREAD_COND_INITIALIZER;
int mtInit = 0;

// Thread 1
void init() {
    mThread = PR_CreateThread(mMain, ...);
    pthread_mutex_lock(&mtLock);
    mtInit = 1;
    pthread_cond_signal(&mtCond);
    pthread_mutex_unlock(&mtLock);
}

// Thread 2
void mMain(...) {
    pthread_mutex_lock(&mtLock);
    while (mtInit == 0) {
        pthread_cond_wait(&mtCond, &mtLock);
    }
    pthread_mutex_unlock(&mtLock);
    mState = mThread->State;
}

```


2. **Deadlock Bugs**
    - Occur when threads are waiting for each other’s resources indefinitely.

#### **Why Deadlocks Occur**

- **Complex Dependencies:** Components depend on each other cyclically, such as virtual memory needing the file system, and vice versa.
- **Encapsulation Challenges:** Encapsulation hides locking logic, making lock order unpredictable.

#### **Conditions for Deadlock**

1. **Mutual Exclusion:** Resources can only be held by one thread at a time.
2. **Hold-and-Wait:** Threads hold resources while waiting for others.
3. **No Preemption:** Locks cannot be forcibly taken from threads.
4. **Circular Wait:** A cycle exists where threads wait on each other.

#### **Techniques to Prevent Deadlock**

- **Circular Wait Prevention:**  
    Enforce a strict lock acquisition order.  
    Example: Always acquire `L1` before `L2`.
    
- **Hold-and-Wait Prevention:**  
    Acquire all required locks at once.
```
pthread_mutex_lock(prevention_lock);
pthread_mutex_lock(L1);
pthread_mutex_lock(L2);
pthread_mutex_unlock(prevention_lock);

```

**No Preemption:**  
Use `pthread_mutex_trylock()` to avoid indefinite waiting:

```
top:
    pthread_mutex_lock(L1);
    if (pthread_mutex_trylock(L2) != 0) {
        pthread_mutex_unlock(L1);
        goto top;
    }

```

**Avoid Mutual Exclusion:**  
Use lock-free techniques like `CompareAndSwap()`:

```
int CompareAndSwap(int *address, int expected, int new) {
    if (*address == expected) {
        *address = new;
        return 1; // success
    }
    return 0; // failure
}


```

#### **Deadlock Avoidance via Scheduling**

- Use global knowledge of thread-lock interactions to avoid deadlocks by scheduling threads appropriately.
- Example:

```
T1 grabs L1, L2
T2 grabs L1, L2
T3 grabs L2
T4 grabs none

```
