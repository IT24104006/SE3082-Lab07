EXERCISE 7: SUMMARY COMPARISON

1. Comparison table

Program        | Collectives used         | Full array on every process? | Root manual sum loop? | Where the result is available
sum_bcast      | Bcast + Send/Recv        | Yes (every rank allocates N) | Yes (Recv loop)       | Root only
sum_scatter    | Scatter + Send/Recv      | No (root only; others chunk) | Yes (Recv loop)       | Root only
sum_gather     | Scatter + Gather         | No (root only; others chunk) | Yes (loop over sums)  | Root only
sum_reduce     | Scatter + Reduce         | No (root only; others chunk) | No                    | Root only
sum_allreduce  | Scatter + Allreduce      | No (root only; others chunk) | No                    | All processes (same value)
sum_scan       | Scatter + Scan           | No (root only; others chunk) | No                    | Different on each rank (prefix sum); last rank holds the total

2. Timings (average of 5 runs, seconds; see timings.txt)

Program        | 2 procs | 4 procs | 8 procs
sum_bcast      | 0.0029  | 0.0065  | 0.0073
sum_scatter    | 0.0022  | 0.0017  | 0.0013
sum_gather     | 0.0016  | 0.0015  | 0.0014
sum_reduce     | 0.0018  | 0.0015  | 0.0014
sum_allreduce  | 0.0021  | 0.0014  | 0.0012
sum_scan       | 0.0017  | 0.0014  | 0.0013

Fastest: the Scatter-based programs, and Reduce/Allreduce/Scan at 8 processes.
sum_bcast is clearly the slowest at 4 and 8 processes. Bcast sends the whole
array (1,000,000 ints) to every process, so the data moved and the memory used
grow with the number of processes, and every rank allocates the full array.
Scatter sends each process only N/P elements, so the work shrinks as processes
are added. The differences among the five Scatter-based programs are about
0.0001 to 0.0004 sec, which is within run-to-run noise on a laptop, so they
cannot be ranked with confidence. Reduce, Allreduce and Scan use tree-based
algorithms (O(log P) steps) instead of a loop of sends, which is the expected
reason they are slightly ahead at 8 processes.

3. Thinking question: MPI_Scan vs MPI_Allreduce

Choose MPI_Scan when each process needs a DIFFERENT cumulative result, not the
same total. Example: every process filters its own chunk of records and keeps a
different number of them. To write the kept records into one global output
array with no gaps, each process needs its starting position, which is the sum
of the kept counts of all lower ranks. MPI_Scan gives that as sum_before_me
(prefix_sum - local count) in one call. MPI_Allreduce would only give the
overall total, which does not tell a process where its own data starts.
