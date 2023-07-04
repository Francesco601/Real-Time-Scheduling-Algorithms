#  Scheduling Algorithms in Real-Time Operating Systems

CPU schedling for real-time Operating Systems involves certain special considerations. In general, there is a distinction to be made between soft real-time systems and hard real time systems. **Soft real-time systems** provide no guarantee as to when a critical real time process will  be scheduled.  They guarantee only that the process will be given priority over non-critical processes. **Hard real-time systems** have stricter requirements. A task must be serviced by its deadline. Service after a deadling has expired is the same as no service at all.

Real time systems are event-driven. The system is typically waiting for a real time event to occur. Events may arise either in software (e.g. a timer has expired) or in harware (e.g.  a remote controlled vehicle detects that it is approaching an obstruction). When an event occurs, the system must respond to and service it as quickly as possible. **Event latency**  is the amount of time that elapses from when an event occurs to when it is serviced.  Two types of latency affect the performance of real-time systems: *interrupt latency* and *dispatch latency*.  Interrupt latency refers to the time from the arrival of an interrupt at the CPU to the start of the routine that services the interrupt. When an interrupt occurs, the OS must first complete the instruction it is executing and determine the type of interrupt that occured. It must then save the state of the current process before servicing the interrupt using the specific Interrupt Service Routine (ISR).  The total time required to carry out these tasks is the interrupt latency. It is obviously important for real-time operating systems to minimize interrupt latency to ensure that real time tasks receive immediate attention. In fact, for hard real-time systems, it's not only necessary to minimize latency, but latency must be bounded to meet the strict requirements of these systems.

The amount of time required for the schedling dispatcher to stop one process and start another is known as dispatch latency. The goal of giving processes immediate access to the CPU mandates minimizing this kind of latency as well. In sum, the most important feature of a real-time operating system is to respond as quickly as possible to a real-time process as soon as that process requires access to the CPU. As a consequence,  the scheduler for a real-time operating system must support a priority based algorithm with preemption. But providing  a preemptive, priority-based scheduler guarantees only soft real-time functionality. Hard real-time systems must further guarantee that real-time tasks wil be served in accordance with their deadline requirements, and making such guarantees requires additional scheduling features. From this point on, I will discuss only scheduling algorithms appropriate  for hard real-time systems.

Before discussing the algorithms,  we must define certain characteristics  of the processes that are to be scheduled. First, the processes are considered **periodic**. They require the CPU at constant intervalls (periods). Once a periodic process has acquired the CPU, it has a fixed processing time *t*, a deadline *d* by which is must be serviced by the CPU, and a period *p*. The relationship among these quantities can be expressed as 0 <= t <= d <= p.  The **rate** of a process is 1/p.  What's unusual about this form of schedling is a process might have to announce its deadline requirements to the scheduler. Then, using a technique known as an "admission control" algorithm, the scheduler does one of two things. It either admits the process, guaranteeing that the process will comlete on time, or it rejects the request as impossible if it can't guarantee that the process will be serviced by its deadline.

## Rate Monotonic Scheduling

The rate-monotonic scheduling algorithm schedules period tasks using a static priority-driven policy with preemption. If a lower-priority process is running and a higher-priority process becomes available to run, it will preempt the lower-priority process. Upon entering the system, each periodic task is assigned an priority inversely proportional to its period. The shorter the period, the higher the priority and viceversa. A set of processes can be scheduled only if they meet the following equation:

```math
\sum_{k=1}^n C_i/T_i \leq n(2^{1/n} - 1)
```
where n is the number of processes in the process set, $C_i$ is the computation time of the process, $T_i$ is the time period for the process to run and U is the processor utilization. Let's consider a simplified example first without rate-monotonic scheduling.  We have two processes, P1 and P2. The periods for P1 and P2 are 50 and 100 respectively. The processing times are T1 = 20 for P1 and T2 = 35 for P2. The deadline for each process requires that it complete its CPU burst by the start of its next period.

First we have to find out whether it's possible to schedule these tasks so that each meets its deadline. If we measure  the CPU utilization of a process $P_i$ as the ratio of its burst to its period-- $T_i/P_i$-- the CPU utilization of  of $P_1$ is 20/50 = 0.40 and that of $P_2$ is 35/100 = 0.35 for a total CPU utilization of 75 percent. Therefore, it seems we can schedule these tasks in such a way that both meet their deadlines and still leave the CPU with available cycles. But suppose we assign $P_2$ a higher priority than $P_1$. The execution of $P_1$ in this situation is different. $P_2$ starts execution first and completes at time 35.  At this point $P_1$ starts; it completes its CPU burst at time 55. However, the first deadline for $P_1$  was at time 50, so the scheduler has caused $P_1$ to miss its deadline.

