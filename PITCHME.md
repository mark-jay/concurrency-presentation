---

## Concurrency, jvm, approaches

```
```

---

## Plan

```text
- Part1. Volatile, when and why. Happens-before
- Part2. Blocking and non-blocking algos. Compare-and-swap instruction. Context switch. Thread lifecycle
- Part3. Different approaches. Thread per request/futures/actor-systems

```

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

    public static void main(String[] args) { // but hard to show:(
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

---

### JMM. happens-before

![Image](https://www.baeldung.com/wp-content/uploads/2017/08/happens-before.png)

Transitive

---

### JMM. happens-before. examples

```text
1. Monitor releasing happens-before acquiring the same monitor.
2. Write to a volatile variable happens-before reading from that var.
3. Writing to a final-field during object construction happens-before
   writing this object to any variable
   (writing that happens outside of this ctor).
4. Atomic* classes. The memory effects for accesses and updates of
   atomics generally follow the rules for volatiles
```

---

### Part 2. Non blocking and blocking algo.

```text
compare and swap:

 - parameters: a pointer, an old value, a new value
 - tries to write new value into the pointer
 - returns true if succeeeded, false otherwise
```

---

### Example. Requirements

```
 - AtomicBoolean - ok
 - AtomicInteger - ok
 - AtomicReference - ok
 - AtomicBigDecimal.incrementAndGet() - ???
```

---

### Blocking example

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

### Non blocking example

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

### Implementing mutex

```java
    boolean lock = false
    ...
    while (!compareAndSwap(lock, false, true)) {
        Thread.yield();
    }
    // access acquired
    doStuff();

    // returning lock
    lock = false
```

---

### Thread life cycle

![Image](https://media.geeksforgeeks.org/wp-content/uploads/threadLifeCycle.jpg)

---

### Thread states

```text
 - NEW:
 Thread has not yet started.

 - RUNNABLE:
 Thread is currently running without blocking/waiting in its run method.

 - BLOCKED:
 Thread is blocked from entering a synchronized block/method, waiting for the monitor lock to be released by the other thread.

 - WAITING:
 Thread is waiting due to one of the these calls; Object.wait(), Thread.join() or LockSupport.park()

 - TIMED_WAITING:
 Thread is waiting due to one of these timeout based method calls; Thread.sleep(long millis), Object.wait(long millis), Thread.join(long millis), LockSupport.parkNanos(Object blocker, long nanos) or LockSupport.parkUntil(Object blocker, long nanos)

 - TERMINATED:
 A thread has exited from its run() method.

```

---

### Part 3. Approaches to hande concurrency

```text
1. One request = one thread
2. Futures + optional async/await syntax sugar
3. Actor system
```

---

### One request per thread

```text
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

```text
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

### Traditional approach

```scala
    case class WebServiceResult()
    case class DbWriteResult()

    // definition
    def callWebService(): WebServiceResult = ???
    def callDBWriterService(p: WebServiceResult): DbWriteResult = ???

    // usage
    val wsResult: WebServiceResult = callWebService()
    val dbWriteResult: DbWriteResult = callDBWriterService(wsResult)
    // or
    val r1: DbWriteResult = callDBWriterService(callWebService())

```

---

### Future chaining

```scala
    // definition
    def callWebServiceAsync(): Future[WebServiceResult] = ???
    def callDBWriterServiceAsync(webServiceResult: WebServiceResult):
        Future[DbWriteResult] = ???

    // usage
    val wsResultFuture: Future[WebServiceResult] =callWebServiceAsync()
    val dbWriteResultFuture: Future[DbWriteResult] =
      wsResultFuture.flatMap(wsRes => {
        callDBWriterServiceAsync(wsRes)
      })
```

---

### Callback hell

```scala
    def anotherFutureCall(dbWriteResult: DbWriteResult):
        Future[DbWriteResult] = ???

    val dbWriteResultFutureNested: Future[DbWriteResult] =
      callWebServiceAsync().flatMap(wsRes => {
        callDBWriterServiceAsync(wsRes).flatMap(anotherRes => {
          anotherFutureCall(anotherRes).flatMap(yetAnotherR => {
            anotherFutureCall(yetAnotherR).flatMap(r => {
              ??? // ...
            })

          })
        })
    })
```

---

### Monad sugar

```scala
    for {
      wsRes         <- callWebServiceAsync()
      anotherRes    <- callDBWriterServiceAsync(wsRes)
      yetAnotherRes <- anotherFutureCall(anotherRes)
      result        <- anotherFutureCall(yetAnotherRes)
    } {
      ??? // ...
    }
```

---

### Async Await

```javascript
async function sayHello(name) {
  return `Hello, ${name}`;
}
console.log(sayHello('Harry')); // returns "Promise { 'Hello, Harry' }"

async function test() {
  const greeting = await sayHello('Harrison');
  console.log(greeting);
}
```

---

### Actors system. Basics and principles


```text
 - actor = living instance of a class
 - actors can receive and send async messages to another actors
 - all messages processing happens one by one(one message processing happens before the next message)
 - single thread illusion
 - at-most-once semantics(unless the same JVM)
 - message ordering guarantees
 - no shared state, only message passing
 - no blocking operations
```


---

### Actors example


```scala
class Worker extends Actor {
  def receive = {
    case "hello" => println("hello back at you")
    case _       => println("huh?")
  }
}

class HelloActor extends Actor {
  val worker = context.actorOf(Props[Worker])
  def receive = {
    case message => worker.forward(message)
  }
}
object Main extends App {
  val system = ActorSystem("HelloSystem")
  // default Actor constructor
  val helloActor = system.actorOf(Props[HelloActor], name = "helloactor")
  helloActor ! "hello"
  helloActor ! "buenos dias"
}
```

---

### Actors. Further

```text
 - netty for io
 - akka streams
 - supervision
 - persistnce
 - scaling out
 - event sourcing support
 - custom dispatchers, mailboxes
 - hocon for application config
 - Up to 50 million msg/sec on a single machine.
 - Small memory footprint; ~2.5 million actors per GB of heap.
```

---

## The end;)


```
```
