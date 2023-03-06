# Starve-free Readers-Writers Problem

The reader-writer problems deal with synchronizing multiple processes trying to do read or write operation upon a shared memory segment. The first and second reader-writer problem provide a solution where one of reader or writer have to starve. The third readers-writers problem deals with an implementation which is more flexible and equitable in terms that both reader-writers synchronize without starving.
The classical solution to the Reader-Writer Problem involves setting up a semaphore with a counter initialised to a number of Readers allowed to simultaneously read the critical section, but this solution has too many deficiencies, such as assumption that the semaphore is buffered, necessity to know upfront a number of possible Readers, and a linear loop depending on the number of Readers.

## Implementing Semaphore

A mutex is a special type of synchronisation concept used to allow concurrent threads mutually exclusive access to a critical data structure. It is implemented as a semaphore which is  a positive number which allows increment and decrement by one operations.First, let's discuss the implementation of the semaphore which is peculiar to required solution. For a starve-free implementation, we need a semaphore that has a FIFO queuen manner of handing operations which are waiting. 

The psedocode implementing semaphore is:

<aside>
💡

// The code for a FIFO semaphore as queue data structure.

```cpp
struct Semaphore
{
int val = 1;   //here val is like setting priority of process which come first in order.
Queue* Q = new Queue();
void wait(int process_id)
{
    val--;
    if(val < 0)
    {
        Q->push(process_id);
        block(); //this function will block the proccess until it's signalled.

    }
}

void signal()
{
    val++;
    if(val <= 0)
    {
        int pid = Q->pop();
        wakeup(pid); //this function will wakeup the process with the given process-id.
    }
}

```

}

//The code for the queue which will allow us to make a FIFO semaphore.

```cpp
struct Queue
{
Node* F, R;           //F=front pointer R=rear pointer
void push(int val)
{
Node* n = new Node();
n->val = val;
if(R != NULL)
{
R->next = n;
R = n;
}
else
{
F = R = n;
}
}
int pop()
{
    if(F == NULL)
    {
        return -1; // Error : underflow.
    }
    else
    {
        int reqval = F->val;
        F = F->next;
        if(F == NULL)
        {
            R = NULL;
        }
        return reqval;
    }
}
}

// A queue node.
Struct Node
{
Node* next;
int val;
}
```

</aside>

Thus ,here a FIFO semaphore is implemented with a queue to manage waiting processes.

## Required variables

<aside>
💡

```cpp
Semaphore* in = new Semaphore();
Semaphore* out = new Semaphore();
Semaphore* wrt = new Semaphore();
wrt_sem->value = 0; // peculiar
int num_started = 0; // number of readers that have started reading.
int num_completed = 0;//number of  readers that have completed reading.
//num_started and num_completed are kept in different variable
//instead of merging them into one is because they would be changed by different semaphores.
bool writer_waiting = false; // this indicates writer is waiting.
```

</aside>

## Reader process

<aside>
💡

```cpp
in.Wait(pid) ; //wait on  in_sem semaphore given the process id is accessible
num_started++; //increment number readers started
in.Signal(pid);
```

//Read ["critical section"]

```cpp
out.Wait(pid); //wait on the out_sem semaphore.
num_completed++;//increment number of readers completed reading
if(writer_waiting && num_started == num_completed)
{
wrt.Signal();   //signals to waiting writer process
}
out.Signal();
```

</aside>

## Writer Process

<aside>
💡

```cpp
//Writer Process
in.Wait(pid);
out.Wait(pid);
if(num_started == num_completed)
{
out.Signal();
}
else
{
writer_waiting = true;
out.Signal();
wrt.Wait();
writer_waiting = false;
}
```

//Write the data. This is the "critical section"

```cpp
in.Signal();
```

</aside>

## Explanation

The main idea here is that a **Writer indicates to Readers its necessity to access the working area**. At the same time no new Readers can start working. Every Reader leaving the working area checks if there is a Writer waiting and the last leaving Reader signals Writer that it is safe to proceed now. Upon completing access to the working area Writer signals waiting Readers that it finished allowing them to access the working area again.

The starve-free solution works on this method, where any number of readers can simultaneously read the data. A writer will make its presence known by setting **writer_waiting** to true and acquiring the **in** semaphore and not leaving it until the end of its process. This ensures that any new process that comes after this (whether it be it reader or writer) will be queued up on **in semaphore**. This is due to the **FIFO** nature of semaphore, where processes come as **RRRWRWRRR** and the first three readers will first read, then the next process which is a writer will take it up until it's done. When the writer is done writing, it won't get to start writing before the process before it, the reader process has completed.

A FIFO semaphore is a solution to the readers-writers problem where all processes are handled in a FIFO manner, allowing other readers to start reading with them but blocking all other processes waiting in the queue from executing before it finishes. To make the **wrt semaphore** synchronizable across both the process where one process only executes wait() and other only executes signal(), we initialize it to 0. To explain the reader process, it first increments the num_started then reads the data and then increments the num_completed. If both variables are equal, it signals the writer semaphore **wrt** and finishes. This way, no process will have to indefinitely wait leading to starvation.

This solutionalso  works with no starvation with semaphores where waiting threads selected randomly by the operating system, because the probability that thread does not get a chance to execute decreases with time exponentially. Actually this solution is semaphore dependent.For example if we are presented with buffered FILO semaphore then it is vulnerable to failure.

In that case we apply Morris algorithm to avoid case of starvation.*Morris proposes a starvation free solution to mutual exclusion problem using three binary semaphores with a weak property that if any process is waiting to complete a P(s) operation, the semaphore is not incremented and one of the waiting processes completes it.*

## References

- H. Ballhausen, arXiv:cs/0303005, 2003
- Morris JM (1979) A starvation-free solution to the mutual exclusion problem. Inf Process Lett 8:76–80
- [https://www.sciencedirect.com/science/article/abs/pii/0020019079901479?via%3Dihub](https://www.sciencedirect.com/science/article/abs/pii/0020019079901479?via%3Dihub)
