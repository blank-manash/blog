<!--
.. title: Concurrent Programming Problems
.. slug: concurrent-programming-problems
.. date: 2024-09-11 14:32:29 UTC+05:30
.. has_math: false
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

# Introduction

This blog page is dedicated to concurrent programming problems. The problems are taken from various sources and solved in Java. I will expain the various programming constructs used to solve problems.

<!-- TEASER_END -->


# Java Concurrent Constructs

## Semaphore

A semaphore is a synchronization construct that controls access to a shared resource through the use of a counter. It is a record of how many units of a particular resource are available. In simple words, a semaphore with capacity N can only be accessed by N threads at a time. If more than N threads try to access the semaphore, the rest of the threads are blocked until the semaphore is released by one of the N threads.

```java
import java.util.concurrent.Semaphore;

Semaphore semaphore = new Semaphore(3);

semaphore.acquire(); // acquire a permit
semaphore.release(); // release a permit
```

## ReentrantLock

ReentrantLock is a synchronization primitive that is used to protect a critical section of code. It is similar to synchronized block but provides more flexibility. It allows to lock and unlock the critical section explicitly. 

```java
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Lock;

Lock lock = new ReentrantLock();

int variable = 0;

try {
    lock.lock(); // acquire lock
    variable++; // only one thread can access this block
} finally {
    lock.unlock();
}// release lock
```

When using ReentrantLock, it is important to release the lock in a finally block to ensure that the lock is released even if an exception is thrown.

Wait and Notify can be used with Condition object to provide more flexibility.

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Lock;

Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

try {
    lock.lock();
    condition.await(); // wait
    condition.signal(); // notify
} finally {
    lock.unlock();
}

```

Condition Variables are used to aid in inter process communication. They are used to block a thread until a particular condition is met.

## Concurrent Data Structures

* ConcurrentHashMap : A thread-safe version of HashMap.
* ConcurrentSkipListMap : A thread-safe version of TreeMap

Here is a very good resource on ConcurrentQueues: [Java Concurrent Queues](https://www.baeldung.com/java-concurrent-queues)

# Problems

## Problem 1: Unisex Bathroom Problem

Statement: There is a unisex bathroom that has 3 stalls. A bathroom can be occupied by either by a male or female. Design a solution to synchronize the bathroom such that it can be occupied by at most 3 people at a time. No male can enter the bathroom if there is a female and vice versa.
```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.Condition;

public class UnisexBathroom {
    final private Semaphore space;
    final private Lock lock = new ReentrantLock();
    private int maleCount = 0;
    private int femaleCount = 0;
    final private Condition bathroomFree = lock.newCondition();

    public UnisexBathroom(int capacity) {
        space = new Semaphore(capacity);
    }


    public maleUseBathroom(Runnable use) {
        space.acquire();
        lock.lock();
        try {
            while(femaleCount > 0) {
                bathroomFree.await();
            }
            maleCount++;
            lock.unlock();

            use.run();

            lock.lock();
            maleCount--;
            if (maleCount == 0) {
                bathroomFree.signalAll();
            }
        } finally {
            lock.unlock();
            space.release();
        }
    }

