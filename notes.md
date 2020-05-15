

2. Futures + optional async/await syntax sugar
3. Actor system


###
--------------------------------------------------------------------------------
soures:
https://www.baeldung.com/java-volatile

https://www.geeksforgeeks.org/lifecycle-and-states-of-a-thread-in-java/

https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.State.html

http://jpbempel.blogspot.com/2012/10/volatile.html

https://www.logicbig.com/tutorials/core-java-tutorial/java-multi-threading/thread-states.html

А я начал со статьи которая показывает что не нужно @volatile почему-то.
https://www.oreilly.com/library/view/applied-akka-patterns/9781491934876/ch04.html

Начал гуглить и вот что я интересного вычитал, краткая выжимка:
1. Есть cpu-кэши и по дефолту в джава(без ключевого слова volatile) данные пишутся туда, иногда флашатся в основную память.
 1.1 Если поле помечено как volatile, то запись всегда флашит сразу в основную память, а чтение происходит всегда из основной памяти
2. Для оптимизации стейтменты могут выполнятся в произвольном порядке, но порядок должен прозводить результаты аналогичные порядку указанному в коде программы.(то есть a=1;b=2; поменять местами можно, а a=1;b=a+2 - нельзя)
3. Если существует happens-before связь между двумя событиями a и b, - это означает что во время выполения операций "b" и операций после-"b" будут видны результаты выполнения "a" и операций до "a".
4. Синхронизация на synchronized/locks устанавливает h-b связь между освобождением и последующим получением лока
5. Запись и последующее считывание из volatile поля устанавливает связь hp между записью и чтением
6. happens-before транзитивно
7. И, наконец, akka дает 2 важные гарантии:
7.1 The actor send rule: the send of the message to an actor happens before the receive of that message by the same actor.
7.2 The actor subsequent processing rule: processing of one message happens before processing of the next message by the same actor.

Сорсы:
https://dzone.com/articles/java-multi-threading-volatile-variables-happens-be-1
https://doc.akka.io/docs/akka/current/general/jmm.html
https://habr.com/ru/post/133981/
https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html

Выводы:
В акка не нужно помечать поля как volatile потому что данные и так будут свежимио из-за "The actor subsequent processing rule"
Вроде как практика подтверждает эту теорию.

PS:
Там оказывается даже написали:
In layman’s terms this means that changes to internal fields of the actor are visible when the next message is processed by that actor. So fields in your actor need not be volatile or equivalent.


--------------------------------------------------------------------------------

./volatile3/TaskRunner.java
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

    public static void main(String[] args) throws Exception {
        long start = new java.util.Date().getTime();
        Thread t = new Reader();
        t.start();
        number = 42;
        ready = true;
        t.join();
        System.out.println(new java.util.Date().getTime() - start);
    }
}
./volatile2/Main.java
import java.util.*;

public class Main {

    private String value = "";
    private boolean hasValue = false;

    public void produce(String value) {
        while (hasValue) {
            try {
                System.out.println("produce: has vaule = " + hasValue);
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Producing " + value + " as the next consumable");
        this.value = value;
        hasValue = true;
    }

    public String consume() {
        while (!hasValue) {
            try {
                System.out.println("consume: has vaule = " + hasValue);
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        String value = this.value;
        hasValue = false;
        System.out.println("Consumed " + value);
        return value;
    }

    public static void main(String[] args) throws Exception {
        Main producerConsumer = new Main();
        List<String> values = Arrays.asList("1", "2", "3", "4", "5", "6", "7", "8",
                                            "9", "10", "11", "12", "13");
        Thread writerThread = new Thread(() -> values.stream()
                                         .forEach(producerConsumer::produce));
        Thread readerThread = new Thread(() -> {
                for (int i = 0; i < values.size(); i++) {
                    producerConsumer.consume();
                }
        });
        writerThread.start();
        readerThread.start();
        writerThread.join();
        readerThread.join();
    }
}
./volatile1/Main.java

import java.util.Date;
import java.util.ArrayList;

class Main {

    private int a = 0;

    public static void main(String[] args) throws Exception {
        new Main().run();
    }

    public void run() throws Exception {

        int cores = Runtime.getRuntime().availableProcessors();
        System.out.println("cores = " + cores);

        Thread thread1 = new Thread(() -> {
                System.out.println("Started thread1");
                for (long i = 0; i < 10000001; i++) {
                    a = (a*3 + 1337) % 1000000;
                }
                System.out.println(new Date().toString() + " finished with A.a = " + a);

        });
        Thread thread2 = new Thread(() -> {
                System.out.println("Started thread2");
                activeWait(2000);
                System.out.println(new Date().toString() + " read value A.a = " + a);
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

    }


    public static void activeWait(int millis) {
        ArrayList<Object> objects = new ArrayList<>();
        for (int i = 0; i < 10000 * millis; i++) {
            //            System.out.println("i = " + i + ", thread = " + Thread.currentThread());
            objects.add(new Object());
        }
    }


    public static void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException ignored) {
        }
    }


}
