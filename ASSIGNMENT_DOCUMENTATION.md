# Assignment 3 - Complete Documentation

**Student Name**: [Yousef Khalid Orqubi]  
**Student ID**: [444051039]  
**Date Submitted**: [May 2 , 2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[444051039]_Assignment3_Synchronization.mp4` https://drive.google.com/file/d/138Q36YvrMFHmGQruxIH15fd7OuQk_b02/view?usp=drivesdk

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 1, 2026, 10:30 AM]
**What I implemented**: 
I opened SchedulerSimulationSync.java after forking the starter repository, ensuring it was public, then cloning it into Visual Studio Code. Additionally, I changed the `studentID` variable to 444051039.
**Challenges encountered**: 
The first difficulty was figuring out how the simulation is impacted by the student ID. The produced number of processes, burst times, priority, and time quantum are all dependent on the student ID since the program utilizes it as the seed for the random number generator.
**How I solved it**: 
I found the studentID variable within the main method and entered my real student ID in place of the default value.
**Testing approach**: 
I compiled and ran the program once to confirm that the header displayed my student ID correctly and generated the expected simulation setup.

**Time spent**: 
10 min
---

### Entry 2 - [May 2, 2026, 1:15 PM]
**What I implemented**: 
I made the synchronization objects in the SharedResources class and supplied the necessary synchronization imports. I added a Semaphore for CPU access control, another ReentrantLock for the execution log, and a ReentrantLock for shared counters.
**Challenges encountered**: 
Determining which shared resources need protection was the difficult part. Because all process threads share the counters and the execution log, race conditions may have an impact on them.
**How I solved it**: 
I introduced cpuSemaphore with a single permit to indicate exclusive access to the simulated CPU, counterLock to safeguard shared counters, and logLock to safeguard the shared ArrayList.
**Testing approach**: 
After adding the imports and synchronization objects, I tested that the program was still compiling.
**Time spent**: 
1 hour
---

### Entry 3 - [May 2, 2026, 2:40 PM]
**What I implemented**: 
I used ReentrantLock to safeguard the shared counter methods. The following methods were updated: addWaitingTime(long time), incrementContextSwitch(), and incrementCompletedProcess().
**Challenges encountered**: 
Understanding that actions like contextSwitchCount++ and totalWaitingTime += time are not atomic was the biggest obstacle. They entail reading, editing, and rewriting the value.
**How I solved it**: 
I put counterLock.unlock() inside a finally block and accompanied each counter update with counterLock.lock().
**Testing approach**: 
I ran the software to make sure it didn't crash or freeze and that the end statistics were still visible.
**Time spent**: 
1 hour.
---

### Entry 4 - [May 2, 2026,]
**What I implemented**: 
I changed the logExecution(String message) method to safeguard the execution log.
**Challenges encountered**: 
The execution log makes use of the non-thread-safe ArrayList. The list may become incorrect if several threads add log items simultaneously.
**How I solved it**: 
To safeguard executionLog.add(message), I utilized logLock. In order to ensure that the lock is always released, I additionally put the unlock action inside a finally block.
**Testing approach**: 
After running the software, I confirmed that the execution log summary had been printed successfully. Total log entries: 64 was displayed in the final output.
**Time spent**: 
50 minutes.

---

### Entry 5 - [May 2, 2026]
**What I implemented**: 
I modified the process execution section to include semaphore control. I used SharedResources.cpuSemaphore.release() inside the finally block and SharedResources.cpuSemaphore.acquire() prior to CPU execution.
**Challenges encountered**: 
Placing the semaphore appropriately was the difficult part. Even in the event of an interruption, it must be obtained prior to the process commencing and released.
**How I solved it**: 
I put release() inside the finally block and acquire() at the start of the run() procedure. This guarantees that the emulated CPU can only be accessed by one process at a time.
**Testing approach**: 
I ran the application several times to make sure every operation finished smoothly. 17 processes, 32 context switches, and 17 finished processes were displayed in my last run.
**Time spent**: 
1 hour.
---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[The first race condition is related to the shared counter variables such as contextSwitchCount, completedProcessCount, and totalWaitingTime. These variables are shared by all process threads and can be updated by multiple threads. For example, contextSwitchCount++ is not atomic because it requires reading the old value, incrementing it, and writing the new value back. If two threads do this at the same time, one update may be lost, causing an incorrect final context switch count.

The second race condition is related to executionLog, which is an ArrayList. ArrayList is not thread-safe, so multiple threads adding messages at the same time can corrupt the internal structure of the list or produce inconsistent log results. This could cause missing log entries or runtime problems such as ConcurrentModificationException. To prevent this, I protected the log update using a separate ReentrantLock.]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[ReentrantLock is mainly used to protect critical sections where shared data is accessed or modified. It provides mutual exclusion, meaning only one thread can enter the locked section at a time. In my code, I used counterLock to protect the shared counters and logLock to protect the executionLog ArrayList.

A Semaphore controls how many threads can access a shared resource at the same time. In my code, I used cpuSemaphore with one permit to control access to the simulated CPU. This means only one process can execute its CPU quantum at a time. Therefore, locks were used for protecting shared data, while the semaphore was used for controlling access to a shared resource.]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Because each thread is holding a resource and waiting for another resource that will never be released, deadlock occurs when threads wait indefinitely. Always releasing locks and semaphores within a `finally` block is one preventative measure. This guarantees the release of resources in the event of an interruption or exception.

Avoiding needless nested locking and keeping important portions brief are two other preventative strategies. Each method in my code only locks the resource it requires and releases it right after following an update. To ensure that no lock or semaphore is held indefinitely, I used `try-finally` with `counterLock`, `logLock`, and `cpuSemaphore`.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[For the three counters, I used one lock called `counterLock`, which is a coarse-grained locking approach. I made this choice because the counter updates are very short operations, and using one lock keeps the implementation simple, readable, and less error-prone. The main advantage of coarse-grained locking is simplicity because all related counter updates are protected using one synchronization object.

The trade-off is that coarse-grained locking may reduce concurrency because only one thread can update any counter at a time, even if the counters are independent. Fine-grained locking, where each counter has its own lock, can provide better concurrency because different threads could update different counters at the same time. Since the three counters are independent, fine-grained locking would theoretically provide better concurrency. However, in this assignment, using one `counterLock` is reasonable because the critical sections are small and the code remains easier to understand and verify..]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
`contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`
**Why they need protection**: 
They are shared variables accessed by multiple process threads. Updating them without synchronization may cause lost updates and incorrect final statistics.

**Synchronization mechanism used**: 
`ReentrantLock` using `counterLock`
**Code snippet**:
```java
/* public static void incrementContextSwitch() {
    counterLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        counterLock.unlock();
    }
}
*/
**Justification**: 
These techniques alter shared counters. The lock makes sure that only one thread changes the counters at a time because addition and increment operations are not atomic. Even in the event of an error, the //finally 
block ensures that the lock is released.
---

### Critical Section #2: Execution Log

**What resource**: 
executionLog, which is a shared ArrayList<String>

**Why it needs protection**: 
ArrayList is not thread-safe. If multiple threads add messages at the same time, the log can become inconsistent or corrupted.

**Synchronization mechanism used**: 
ReentrantLock using logLock.
**Code snippet**:
```java
/* public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}
*/

**Justification**: 
The log is shared by all processes. The lock prevents multiple threads from modifying the ArrayList at the same time and avoids inconsistent logging behavior.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore controls access to the simulated CPU.

**Number of permits and why**: 
The semaphore has one permit because only one process should use the simulated CPU at a time.

**Where implemented**: 
Inside the //run()
 method of the Process class.
**Code snippet**:

```java
/* public void run() {
    try {
        SharedResources.cpuSemaphore.acquire();

        if (startTime == -1) {
            startTime = System.currentTimeMillis();
        }

        SharedResources.incrementContextSwitch();

        int runTime = Math.min(timeQuantum, remainingTime);

        // Process execution code remains here

    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        SharedResources.cpuSemaphore.release();
    }
}
*/
**Effect on program behavior**: 
The semaphore ensures that only one process can execute its CPU quantum at a time. If another process tries to execute while the CPU is being used, it must wait until the semaphore is released. This supports controlled access to the simulated CPU and prevents concurrent CPU execution.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync

**Results**: 
(Show that running multiple times produces consistent, correct results)
═══ Synchronization Statistics ═══
Total Context Switches: 32
Total Completed Processes: 17
Total Waiting Time: 978023ms
Average Waiting Time: 57530ms

═══ Process Summary Table ═══
Process    Priority     Burst Time   Waiting Time
────────────────────────────────────────────────
P1         5            3022         23          
P2         1            3312         3045        
P3         3            8823         81939       
P4         4            5833         60117       
P5         4            4755         61961       
P6         3            6617         62728       
P7         5            9060         82767       
P8         4            2409         26635       
P9         2            2886         29063       
P10        3            8696         83831       
P11        5            9986         84532       
P12        3            6757         77490       
P13        4            5643         80249       
P14        4            2806         48146       
P15        2            2935         50964       
P16        5            9696         86561       
P17        2            2075         57972       

═══ Execution Log Summary ═══
Total log entries: 64

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)
Without synchronization, race conditions could still arise even if the software seems to generate accurate results. Multiple threads can access shared variables like contextSwitchCount, completedProcessCount, and totalWaitingTime. Operations such as increasing a counter (contextSwitchCount++) are not atomic in the absence of locks and could result in lost changes. In a similar vein, the executionLog is not thread-safe because it is an ArrayList. Data corruption or runtime problems could result from multiple threads making simultaneous changes to it. Safe access to these shared resources is guaranteed by synchronization.

