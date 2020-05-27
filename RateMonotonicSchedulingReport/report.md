# <p style="text-align: center;">Rate Monotonic Scheduling on a Single CPU</p>

## Introduction
During the late 20<sup>th</sup> century, the market for real-time systems experienced some rapid growth; which 
lead to the rising need of scheduling algorithms that can provide deadline guarantees for the hard-real-time 
systems. While the importance of a scheduling algorithm might not be directly obvious at the beginning, simply
thinking of a job that is taking control over the CPU while it is waiting for I/O while other jobs are waiting
will give you an idea of how putting this I/O bound job will give a massive performance boost and increase
efficiency in multi-job systems. At that time, a paper by Liu and Layland [1] was published in which they 
suggested a couple of scheduling algorithms. One of the algorithms introduced in this paper is the "Rate 
Monotonic Scheduling" algorithm.  Despite being created over four decades ago, it is still one of the most 
widely used scheduling algorithms used for real-time systems. In this article, we'll be discussing its major 
features, general idea and the upper bound of its theoretical single CPU utilization.

## Assumptions
For the "Rate Monotonic Scheduling" algorithm to be considered viable, some assumptions are made by Liu and 
Layland [1] so that any external factors are ignored and only the efficiency of the algorithm itself is the
main focus:
1. The requests for all jobs for which hard deadlines exist are periodic, with constant interval between requests.
2. Deadlines consist of run-ability constraints only--i.e. each job must be completed before the next request for it occurs.
3. The jobs are independent in that requests for a certain job do not depend on the initiation or the completion of requests for other jobs.
4. Run-time for each job is constant for that job and does not vary with time. Run-time here refers to the time which is taken by a processor to execute the job without interruption.
5. Any non-periodic jobs in the system are special; they are initialization or failure-recovery routines; they displace periodic jobs while they themselves are being run, and do not themselves have hard, critical deadlines.

## Background
For clarification, the following example was taken directly from Marilyn Wolf's *Computers as Components* [2]:

For the simple set of jobs described by this table:

    +----------------+----------------+----------------+
    |    Process     | Execution Time |     Period     |
    +----------------+----------------+----------------+
    |      P1        |       1        |       4        |
    +----------------+----------------+----------------+
    |      P2        |       2        |       6        |
    +----------------+----------------+----------------+
    |      P3        |       3        |       12       |
    +----------------+----------------+----------------+

By applying the principles of "Rate Monotonic Analysis", we give **P1** the highest priority, followed by 
**P2** and, lastly, **P3**. Then, to understand all the interactions between the periods, we construct a
timeline that is equal in length to the least-common-multiple of the process periods, which is 12 in this 
case. The timeline is shown below:

        |              +----+    +----+              +----+
    P3  |              | P3 |    | P3 |              | P3 |
        |              +----+    +----+              +----+
        |    +---------+              +---------+         
    P2  |    |   P2    |              |   P2    |         
        |    +---------+              +---------+             
        |----+              +----+              +----+     
    P1  | P1 |              | P1 |              | P1 |     
        |----+              +----+              +----+     
        +----------------------------------------------------------------
        0    1    2    3    4    5    6    7    8    9    10    11    12

While all three periods start at time 0, due to **P1** having the lowest execution time, it becomes the job 
with the highest priority and gets to execute immediately. After one time unit, **P1** finishes and goes out 
of the *Ready* state. At time 1, **P2** starts executing as it's the highest-priority *Ready* process. At 
time 3, **P2** finishes and **P3** starts executing. **P1**'s next iteration starts at time 4, at which point 
it interrupts **P3**. **P3** gets one more time unit of execution between the second iteration of **P1** and 
**P2**, but **P3** does not get to finish until after the third iteration of **P1**.

An important concept in determining priority assignment rules is that of the *critical instant* for a job. A
*critical instant* is generally an instant at which a request for that job will have the longest response
time. This leads us to this theorem (proof in [1]):
> **Theorem 1:** A critical instant for any job occurs whenever the job is requested simultaneously with requests for all higher priority job.  
>  *&mdash;  Liu and Layland [1]*

