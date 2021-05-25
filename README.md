# Starve-Free-Readers-Writers-Problem
CSN-232 Reading Assignment

Provides solution of the Third Reader Writer Problem
### Problem
The readers-writers problem is a classical one in computer science: we have a resource (for example: a database) accessible by readers but not modifiable, and writers who can modify the resource. When the resource is modified by a writer neither the reader nor any other writer can access it simultaneously as another writer could corrupt the resource and a reader sees a partialy modifies value which is inconsistent.
> **1st Problem** => Give readers priority: when there is at least one reader currently accessing the resource, allow new readers to access it as well. This can cause starvation if there are writers waiting to modify the resource and new readers arrive all the time, as the writers will never be granted access as long as there is at least one active reader.

> **2nd Problem** => Give writers priority: here, readers may starve.

> **3rd Problem** => Give neither priority: all readers and writers will be granted access to the resource in their order of arrival. If a writer arrives while readers are accessing the resource, it will wait until those readers free the resource, and then modify it. New readers arriving in the meantime will have to wait.

## Solution

Whole pseudocode is implemented in C++.

Firstly, we will design a semaphore which handles processes in the FIFO(First-In-First-Out) order.The processes stored in the queue are given access to the semaphore in order they called wait.According to the processes calling wait order it gets blocked or pushed in the queue.

```C++
//declare a struct Process with an int procid element
Struct Proc {
    Proc* next;
    int process_id;
}
//declare a struct Queue which takes procid as input and construct two functions push and pop 
struct Queue {
    Proc* head, back;
    
   	void push(int procid) {
        Proc* proc = new Proc();
        proc->process_id = procid;
        if(back == NULL) {
            head = proc;
            back = proc; 
        } else {
            back->next = proc;
            back = proc;
        }
    }
    
    int pop() {
        if(head == NULL) {
            return -1; // underflow 
        } else {
            int procid = head->process_id;
            head = head->next;
            if(head == NULL) {
                back = NULL;
            }
            
            return procid;
        }
    }
}

//declaring Semaphore struct
struct Semaphore{
  Queue* que = new Queue();
  int val;
  
  void wait(int procid){
    val--;
    if(val<0){
    que->push(procid);
    block(procid);
    }
  }
  void signal(){
    val++;
    if(val<=0){
      int procid = que->pop();
      wake(procid);
    }
  }
}

//Global Variables
Semaphore* input, output, write;
input.val = 1;// initial value of input is set to 1
output.val = 1;// initial value ofoutput is set to 1
write.val = 0; // initial value of write is set to 0

int input_read = 0;//started reading by readers
int output_read = 0;//completed reading by readers
bool wait_write = false;

//Reader Process
  input->wait(procid) // wait until call order
  input_read++; //increment if a process starts reading
  input->signal();//send the signal
  //Reading the data
  output->wait(procid); // wait on the out semaphore 
  output_read++;             // increment after reader completed
  if(input_read == output_read && wait_write) {   //writer is waiting and it is last reader
      write->signal();                     
    }                                       // signal write semaphore
  output->signal();

//Writer Process
input->wait(procid); 
output->wait(procid); 
if(input_read == output_read) {   // no processes currently reading
    output->signal(); 
} else {                  //after readers finished
    wait_write = true; 
    output->signal(); 
    writer->wait(); 
    wait_write = false; 
}

//Writing data
input->signal();
```
## Conclusion

Hence, we have created a pseudo code where multiple readers can read the resource and once a writer has queued in the waiting list no reader process can begin until the writer gets access to the resource.Writers presence can be known when the writer sets the value of wait_write = true.If a new writer appears before the first writer leaving then the new writer is queued up. As Semaphore performs in FIFO order the readers who entered between the first writer and the second writer priority is given to the readers.Thus, readers do not starve in the resource.Hence, this is a valid solution.

## Resources
[Samuel Tardieu](https://rfc1149.net/blog/2011/01/07/the-third-readers-writers-problem/)

[Research Paper](https://arxiv.org/ftp/arxiv/papers/1309/1309.4507.pdf)
