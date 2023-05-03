Download Link: https://assignmentchef.com/product/solved-cs-5600-homework-2-the-sleeping-barber-problem
<br>
The Sleeping Barber problem is attributed to Djikstra, and is based on the following scenario:

A barbershop contains a barber, the barber’s chair, and N chairs for waiting customers. When there are no customers, the barber sits in his chair and sleeps. When a customer arrives, (a) if the barber is sleeping, the customer awakens the barber and sits down for a haircut; (b) if the barber is busy and there is a free chair, the customer sits down and waits; (c) if the barber is busy and there are no free chairs, the customer leaves immediately.

More specifically, there are 10 customers and 1 barber, each represented by a thread, and a monitor representing the barbershop with two methods, barber() and customer().

<ul>

 <li>The barber thread calls barber(), which never returns. (i.e. the barber never leaves the shop) The barber sleeps until woken by a customer, and then gives haircuts (average time 1.2 seconds, exponentially distributed) until there are no customers left, after which he goes to sleep again.</li>

 <li>Each customer thread loops, alternately sleeping for a period of time (average 10 seconds, exponentially distributed) and then calling customer(), which returns after the customer leaves the shop. (immediately if all chairs are full, or after getting a haircut)</li>

</ul>

<h1>Question 1 – Synchronization</h1>

Provide pseudo-code for a monitor which has two methods, barber() and customer(), modeling a barbershop with 4 waiting chairs and one barber’s chair.

Remember the following characteristics of the monitor definition used in this class:

<ul>

 <li>Only one thread can be <u>in</u> the monitor at a time; threads enter the monitor at the beginning of a</li>

</ul>

method or when returning from <u>wait()</u>, and leave the monitor by returning from a method or entering <u>wait()</u>.

<ul>

 <li>This means there is no preemption – a thread in a method executes without interruption until it returns or waits.</li>

 <li>When thread A calls signal() to release thread B from waiting on a condition variable, you don’t know whether B will run before or after some thread C that tries to enter the method at the same time. (that’s why they’re called <em>race</em> conditions)</li>

 <li>You don’t need a separate mutex – this is a monitor, not a pthreads translation of a monitor.</li>

</ul>

<h1>Question 2 – POSIX Threads</h1>

There is a straightforward translation from monitor pseudo-code to POSIX thread primitives:

<ol>

 <li>Create a per-object mutex, <em>m</em>, of type pthread_mutex_t which is locked on entry to each method and unlocked on exit. (be careful when using multiple exits) Actually, since there’s only one object – the barbershop – there should only be one mutex.</li>

 <li>Condition variables translate directly to objects of type pthread_cond_t; C.signal() and</li>

</ol>

C.broadcast() become pthread_cond_signal(C) and pthread_cond_broadcast(C);

<ol start="3">

 <li>The monitor mutex must be passed to wait calls; thus C.wait() becomes pthread_cond_wait(C,m).</li>

</ol>

For this exercise we will create a singleton monitor, using global variables instead of object variables, and functions rather than object methods.

You will use a single file, <em>homework.c</em>, for both question 2 and 3, using conditional compilation to separate the code for the two; code for question 2 will be compiled with the script <em>compile-q2.sh</em> creating an executable named <em>homework-q2. </em>The startup code for this question will go in the function <em>q2()</em>, and will do the following:

<ul>

 <li>Initialize the monitor objects. Note that mutexes and condition variables may be initialized either statically or dynamically:</li>

</ul>

pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER; /* static init */ pthread_cond_t C = PTHREAD_COND_INITIALIZER;

pthread_mutex_t m;                             /* dynamic */

pthread_mutex_init(&amp;m, NULL);                  /* NULL = default params */

pthread_cond_t C; pthread_cond_init(&amp;C, NULL);

<ul>

 <li>Create N=10 customer threads; each thread will loop doing the following: … sleep for random(T seconds)… customer()</li>

</ul>

You are provided with a sleep function, <em>sleep_exp(T)</em>, where T is a floating point number giving the mean sleep time in seconds. Each thread will need to know its thread number, from 1 to 10; there is a comment in the code describing how to pass this value when starting a thread.