## Achievable CPU Utilization
First, we define the **CPU Utilization** to be: "the fraction of processor time spent in the execution of the
job set." Assuming $T_i$ is the request time and $C_i$ is the amount of processor time spent in
executing a job $\tau_i$, then $C_i/T_i$ is the fraction of total processor time spent in executing $\tau_i$.
This means that: for $m$ jobs, the utilization factor is given by the rule:

$$U = \sum_{i=1}^{m}(C_i/T_i)$$

Although the utilization factor can be improved by increasing the values of the $C_i$s or by decreasing the 
values of $T_i$, this method is upper-bounded by the requirement that all jobs satisfy their deadlines at their
*critical instant*s.

> **THEOREM 4:** For a set of m jobs with fixed priority order, and the restriction that the ratio between any two request periods is less than 2, the least upper bound to the processor utilization factor is:  
> $U = m(2^{1/m} - 1)$  

Proof:
Let $\tau_1, \tau_2, ..., \tau_m$ denote the $m$ jobs. Let $C_1, C_2, ..., C_m$ be the run-times of the jobs
that fully utilize the processor and minimize the processor utilization factor. Assume that $T_m > T_{m-1} > ... > T_2 > T_1$.
Let $U$ denote the processor utilization factor. We wish to show that
$$C_1 = T_2 - T_1$$
Suppose that
$$C_1 = T_2 - T_1 + \Delta, \qquad \Delta > 0$$
Let
$$\begin{array}{c}
	C_1' \qquad= T_2 - T_1\\
	C_2' \qquad= C_2 + \Delta\\
	C_3' \qquad= C_3\qquad\\
	.\\
	.\\
	C_{m-1}' \quad= C_{m-1}\quad\\
	C_m' \qquad= C_m\qquad
\end{array}$$
Clearly, $C_1', C_2', ..., C_{m-1}', C_m'$ also fully utilize the processor. Let $U'$ denote the corresponding
utilization factor. We have
$$U - U' = (\Delta/T_1) - (\Delta/T_2) > 0$$
Alternatively, suppose that
$$C_1 = T_2 - T_1 - \Delta, \qquad \Delta > 0$$
Let
$$\begin{array}{c}
C_1'' \qquad= T_2 - T_1\\
\,\,\,C_2'' \qquad= C_2 - 2\Delta\\
C_3'' \qquad= C_3\qquad\\
.\\
.\\
C_{m-1}'' \quad= C_{m-1}\quad\\
C_m''  \qquad= C_m\qquad
\end{array}$$

Again, $C_1'', C_2'', ..., C_{m-1}'', C_m''$ fully utilize the processor. Let $U''$ denote the corresponding
utilization factor. We have
$$U - U'' = -(\Delta/T_1) + (2\Delta/T_2) > 0$$
Therefore, if indeed $U$ is the minimum utilization factor, then
$$C_1 = T_2 - T_1$$
In a similar way, we can show that
$$\begin{array}{c}
C_2 \qquad= T_3 - T_2\\
C_3 \qquad= T_4 - T_3\\
.\\
.\\
\,\,\qquad C_{m-1} \quad= T_m - T_{m-1}\quad
\end{array}$$

Consequently,
$$ C_m = T_m - 2(C_1 + C_2 + ... + C_{m-1})$$
To simplify the notation, let
$$g_i = (T_m - T_i)/T_i, \qquad i = 1, 2, ..., m$$
Thus
$$ C_i = T_{i+1} - T_i = g_iT_i - g_{i+1}T_{i+1}, \qquad i = 1, 2, ..., m - 1$$
and
$$ C_m = T_m - 2g_1T_1$$
and finally,
$$U     = \sum_{i=1}^{m}(C_i/T_i) = \sum_{i=1}^{m-1}[g_i - g_{i+1}(T_{i+1}/T_i)] + 1 - 2g_1(T_1/T_m)$$
$$      = \sum_{i=1}^{m-1}[g_i - g_{i+1}(g_i + 1)/(g_{i+1} + 1)] + 1 - 2[g_1/(g_1 + 1)]$$
$$\qquad\quad\,\,\,= 1 + g_1[(g_1-1)/(g_1 + 1)] + \sum_{1=2}^{m-1}g_i[(g_i - g_{i-1})/(g_i + 1)] \qquad(1)$$ 