    public femaleUseBathroom(Runnable use) {
        space.acquire();
        lock.lock();
        try {
            while(maleCount > 0) {
                bathroomFree.await();
            }

            femaleCount++;
            lock.unlock();

            use.run();

            lock.lock();
            femaleCount--;
            if (femaleCount == 0) {
                bathroomFree.signalAll();
            }
        } finally {
            lock.unlock();
            space.release();
        }
    }
}
```

# Problem 2 : Uber Ride Problem

Statement: Democrats and Republics are waiting for a cab. Only these combinations are allowed to travel together. (2D, 2R), (4R), (4D). Design a solution to synchronize the cab such that it can be occupied by at most 4 people at a time with the above mentioned combinations.

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.CyclicBarrier;

public class UberRideProblem {
    private int democratCount = 0;
    private int republicCount = 0;
    final private Semaphore democratWaiting = new Semaphore(0);
    final private Semaphore republicWaiting = new Semaphore(0);
    final private CycleBarrier barrier = new CycleBarrier(4);
    final private Lock lock = new ReentrantLock();

    public UberRideProblem() {}

    public void demoRide() {
        lock.lock();
        democratCount++;
        boolean rideLeader = false;

        if (democratCount >= 4) {
            democratWaiting.release(3);
            democratCount -= 4;
            rideLeader = true;
        } else if (democratCount >= 2 && republicCount >= 2) {
            democratWaiting.release(1);
            republicWaiting.release(2);
            rideLeader = true;
        } else {
            lock.unlock(); // This order is important, if we acquire first, then there could be deadlock.
            democratWaiting.acquire();
        }

        barrier.await();

        if (rideLeader) {
            System.out.println("Riding");
            lock.unlock();
        }

    }

    public void republicRide() {
        lock.lock();
        republicCount++;
        boolean rideLeader = false;

        if (republicCount >= 4) {
            republicWaiting.release(3);
            republicCount -= 4;
            rideLeader = true;
        } else if (republicCount >= 2 && democratCount >= 2) {
            republicWaiting.release(1);
            democratWaiting.release(2);
            rideLeader = true;
        } else {
            lock.unlock();
            republicWaiting.acquire();
        }

        barrier.await();

        if (rideLeader) {
            System.out.println("Riding");
            lock.unlock();
        }
    }
};
```

# Problem 3: Dining Philosophers Problem

Statement: There are 5 philosophers sitting at a round table. Each philosopher has a plate of spaghetti. A philosopher can either eat or think. To eat, the philosopher needs to have both left and right forks. Design a solution to synchronize the philosophers such that they can eat without causing a deadlock.

Solution: I like to solve this problem by keep the maximum number of concurrent philosophers to 4. This way, we can guarantee that at least one philosopher can eat at a time. But there are other solutions as well. 

```java
import java.util.concurrent.Semaphore;

public class DiningPhilosophersProblem {
    final private Semaphore maxDine = new Semaphore(4);
    final private List<Semaphore> forks = new ArrayList<>();

    public DiningPhilosophersProblem() {
        for (int i = 0; i < 5; i++) {
            forks.add(new Semaphore(1));
        }
    }

    private Semaphore leftFork(int i) {
        return forks.get(i);
    }

    private Semaphore rightFork(int i) {
        return forks.get((i + 1) % 5);
    }

    public void dine(int i) {
        maxDine.acquire();
        leftFork(i).acquire();
        rightFork(i).acquire();

        System.out.println("Philosopher " + i + " is eating");

        leftFork(i).release();
        rightFork(i).release();
        maxDine.release();
    }
}
```

# Problem 4: Design a Bounded Blocking Queue

Statement: Design a bounded blocking queue that has the following methods: enqueue, dequeue, size, isEmpty, isFull. The queue should be able to store a maximum of `capacity` elements.

```java
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.Semaphore;
import java.collections.LinkedList;
import java.util.Queue;
import java.util.LinkedList;

public class BoundedBlockingQueue {
    final private Semaphore enqueueSemaphore;
    final private Semaphore dequeueSemaphore;
    final private queue = new LinkedList<>();
    final private ReentrantLock lock = new ReentrantLock();

    public BoundedBlockingQueue(int capacity) {
        enqueueSemaphore = new Semaphore(capacity);
        dequeueSemaphore = new Semaphore(0);
    }

    public void enqueue(int element) {
        enqueueSemaphore.acquire();
        lock.lock();
        try {
            queue.add(element);
            dequeueSemaphore.release();
        } finally {
            lock.unlock();
        }
    }

    public int dequeue() {
        dequeueSemaphore.acquire();
        lock.lock();
        Integer element = null;
        try {
            element = queue.remove();
            enqueueSemaphore.release();
        } finally {
            lock.unlock();
            return element;
        }
    }
}
```

# Problem 5: Barber Shop Problem