**Conclusion**: 
After synchronization methods are applied, the program operates consistently and accurately. Every procedure is completed successfully, and every statistic is correct.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 
I ran the application several times while keeping an eye on the behavior of the execution log. I concentrated on making sure that adding items to executionLog by several threads wouldn't result in errors.

**Results**: 
The program completed successfully without throwing ConcurrentModificationException. The execution log summary showed a total of 64 log entries.

**What this proves**: 
This demonstrates that concurrent modification problems are successfully avoided by utilizing ReentrantLock to safeguard the executionLog. It guarantees that the ArrayList is only modified by one thread at a time.
---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 
The total number of produced processes and the number of completed processes should be equal. The anticipated number of finished processes is 17 as the program produced 17 processes. The number of times processes were scheduled on the CPU should be reflected in the context switch count.

**Actual values**: 
Total Context Switches: 32
Total Completed Processes: 17
Total Waiting Time: 978023ms
Average Waiting Time: 57530ms

**Analysis**: 
The number of produced processes and the completed process count match, indicating that every process has completed its execution. The number of CPU scheduling events is reflected in the context switch count. The execution log attests to the accurate recording of process execution events. These outcomes confirm that the program operates as intended.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 
to confirm that processes requiring one quantum and those requiring multiple quanta are handled appropriately by the scheduler.
**Results**: 
Processes with burst times less than or equal to the time quantum (4000ms) completed in one execution cycle. Examples include P1, P2, P8, P9, P14, P15, and P17. Processes with burst times greater than the time quantum required multiple cycles and were re-added to the ready queue, such as P3, P7, P10, P11, and P16.
**What I learned**: 
The Round Robin algorithm is appropriately followed by the scheduler. Every process is only allowed to run for a single time quantum before being rescheduled if there is still time.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[I discovered that when several threads access common resources, synchronization is crucial. Because they are not atomic, even basic operations like increasing a variable are unsafe in a multithreaded context. I recognized the significance of locating crucial software segments and safeguarding them with synchronization techniques. I was able to guarantee mutual exclusion when updating shared counters and logs by using ReentrantLock. Additionally, I discovered that semaphores can be used to manage access to finite resources, such a CPU. The use of try-finally blocks to guarantee that locks and semaphores are always released was the most crucial lesson. I was able to make the connection between operating system theory and real-world application thanks to this assignment.]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking systems where multiple transactions access the same account balance. Synchronization ensures that concurrent transactions do not produce incorrect balances.