From the equation shown above, we can see that the utilization bound becomes 1 if $g_i = 0$, for all $i$. To
find the least upper bound to the utilization factor, the equation needs to be minimized over the $g_i$'s.
This can be done by setting the first derivative of $U$ with respect to each of the $g_i$'s equal to zero, and
solving the resultant difference equations:
$$\partial U/\partial g_i = (g_i^2 + 2g_i - g_{i-1})/(g_i+1)^2 - (g_{i+1})/(g_{i+1} + 1) = 0,$$
$$\qquad\qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad \qquad i = 1, 2, ..., m-1 \qquad (2)$$

The definition $g_0 = 1$ has been adopted for convenience. The general solution for eqs. (2) can be shown to
be
$$\qquad g_i = 2^{(m-i)/m} - 1, \qquad \qquad \qquad \qquad \qquad i = 0, 1, ..., m-1 \qquad (3)$$
It follows that
$$ U = m(2^{1/m} - 1),$$
which is the relation we desired to prove.
Using this relation, we can calculate the CPU utilization factor for any number of jobs.

    +----------------+------------------+
    |    Job Count   | CPU Utilization  |
    +----------------+------------------+
    |       1        |       1.00       |
    +----------------+------------------+
    |       2        |       0.83       |
    +----------------+------------------+
    |       3        |       0.78       |
    +----------------+------------------+
    |       ∞        |  ln(2)  ≃  0.70  |
    +----------------+------------------+

In the above table, it is shown that the only time in which the utilization's upper bound is $100\%$ is when 
there is only one job (which is not a practical case).

## Improvements

While 70% CPU utilization doesn't seem much, it is more about how having "rate-monotonic scheduling" as a
base algorithm, and building upon it, can provide substantial utilization boosts. An example of that is [3],
where the author of the paper tried mixing two algorithms together; one of which is the "rate-monotonic scheduling". The
other algorithm is the "Deadline Driven Scheduling" algorithm; which was also introduced by Liu and Layland 
in [1]. Details about the mixed algorithm and the proof of how the utilization is calculated can be found in
[1] and [3]. An example with arbitrary jobs was provided in [1]. It showed that in a scenario where 
"rate-monotonic scheduling" would give an utilization of $78.3\%$, the mixed algorithm far surpasses it with
$98.3\%$.

## Summary
In this research, we talked about why scheduling algorithms are needed and introduced one of the most famous 
algorithms (rate monotonic scheduling) and gave a brief description of what it's all about. We also showed
the equation that is used to calculate the single-core CPU utilization for a given number of jobs, and walked
through how this equation was derived. Moreover, we mentioned a couple of papers in which the base algorithm
was improved upon by mixing it with another one, mainly the "Deadline Driven Scheduling" algorithm. This
merging process led to a huge utilization boost on single-core CPUs by more than 20% for the given example.
While some of the details were not provided directly, papers were mentioned that included all of those extra
bits.

## References
<p style="font-size:12px;">
<span style="font-weight:bold;">[1]</span> 
Liu, Chung Laung, and James W. Layland. 
<a style="color:blue;" href="https://doi.org/10.1145/321738.321743">
"Scheduling algorithms for multiprogramming in a hard-real-time environment." 
</a> 
Journal of the ACM (JACM) 20, no. 1 (1973): 46-61.
</p>
<p style="font-size:12px;">
<span style="font-weight:bold;">[2]</span> 
Marilyn Wolf, "Processes and Operating Systems," in
<a style="color:blue;font-style:italic" href="https://www.amazon.com/Computers-Components-Principles-Computing-Architecture/dp/0128053879">
Computers as Components: Principles of Embedded Computing System Design, 4th Edition.
</a> 
Morgan Kaufmann: The Morgan Kaufmann series, 2016.
</p>
<p style="font-size:12px;">
<span style="font-weight:bold;">[3]</span> 
Naghibzadeh, Mahmoud. 
<a style="color:blue;" href="https://ieeexplore.ieee.org/abstract/document/1000064/">
"A modified version of rate-monotonic scheduling algorithm and its' efficiency assessment." 
</a> 
In Proceedings of the Seventh IEEE International Workshop on Object-Oriented Real-Time Dependable Systems.(WORDS 2002), pp. 289-294. IEEE, 2002.
</p>
