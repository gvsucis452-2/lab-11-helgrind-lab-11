1. First build `main-race.c`. Examine the code so you can see the
   (hopefully obvious) data race in the code. Now run helgrind (by
   typing `valgrind --tool=helgrind ./main-race`) to see how it
   reports the race. 
   * Does it point to the right lines of code? 
   * What other information does it give to you?

**Answer**: It correctly points to lines 15 and 8 as a potential race condition. It gives information about which threads are created, which the main thread is, number of locks held, and how many bytes the write of the race condition is. It also gives a summary of the number of errors and the number of contexts that give errors.

2. What happens when you remove (e.g., comment out) one of the offending lines of code?

**Answer**: Helgrind prints out no errors

3. Add a lock around _one_ of the updates to the shared variable. What does helgrind report?

**Answer**: It prints out a message similar to the original. However, it notes that one has a lock held and the other does not. It still correclty detects the race conditions at both balance++ lines. 

4. Now add locks around both. What does helgrind report?

**Answer**: It prints the same as when they are commented out. No errors.

5. Next examine `main-deadlock.c`. This
   code has a problem known as _deadlock_ (which we discuss in much
   more depth in a forthcoming chapter). Based on this code, 
   * Describe what a deadlock is.
   * Why specifically does this code have a deadlock?

**Answer**: A deadlock is when the process grinds to a halt. This happens in this code if a context switch happens after one thread grabs lock m1, but before grabbing m2. The other process then runs and grabs lock m2, but cannot proceed because m1 is locked. When the switch back to the first thread happens, it goes to grab lock m2, but cannot proceed because it is held by the other process. This creates the deadlock. 

6. Run helgrind on this code. What does it report?

**Answer**: It reports that there is a lock order violation on thread 3. It then lists the lines that the locks are out of order. (lines 11 and 14) and (lines 10 and 13).

7. Examine `main-deadlock-global.c`. 
   * Does it have the same problem `that main-deadlock.c`
   has? 
   * Why or why not? 
   * Should helgrind be reporting the same error? 
   * What does this tell you about tools like helgrind?

**Answer**: No, this should not have a deadlock problem, since the out of order locks are wrapped in another lock. Two threads cannot hold m1 or m2 at the same time since they are wrapped in this critical section. Helgrind should not be reporting an error, but still reports the out of order locks.

This means that helgrind is looking more preemptively for out of order locks, it doesn't run and *verify* an error when it actually happens. However, the inner locks are technically out of order and this kind of setup does not make any sense in practicle applications. Helgrind is doing it's job.

8. Look at `main-signal.c`. This code uses a variable (`done`)
   to signal that the child is done and that the parent can now continue.
   Why is this code inefficient? (what does the parent end up spending its time doing, particularly if the child thread takes a long time
   to complete?)

9. Run helgrind on this program. 
   * What does it report? 
   * Is the code correct?


10. Now look at the slightly modified version of the code found
   in `main-signal-cv`.c. This version uses a condition variable to
   do the signaling (and associated lock). 
   * Why is this code preferred to the previous version? 
   * Is it correctness, or performance, or both?

11. Once again run helgrind on `main-signal-cv`. Does it report
   any errors?
