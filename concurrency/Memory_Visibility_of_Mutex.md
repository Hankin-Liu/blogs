# Memory Visibility of Mutex   
In concurrency programing, mutex is very useful. As we all know that, it can prevent milti-threads from accessing critical section at the same time. In this article, we only talk about the memory visibility of Mutex. Let us see the following example.  
```c++
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
| Global Variables : int a = 0, b = 0;                                                         |  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
| Thread 1                                   |  Thread 2                                       |   
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
| step 1: a = 10; // global variable         |                                                 |   
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
| step 2: lock(); // thread 1 lock mutex     |                                                 |   
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
| step 3: b = 11; // global variable         |                                                 |  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
| step 4: unlock(); // thread 1 unlock mutex |                                                 |  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
|                                            |  step 5: lock(); // thread 2 lock mutex         |  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
|                                            |  step 6: print a; // read a, a = 10             |  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
|                                            |  step 7: print b; // read b, b = 11             |  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
|                                            |  step 8: unlock(); // thread 2 unlock mutex     |  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
```
## Summary:  
(1) Thread 1's unlock(step 4) happens-before thread 2's lock(step 5)  
(2) Steps of thread 1 before unlock happens-before steps of thread 2 after lock  
    Step 1,2,3,4 happens-before step 5,6,7,8. So in step 6 and 7, a = 10, b = 11, which means thread 2 can see the newest values of a and b which were modified by thread 1 before.  
## Principle
(1) The lock operation contains a read memory barrier. So the read operation in thread 2(step 6 and 7) can't cross step 5.  
(2) The unlock operation contains a write memory barrier. So the write operation in thread 1(step 1 and 3) can't cross step 4.

[\[Back\]](https://github.com/Hankin-Liu/hankin.github.io/blob/master/concurrency/Concurrency.md)
