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
- `public synchronized void start()`: starts the thread object on which the method is called
- `public abstract void run()`: starts the thread object on which the method is called
- `public static native void sleep()`: causes current thread to suspend its execution for the specified time as param (not exact, depends on os). Does not lose monitor or lock objects while sleeping. Called in general inside the `public abstract void run()` method to make the custom thread sleep. Throws an `InterruptedException` if thread is interrupted while sleeping.
- `public void interrupt()`: causes the thread object on which the method is called to be interrupted. The thread must support its own interruption by either catching an `InterruptedException` and treating the exception in the catch block(generally by returning silently), or by checking the `public static boolean interrupted()` method to verify the status of the interruption.
- `public static boolean interrupted()`: called in the `public abstract void run()` generally by a thread object to verify its own interruption while doing heavy processing that does not generally cause an `InterruptedException` to be thrown. Changes the `interrupted` status flag to false.
- `public boolean isInterrupted()`: used to query the interrupt status of another thread by one thread. Does not change the interrupt status of the object is called on.
- `public final void join()`: causes the current executing thread to wait until the `t thread (t.join())` finishes its execution. Throws `InterruptedException` if thread is interrupted while method is executing

### **Thread Issues/Problems/Errors**
- **Thread interference**: errors occurring while threads operate on shared data. One thread incrementing an object's field while the second decrements it at almost the same time. The reading of the object's field can result in unexpected results and thread results being lost (`int a = 0; t1 increments a; t2 decrements a; a can be seen as both 1 and -1`)
- **Memory consistency errors**: errors that result from inconsistent view of shares data between threads (`int a = 0; t1 increments a; t2 reads a; t2 can see a = 0 or a = 1`)
> [!IMPORTANT]
> **Synchronization** solves these problems, but introduces new ones: (**Thread contention/liveness problems**)
- **Thread contention/Liveness problems**: because of **synchronization**, shared resource access can cause threads to execute slower or suspend their execution. 
  - **Starvation**: 
  - **Livelock**:
  - **Deadlock**:

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
    - **Reentrant synchronization**: anti **deadlock** mechanism for one thread. One thread can reacquire a lock held by itself. Can cause **livelock** if one thread reacquires the same lock for too much time
    - 