Statement: There is a barber shop that has 3 chairs and a barber. Customers enter the shop and if there are empty chairs, they sit on the chair. If all the chairs are occupied, they leave the shop. The barber cuts the hair of the customer. Design a solution to synchronize the barber shop such that it can be occupied by at most 3 customers at a time.

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Lock;

public class BarberShopProblem {
    final private int CHAIRS;
    final private Semaphore waitForCustomertoEnter = new Semaphore(0);
    final private Semaphore waitForBarberToGetReady = new Semaphore(0);
    final private Semaphore waitForBarberToCutHair = new Semaphore(0);
    final private Semaphore waitForCustomerToLeave = new Semaphore(0);
    final private Lock lock = new ReentrantLock();

    private int waitingCustomers = 0;

    public BarberShopProblem(int capacity) {
        this.CHAIRS = capacity;
    }

    public void runShop() {
        while(true) { // Shop runs indefinitely
            waitForCustomertoEnter.acquire();
            waitForBarberToGetReady.release();
            System.out.println("Barber is Cutting Hair");
            waitForBarberToCutHair.release();
            waitForCustomerToLeave.acquire();
        }
    }

    public void customerWalkIn() {
        lock.lock();
        if (waitingCustomers == CHAIRS) {
            System.out.println("Customer left the shop");
            lock.unlock();
            return;
        }
        waitingCustomers++;
        lock.unlock();

        waitForCustomertoEnter.release();
        waitForBarberToGetReady.acquire();

        lock.lock();
        waitingCustomers--;
        lock.unlock();

        waitForBarberToCutHair.acquire();
        waitForCustomerToLeave.release();
    }
}
```

# Problem 6: Print FizzBuzz

Statement: Write a program that outputs the string representation of numbers from 1 to n. But for multiples of three it should output “Fizz” instead of the number and for the multiples of five output “Buzz”. For numbers which are multiples of both three and five output “FizzBuzz”. If none of the above conditions are met, output the number. There will be 4 threads that will call the following methods: fizz, buzz, fizzbuzz, number. Syncronize the threads such that the output is correct.

```java
import java.util.EnumMap;
enum State {
    FIZZ,
    BUZZ,
    FB,
    NUM
};


class FizzBuzz {
    private int n;
    private int counter = 1;
    private boolean done = false;
    private final EnumMap<State, Semaphore> sem = new EnumMap<>(State.class);

    public FizzBuzz(int n) {
        this.n = n;
        sem.put(State.FIZZ, new Semaphore(0));
        sem.put(State.BUZZ, new Semaphore(0));
        sem.put(State.FB, new Semaphore(0));
        sem.put(State.NUM, new Semaphore(1));
    }

    private State getState(int x) {
        if (x % 3 == 0 && x % 5 == 0) {
            return State.FB;
        }
        if (x % 3 == 0) {
            return State.FIZZ;
        }
        if (x % 5 == 0) {
            return State.BUZZ;
        }
        return State.NUM;
    }

    private void execute(State state, Runnable task) throws InterruptedException {
        System.out.println("Currenty at: " + state.name());
        while(!done && counter <= n) {
            sem.get(state).acquire();
            if (done) {
                return;
            }
            task.run();
            counter += 1;
            if (counter > n) {
                done = true;
                for(Semaphore semaphore : sem.values()) {
                    semaphore.release();
                }
                return;
            }
            State nextState = getState(counter);
            sem.get(nextState).release();
        }
    }


    // printFizz.run() outputs "fizz".
    public void fizz(Runnable printFizz) throws InterruptedException {
        execute(State.FIZZ, printFizz);
    }

    // printBuzz.run() outputs "buzz".
    public void buzz(Runnable printBuzz) throws InterruptedException {
        execute(State.BUZZ, printBuzz);
    }

    // printFizzBuzz.run() outputs "fizzbuzz".
    public void fizzbuzz(Runnable printFizzBuzz) throws InterruptedException {
        execute(State.FB, printFizzBuzz);
    }

    public void number(IntConsumer printNumber) throws InterruptedException {
        execute(State.NUM, () -> { printNumber.accept(counter); });
    }
}
```