<ul>

 <li>Call <em>wait_until_done(),</em> which will sleep until a command-line-provided timeout or until the user types ^C.</li>

</ul>




<strong>Running question 2</strong>: The <em>homework-q2</em> command is used as follows:

./homework-q2 [-speedup &lt;speedup&gt;] [&lt;time&gt;] where &lt;time&gt; is a total number of seconds the homework should run for, and &lt;speedup&gt; is a speedup factor. (e.g. <em>./homework-q2 100 </em>would run for 100 seconds; <em>./homework-q2 -speedup 2.5 100 </em>would do the same amount of work, but run 2.5 times faster, completing 100 simulated seconds in 40 real seconds.)

To use the debug script provided for this question, you will need to print the following lines as your code executes:

DEBUG: TTT customer # enters shop

DEBUG: TTT customer # starts haircut

DEBUG: TTT customer # leaves shop

DEBUG: TTT barber wakes up

DEBUG: TTT barber goes to sleep where “TTT” is a floating point timestamp returned by the <em>timestamp()</em> function. Use the provided print_* functions to produce these debug lines.

If you redirect the output of the command into a file, you should be able to use the <em>q2test.py</em> script to determine whether or not your implementation obeyed all the rules:

./homework-q2 … &gt; q2.out

./q2test.py q2.out

SUCCESS

You can also pipe the output of <em>homework-q2</em> directly to <em>q2test.py</em>.

./homework-q2 … | python q2test.py-q2

SUCCESS

Note that if you run your program for a short period of time, it is less likely to break any rules even if it is incorrect.

I suggest running Q2 for a while with a high speedup, and possibly performing some work in another terminal window on the same machine (to disturb the thread scheduling order) in order to determine whether it operates correctly and whether it deadlocks.

<h1>Question 3 – Discrete Event Simulation</h1>

For this question you will compile your code using the <em>compile-q3.sh</em> script, which will build it with a framework (fromn <em>misc.c</em>, using the GNU Pth library) for discrete-event simulation. What this means is that your code will run in <em>simulated time</em> – basically the thread library “skips” time forward whenever threads are sleeping, and stops the clock when a thread is running. For simulations of small, slow systems (like ours) this results in a simulation running much faster than real time; for fast, complex systems (e.g. simulating operation of an integrated circuit) the simulation might run thousands of times slower than real time.

The simulation library is designed so that it is compatible with Pthreads operations, so you should be able to compile and run the same monitor code you used in Question 2. In addition several functions have been provided for gathering statistics:

void *counter = stat_counter(); stat_count_incr(counter); stat_count_decr(counter); double val = stat_count_mean(counter); void *timer = stat_timer(); stat_timer_start(timer); stat_timer_stop(timer); double val = stat_timer_mean(timer)

A counter tracks an integer variable (e.g. the number of customers waiting in the shop) and provides its average value over time. A timer tracks the interval between a single thread calling <em>start()</em> and <em>stop()</em>, and provides the average value of these measured intervals.

Run your code for at least 1000 seconds  (10000 would be better) of simulated time and measure average values for the following:

<ul>

 <li>fraction of customer visits result in turning away due to a full shop (you’ll have to keep your own counter for this one)</li>

 <li>average time spent in the shop (including haircut) by a customer who does not find a full shop</li>

 <li>average number of customers in the shop (including the barber’s chair)</li>

 <li>fraction of time someone is sitting in the barber’s chair (hint – use a stat_counter with value 0 for empty and 1 for full)</li>

</ul>

You simply need to demonstrate that your recorded statistics are sensible.

To compile q3 you will need the GNU PTH library (gnu portable threads). Pth 2.0.7 is provided in the repository, as well as a script to compile the library. If you are on linux you can install pth to the system with <em>sudo apt-get install pth-dev, </em>although you will have to modify compile-q3.sh to use the system provided pth library.

To build the provided pth library run <em>build-pth.sh</em>. It will untar the pth-2.0.7.tar.gz, cd to the new directory, run ./configure; make; make install.

The pth library is known to build on a linux machine.