Now suppose we use rate-monotonic schedling, in which we assign $P_1$ a higher priority than $P_2$  because the period of $P_1$ is shorter than that of $P_2$. In this case, $P_1$ starts first and completes its CPU burst at time 20, thereby meeting its first deadline. $P_2$ starts at this point and  runs until time 50. At this time, it is preempted by $P_1$, although it still has 5 milliseconds remianing in its CPU burst. $P_1$  completes its CPU burst at time 70, at which point the scheduler resumes $P_2$. $P_2$  completes its CPU burst at time 75, also meeting its first deadline. The system is idle until time 100, when $P_1$ is scheduled again. Rate-monotonic scheduling is considered optimal in that if a set of processes cannot be  scheduled by this algorithm, it cannot be assigned by any other algorithm that assigns static priorities.

Let's try another example with a set of processes that can't be scheduled using the rate-monotonic algorithm. Assume that process $P_1$  has a period of $p_1 = 50$  and a CPU burst of $t_1=25$. For $P_2$ , the corresponding values are $p_2=80$ and $t_2=35$. Rate-montonic scheduling would assign $P_1$ a higher priority, since it has the shorter period. The total CPU utilization of the two processes is (25/0) + (35/80) = 0.94. and it therefore seems logical that the two processes could be scheduled and still leave the CPU with 6 percent available time. Initially, $P_1$  runs until it completes its CPU burst time at time 25. $P_2$ then starts running and runs until time 50, when it is preempted by $P_1$. At this point, $P_2$ still has 10 milliseconds remaining in its CPU burst. Process $P_1$ runs until time 75, therefore $P_2$ finishes its burst at time 85, after the deadline for completion of its CPU burst at time 80.

Despite being optimal, rate-monotonic scheduling has a serios limitation. CPU utilization is bounded, and it is not always possible to maximize CPU resources fully. With one process in the system. CPU utilization is 100 percent but it falls to 69 precent as the number of proceses approaches infinity. With two processses, CPU utilization is bounded at about 83 percent. Combined CPU utilization for thr two processes in our first example  is 75%. so the rate-monotice scheduing algorithm is guaranteed to schedule them  so that they meet their deadlines. For the two processes in our second example, combined utilization  is approximately  94 percent: therefore, rate monotonic scheduing cannot guarantee that they be scheduled to meet their deadlines.

## Earliest Deadline First Scheduling

Earliest deadline first scheduing  assigns priorities dynamically  according to deadline. The earlier the deadline, the higher the priority; the later the deadline the lower the priority.  Under the EDF policy, when a policy becomes runnable, it must announce its deadline requirements to the system. Priorities may have to be adjusted to reflect the deadlines of the newly runnable processes. Notice how this differs from rate-monotonic scheduling, where priorities are fixed. To illustrate EDF scheduling , we again schedule the processes of our last example,  which failed to meet deadline requirements under rate-monotonic scheduing.  Recall that $P_1$  has values of $p_1$ = 50 and $t_1$ = 25 and that $P_2$ has values of $p_1$ = 80 and $t_1$ = 35. Under EDF scheduling, process $P_1$ has the earliest deadline, so its initial priority is higher than that of $P_2$. Process $P_2$  starts running at the end of the CPU burst for $P_1$. However, whereas rate monotonic scheduling allows $P_1$ to preempt $P_2$  at the beginning of its next period at time 50, EDF allows $P_2$ to continue running. $P_2$ now has a higher priority than $P_1$ becuase its next deadline (at time 80)  is earlier than that of $P_1$ (at time 100). Thus, both $P_1$ and %P_2$ meet their first deadlines.  Process $P_1$ again begings running at time 60 and completes its second CPU burst at time 85, also meeting its second deadline at time 100. $P_2$ begings running at this point, only to be prempted by $P_1$ at the start of its next period at time 100. $P_2$ is preempted because $P_1$ has an earlier deadline (time 150) than $P_2$ (time 180).  At time 125, $P_1$ completes its CPU burst  and $P_2$ resumes execution, finishing ar time 145 and meeting its deadline as well The system is idle until time  150, when $P_1$ is schduled to run again.

Unlike rate-monotonic scheduling, EDF scheduling does not require that processes be periodic, nor must a process  require a constant amount of CPU time per burst. The only requirement  is that a process  announce its deadline to the scheduler whenever it becomes runnable. The appeal of EDF schedling is that it is theoretically optimal--theoretically it can schedule processes so that each process meets its deadline requirements and CPU utilization is at 100 percent. In practice, however,  it is impossible to achieve this level of CPU utilization due to the cost of context switching between processes and interruopt handling.

## References and related Resources

<a href="https://github.com/Francesco601/BEST-Operating-System-Resources"> Operating System collection  </a>



















