# **Contents**

- [Threads](#threads)


<a id="threads"></a>
## **Threads**
### **Starting a thread**
- By implementing `Runnable` in a custom class, overriding the `public abstract void run()` method and passing the runnable object to the `Thread` constructor as a parameter and then calling the `public synchronized void start()` method;
- By extending `Thread` class, overriding the **public abstract void run()** method and then calling the `public synchronized void start()` method.
> [!IMPORTANT]
> Calling the `public abstract void run()` method from the `Runnable` instantiated object will not start a new thread and the code in the method will be executed in the current thread

### **Thread methods**
- `public synchronized void start()/ t.start()`: starts the thread object on which the method is called
- `public abstract void run()/ r.run()`: the work needing to be done by a thread. Does not start a separate thread if called.
- `public static native void sleep()/ Thread.sleep()`: causes current thread to suspend its execution for the specified time as param (not exact, depends on os). Does not lose monitor or lock objects while sleeping. Called in general inside the `public abstract void run()` method to make the custom thread sleep. Throws an `InterruptedException` if thread is interrupted while sleeping.
- `public void interrupt()/ t.interrupt()`: causes the thread object on which the method is called to be interrupted. The thread must support its own interruption by either catching an `InterruptedException` and treating the exception in the catch block(generally by returning silently), or by checking the `public static boolean interrupted()` method to verify the status of the interruption.
- `public static boolean interrupted()/ t.interrupted()`: called in the `public abstract void run()` generally by a thread object to verify its own interruption while doing heavy processing that does not generally cause an `InterruptedException` to be thrown. Changes the `interrupted` status flag to false.
- `public boolean isInterrupted()/ t.isInterrupted()`: used to query the interrupt status of another thread by one thread. Does not change the interrupt status of the object is called on.
- `public final void join()/ t.join()`: causes the current executing thread to wait until the `t thread (in t.join())` finishes its execution. Throws `InterruptedException` if thread is interrupted while method is executing
- `Object.public final void wait()/ o.wait()`: causes the current executing thread of the code block to wait until it is notified by method call `Object.public final native void notifyAll()/notify()`. Executing thread must own the monitor lock of resource object that called this method. When an object calls this method, the affected thread will release the object monitor lock and is added to the **wait set** of the object. Throws an `InterruptedException` if thread is interrupted while waiting. 
- `Object.public final native void notifyAll()/notify()/ o.notifyAll()`: causes all the threads that are waiting to acquire the monitor lock of the object calling this method to wake up. If multiple threads are woken up, all threads compete to acquire the object's monitor lock. The thread is removed from the **wait set** of the object. 

### **Thread Issues/Problems/Errors**
- **Thread interference**: errors occurring while threads operate on shared data. One thread incrementing an object's field while the second decrements it at almost the same time. The reading of the object's field can result in unexpected results and thread results being lost (`int a = 0; t1 increments a; t2 decrements a; a can be seen as both 1 and -1`)
- **Memory consistency errors**: errors that result from inconsistent view of shares data between threads (`int a = 0; t1 increments a; t2 reads a; t2 can see a = 0 or a = 1`)
> [!IMPORTANT]
> **Synchronization** solves these problems, but introduces new ones: (**Thread contention/liveness problems**)
- **Thread contention/Liveness problems**: because of **synchronization**, shared resource access can cause threads to execute slower or suspend their execution. 
  - **Starvation**: greedy thread t1 acquires lock of resource object a for long period of time resulting in other threads not being able to access resource object a.
  > [!IMPORTANT]
  > **Starvation avoidance:**
  > // TODO
  - **Livelock**: The same situation as in **deadlock**, but thread t1 releases lock for object a and thread t2 releases lock for object b to unblock the other thread, and start all over from step 1 in **deadlock** scenario.
  > [!IMPORTANT]
  > **Livelock avoidance:**
  > 1) Different intervals for the livelocked threads to reacquire the locks. Thread t1 waits 1s and thread t2 waits 2s.
  > 2) Use of **guarded blocks** with a notification system in order for one thread to notify the other thread when a lock is released
  - **Deadlock**: 2 resource objects a, b. Thread t1 acquires lock for object a at the same time thread t2 acquires lock for object b. Thread t1 then tries to acquire lock for object b and t2 tries to acquire lock for object a. No thread releases any lock. Threads can not advance
  > [!IMPORTANT]
  > **Deadlock avoidance:** 
  > 1) avoid acquiring multiple locks for a single thread.
  > 2) if not possible, lock order should be maintained for all threads. 

### **Solving Thread Issues/Problems/Errors**
- **Synchronization**: solves **thread interference** and **memory consistency errors** => can cause **liveness/thread contention** problems
  - `synchronized` keyword is used on methods or code blocks(must specify the lock object, most common `this` instance): 
    - acquires an object's intrinsic lock(monitor lock/monitor) or the `Class` object(for static methods or fields)
    - makes a resource accessible by one thread at a time
    - establishes a happens before relationship with every other subsequent access to that resource. The resource modification by one thread is seen by other threads accessing it afterwards
    > [!IMPORTANT]
    > `synchronized` can be used in methods for code blocks as well: 
    >```java
    >public void addName(String name) {
    >  synchronized(this) {
    >    lastName = name;
    >    nameCount++;
    >  }
    >  nameList.add(name);
    >}
    >```
    > `synchronized` code blocks **should** avoid invocations of other object's methods: `nameList.add(name)` because it may cause liveness issues
    - `synchronized` can be used to fine grain access to separate resources within a class (2 separate fields that are never used together can be synchronized separately in 2 different synchronized blocks)
    - **Reentrant synchronization**: anti **deadlock** mechanism for one thread. One thread can reacquire a lock held by itself. Can cause **livelock** if one thread reacquires the same lock for too much time.
- **Guarded blocks**: solves **thread synchronization/coordination** issues
  - condition based code block used for threads to wait until condition is fulfilled
  >```java
  >public synchronized void guardedJoy() {
  >    while(!joy) {
  >        try {
  >            d.wait();
  >        } catch (InterruptedException e) {}
  >    }
  >    System.out.println("Joy and efficiency have been achieved!");
  >}
  >```
  - `guardedJoy()` method is synchronized because the thread accessing the method must own the lock of the object in order to call `wait()`, otherwise `IllegalMonitorStateException` is thrown.
  - `wait()` method is used because it is more expensive to loop until the condition is satisfied than to puase execution
  - the condition for the thread to wait is until `joy = true`
  - to wake up waiting threads `notify()/notifyAll()` can be used
  >```java
  >public synchronized notifyJoy() {
  >    joy = true;
  >    notifyAll();
  >}
  >```
  - once `joy = true`, the waiting threads are woken up by calling `notifyAll()`
  - the waiting threads now compete to acquire the lock for the object and start executing the remaining `guardedJoy()` method code
  - `notify()` call removes threads from resource object **wait set**
- **Atomic access/ `volatile` keyword**: solves **thread interference** at field level
  - an **atomic action** means an action that happens all at once (`a++` does not translate anymore to `read a; a = a + 1; write to a`)
  - actions/changes on a `volatile` variable are seen by other threads only after the action has taken place
  - establishes a **happens-before** relationship with subsequent reads