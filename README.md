# Multithreading & Concurrency
## Introduction

There's two kinds of concurrent programming model:
* **Multiprocessing**
  
  In Multiprocessing there's multiple processes running, each with a single thread, and the processes communicate with each other at regular intervals through ipc via files, message queues, etc. 
* **Multithreading**

  In Multithreading one process contains two or more thread and the threads communicate via shared memory.

Pros and Cons:

- A thread is faster to start, its usually slow and complicated to start a process. A thread is a lightweight process.

- A thread takes lower overhead in running; a process has more overhead. For example, the operating system needs to provide process a lot of protection, so that one process don't accidentally step on to another process.

- Communicating through shared memory is a lot faster than communicating through ipc channels

> Multithreading provides better performance than multiprocessing.
 
- Multithreading is difficult to implement
> Multithreading program cannot be run in a distributed system, whereas multiprocessing can be easily distributed to multiple computers and run concurrently.


```
#include <iostream>
#include <thread>
using namespace std;

void func() {
  cout<< "Hello World"<<endl;
}

int main() {
  std::thread t1(func); //t1 starts running
  t1.join(); //main thread waits for t1 to finish

  return 0;
}
```

Now we have two threads running, the main thread and the t1. Here, main thread waits for t1 to finish, we can also detach the t1 thread and make it independent with *t1.detach()* instead of t1.join. the main thread can run fast and end up exiting without t1 to finish up, and hence cout nothing. t1 now becomes a daemon process, some daemon process may be alive till the system is shut down. 

You can join or detach the thread only once,else it will crash - once detached always detached.

move() - transfers ownership

## Race Condition and Mutex

```
#include <iostream>
#include <thread>
using namespace std;

void func() {
    for(int i=0;i<10;i++){
        cout<<"From thread: "<<i<<endl;
    }
}

int main() {
    thread t1(func);

    for(int i=-10;i<0;i++){
        cout<<"From main: "<<i<<endl;
    }

    t1.join();
    return 0;
}

Output:
From thread: From main: 0-10
From thread: 1
From thread: 2
....
```

Here both the t1 thread and the main thread is competing(or racing) for a common resource - cout and is causing the messedup output. One way to solve for race conditon is using mutex to synchronize the access of shared resource. 

Instead of calling cout from each thread we can create a new function and lock the print function - 
```
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

std::mutex mu;

void shared_print(string msg, int id) {
    mu.lock();
    cout<<msg<<id<<endl;
    mu.unlock();
}

void func() {
    for(int i=0;i<10;i++){
        shared_print("From thread: ", i);
    }
}

int main() {
    thread t1(func);

    for(int i=-10;i<0;i++){
        shared_print("From main: ", i);
    }

    t1.join();
    return 0;
}

This works completely fine.
```

Issues: what if the cout in shared_print throw an exception, the print statement will always be locked and hence inaccessible, we hence use lock guard instead of mu.lock and unlock. This ensures the mutex will be unlocked when the resource acquisition goes out of scope.

```
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

std::mutex mu;

void shared_print(string msg, int id) {
    lock_guard<mutex> guard(mu);
    cout<<msg<<id<<endl;
}

void func() {
    for(int i=0;i<10;i++){
        shared_print("From thread: ", i);
    }
}

int main() {
    thread t1(func);

    for(int i=-10;i<0;i++){
        shared_print("From main: ", i);
    }

    t1.join();
    return 0;
}
```
Another issue - cout is not completely under the mutex lock as its a global resource and is still accessible elsewhere even though its locked in shared_print.