**Example 2**: 
Operating system printer queues, where multiple users send print jobs. Synchronization ensures that jobs are processed in order without conflict.
---

### How I would explain synchronization to others:

[Controlling access to a shared resource is analogous to synchronization. Consider a space with just one chair. People will interfere with one another if they attempt to sit at the same time. Only one person can access a room at a time thanks to a lock, which functions similarly to a key. A semaphore permits a set number of people to enter, much like a set number of keys. Shared variables are the room in this assignment, and synchronization makes sure that only one thread may change them at a time.]

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/yousef-1239/OS-Assignment3-Yousef-Oequbi.git

**Number of commits**: 5 commits

**Commit messages**: 
1. change my id number to 444051039
2. Add ReentrantLock and Semaphore for synchronization
3. Protect shared counters using ReentrantLock
4. Protect execution log using ReentrantLock
5. Apply semaphore control to runToCompletion method

---

## Summary

**Total time spent on assignment**: 
2 days
**Key takeaways**: 
1. Race conditions can occur even in simple operations without synchronization
2. Locks ensure mutual exclusion for shared data
3. Semaphores control access to shared resources

**Most challenging aspect**: 
Identifying all critical sections and ensuring proper placement of locks and semaphores

**What I'm most proud of**: 
Successfully implementing synchronization mechanisms and achieving correct, stable program output
---

**End of Documentation**
