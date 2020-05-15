---

### Part 1. A bit of baeldung

```java
public class TaskRunner {

    private static int number;
    private static boolean ready;

    private static class Reader extends Thread {

        @Override
        public void run() {
            while (!ready) {
                Thread.yield();
            }

            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new Reader().start();
        number = 42;
        ready = true;
    }
}
```

---

### A bit of baeldung

![image](https://www.baeldung.com/wp-content/uploads/2017/08/cpu.png)

```
- Out of Order Execution
- Branch Prediction
- Speculative Execution
- Visibility - Caching.(CPU flushing fallacy?)
```

---

### Timing

![image](https://4.bp.blogspot.com/-fvYQdIN_XRM/URy239FMHPI/AAAAAAAAAGs/Jkqa8T3gbTk/s1600/MemoryHeirarchy.png)

---?color=#FFFFFF

### Reproduction

```
 - На машинах с TSO (к коим относится x86) довольно сложно
   показать ломающий reordering.
```

---

### JMM. happens-before

![Image](https://www.baeldung.com/wp-content/uploads/2017/08/happens-before.png)

- Transitive

---

### JMM. happens-before. examples

```
1. Monitor releasing happens-before acquiring the same monitor.
2. Write to a volatile variable happens-before reading from that variable.
3. Writing to a final-field during object construction happens-before writing this object to any variable(writing that happens outside of this ctor).
4. Atomic* classes. The memory effects for accesses and updates of atomics generally follow the rules for volatiles
```

---

### Part 2. Non blocking and blocking algo

```
- Compare and swap
```

---

### Non blocking and blocking algo. Example

```
- AtomicBigDecimal.incrementAndGet()
```

---

### Non blocking and blocking algo. Example

```
    private BigDecimal value = new BigDecimal(0);

    public BigDecimal incrementAndGet() {
        synchronized (this) {
            value = value.add(1);
            return value;
        }
    }
```

---

### Non blocking and blocking algo. Example

```
    private final AtomicReference<BigDecimal> value = new AtomicReference<BigDecimal>(new BigDecimal(0));

    public BigDecimal incrementAndGet() {
        while (true) {
            BigDecimal current = value.get();
            BigDecimal next = current.add(BigDecimal.ONE);
            if (value.compareAndSet(current, next)) {
                return next;
            }
        }
    }
```

---

### Non blocking and blocking algo.

![Image](https://media.geeksforgeeks.org/wp-content/uploads/threadLifeCycle.jpg)

---

### Thread states

```
 - NEW:	           Thread has not yet started.
 - RUNNABLE:	   Thread is currently running without blocking/waiting in its run method.
 - BLOCKED:	       Thread is blocked from entering a synchronized block/method, waiting for the monitor lock to be released by the other thread.
 - WAITING:	       Thread is waiting due to one of the these calls; Object.wait(), Thread.join() or LockSupport.park()
 - TIMED_WAITING:  Thread is waiting due to one of these timeout based method calls; Thread.sleep(long millis), Object.wait(long millis), Thread.join(long millis), LockSupport.parkNanos(Object blocker, long nanos) or LockSupport.parkUntil(Object blocker, long nanos)
 - TERMINATED:     A thread has exited from its run() method.
```

---

### Part 3. Approaches to hande concurrency

```
1. One request = one thread
2. Futures + optional async/await syntax sugar
3. Actor system
```

---

### One request per thread

```
pros:
 - easy to read and maintain
 - allows ThreadLocal contexts(MDC, audit, etc)

cons:
 - expensive threads(1Mb?)
 - expensive context switch
```

---

### Futures. Interface

```
 - boolean cancel(boolean mayInterruptIfRunning);
 - boolean isCancelled();
 - boolean isDone();
 - V get() throws InterruptedException, ExecutionException;
 - V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
```

---

### Futures. Memory consistency effects

```
 - Actions in a thread prior to the submission of a Runnable or
   Callable task to an ExecutorService happen-before
 - any actions taken by that task, which in turn happen-before
 - the result is retrieved via Future.get().
```

---

### CompletableFuture(aka Promises)

```
 - final boolean completeValue(T t)
 - final boolean completeThrowable(Throwable x)
```

---

### TODO

```
   WebServiceResult wsRes = callWebService();

```

---

### TODO

---

### TODO

---

### TODO

---

### TODO

---

### TODO
