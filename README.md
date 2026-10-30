# Experiment 03: Analyze Synchronization Mechanisms in Parallel Programs

## 1. Objective

To implement and study common synchronization mechanisms used in shared-memory parallel programs:
- Unsynchronized shared-variable update (race condition)
- OpenMP critical section
- OpenMP atomic operation
- OpenMP barrier
- POSIX semaphore
- OpenMP lock

The experiment evaluates correctness and execution time as the number of threads increases.

## 2. Software and environment

| Tool | Purpose |
|---|---|
| VMware Workstation Pro | Hosts the Ubuntu virtual machine |
| Ubuntu Linux | Operating system used to compile and execute the programs |
| GCC (`build-essential`) | C compiler and build tools |
| OpenMP (`-fopenmp`) | Parallel loops and synchronization directives |
| POSIX threads (`-pthread`) | Creates and manages threads in the semaphore program |
| POSIX semaphore library | Provides `sem_wait`, `sem_post`, and semaphore initialization/destruction |
| Ubuntu Terminal | Creates, compiles, runs programs, and records results |

The work was carried out in the Ubuntu VM whose terminal prompt identified the host as `master-mpi`. The experiment directory used was:

```text
/home/sukanya-m-s/parallel_lab/synchronization
```

## 3. Theory

### 3.1 Race condition

A race condition occurs when multiple threads access and modify shared data without suitable coordination. `counter++` is a read-modify-write operation; concurrent updates can interfere and cause lost increments. In C, an unsynchronized data race is undefined behavior.

### 3.2 Critical section

OpenMP's `#pragma omp critical` ensures that only one thread at a time executes the protected region associated with that critical construct. This preserves correctness but can cause waiting and contention.

### 3.3 Atomic operation

OpenMP's `#pragma omp atomic` makes a supported memory operation, such as `counter++`, atomic. It is suitable for simple updates and does not protect an arbitrary multi-statement block.

### 3.4 Barrier

`#pragma omp barrier` is a synchronization point. Every thread in the team must reach the barrier before any thread can proceed beyond it. A barrier coordinates phases; it does not by itself protect a shared variable from data races.

### 3.5 Semaphore

A POSIX semaphore controls access using a counter. In this experiment, a semaphore initialized to one is used as a binary semaphore: `sem_wait()` acquires it and `sem_post()` releases it, allowing one thread at a time into the protected counter update.

### 3.6 Lock

An OpenMP lock is explicitly initialized, acquired before the shared update, released afterward, and destroyed when no longer needed. It provides mutual exclusion similar to a critical section, with explicit lock management.

## 4. Programs implemented

| Program | Source file | What it demonstrates |
|---|---|---|
| Program 1 | `race.c` | OpenMP parallel loop updating a shared counter without synchronization |
| Program 2 | `critical.c` | Protecting the counter increment with `#pragma omp critical` |
| Program 3 | `atomic.c` | Protecting the increment with `#pragma omp atomic` |
| Program 4 | `barrier.c` | Printing before and after an OpenMP barrier to demonstrate phase synchronization |
| Program 5 | `semaphore.c` | Two POSIX threads each perform 100,000 increments while protected by a binary semaphore |
| Program 6 | `compare_sync.c` | Comparing None, Critical, Atomic, and OpenMP Lock for 1, 2, 4, and 8 threads |

### Program 1 — Race condition (`race.c`)

The program initializes `counter` to zero and executes ten million loop iterations using `#pragma omp parallel for`. Each iteration performs `counter++` without protection. The output may be less than the expected 10,000,000 because updates can be lost. The result can vary between runs.

### Program 2 — Critical section (`critical.c`)

The counter increment is enclosed in:

```c
#pragma omp critical
{
    counter++;
}
```

Only one thread at a time can execute this protected region, so the final count should be 10,000,000.

### Program 3 — Atomic operation (`atomic.c`)

The increment is preceded by:

```c
#pragma omp atomic
counter++;
```

This makes the supported increment atomic and should produce the correct final count of 10,000,000.

### Program 4 — Barrier (`barrier.c`)

Each OpenMP thread prints its thread number before the barrier, waits at `#pragma omp barrier`, and then prints after the barrier. The ordering of messages can vary, but no thread can pass the barrier until all threads in the team have arrived.

### Program 5 — Semaphore (`semaphore.c`)

The semaphore is initialized with value one using `sem_init(&semaphore, 0, 1)`. Two POSIX threads execute the worker function. Each performs 100,000 increments, acquiring the semaphore with `sem_wait()` before the update and releasing it with `sem_post()` afterward. `pthread_join()` waits for both threads. The expected final count is 200,000.

### Program 6 — Comparison benchmark (`compare_sync.c`)

The benchmark defines `N` as 10,000,000 and tests four methods:
- None: no synchronization
- Critical: OpenMP critical region
- Atomic: OpenMP atomic increment
- Lock: OpenMP lock acquired and released around each increment

Each method is tested with 1, 2, 4, and 8 threads. `omp_get_wtime()` measures elapsed wall-clock time. The final count is checked against `N` to report `Correct` or `INCORRECT`.

## 5. Commands used and their purpose

Run these commands in the Ubuntu terminal. The commands below document the workflow used for this experiment.

| Command | Role / why it was used |
|---|---|
| `whoami` | Displays the current Ubuntu username. |
| `pwd` | Displays the current working directory. |
| `gcc --version` | Checks the installed GCC compiler version. |
| `echo '#include <omp.h>' \| gcc -E -fopenmp -xc - > /dev/null && echo "OpenMP OK"` | Checks that GCC can preprocess a small test including the OpenMP header with OpenMP enabled. |
| `mkdir -p ~/parallel_lab/synchronization` | Creates the experiment directory and any missing parent directories. |
| `cd ~/parallel_lab/synchronization` | Changes the terminal's working directory to the experiment folder. |
| `nano race.c` | Creates or edits the race-condition C source file in Nano. The same `nano` command was used for the other `.c` files. |
| `gcc race.c -fopenmp -o race` | Compiles `race.c` with OpenMP enabled and names the executable `race`. |
| `gcc critical.c -fopenmp -o critical` | Compiles the critical-section program. |
| `gcc atomic.c -fopenmp -o atomic` | Compiles the atomic-operation program. |
| `gcc barrier.c -fopenmp -o barrier` | Compiles the barrier program. |
| `gcc semaphore.c -pthread -o semaphore` | Compiles the POSIX-thread/semaphore program with pthread support. |
| `gcc compare_sync.c -fopenmp -o compare_sync` | Compiles the comparison benchmark. |
| `./race` | Runs the race-condition executable. |
| `./critical` | Runs the critical-section executable. |
| `./atomic` | Runs the atomic-operation executable. |
| `OMP_NUM_THREADS=4 ./barrier` | Runs the barrier program with four OpenMP threads for this command. |
| `./semaphore` | Runs the POSIX semaphore executable. |
| `./compare_sync` | Runs all benchmark combinations for 1, 2, 4, and 8 threads. |
| `./compare_sync \| tee comparison_results.txt` | Runs the benchmark, displays its output, and saves a copy in `comparison_results.txt`. It executes the benchmark again, so the timings and race-condition counts may differ from the earlier run. |
| `ls` | Lists files in the current directory. |
| `ls *.c` | Lists C source files whose names end in `.c`. |

### Important command options

- `gcc`: GNU C compiler.
- `-fopenmp`: enables OpenMP directives and links the OpenMP runtime.
- `-pthread`: enables the compiler and linker options needed for POSIX threads.
- `-o NAME`: chooses the output executable name.
- `./NAME`: executes a program from the current directory.
- `|`: sends the output of one command to another command.
- `tee FILE`: displays output and writes a copy to a file.
- `OMP_NUM_THREADS=4`: sets the OpenMP default thread count to four for that command.

In Nano, `Ctrl+O` saves the file, `Enter` confirms the filename, and `Ctrl+X` exits the editor.

## 6. Actual benchmark results

The following measurements are reproduced from the benchmark output supplied for this experiment. The benchmark performs 10,000,000 increments for each method and thread count. Times are in seconds.

| Method | Threads | Final count | Correctness | Time (s) |
|---|---:|---:|---|---:|
| None | 1 | 10,000,000 | Correct | 0.0084 |
| Critical | 1 | 10,000,000 | Correct | 0.1135 |
| Atomic | 1 | 10,000,000 | Correct | 0.0669 |
| Lock | 1 | 10,000,000 | Correct | 0.1119 |
| None | 2 | 5,272,170 | INCORRECT | 0.0112 |
| Critical | 2 | 10,000,000 | Correct | 0.3401 |
| Atomic | 2 | 10,000,000 | Correct | 0.1641 |
| Lock | 2 | 10,000,000 | Correct | 0.3700 |
| None | 4 | 4,792,950 | INCORRECT | 0.0062 |
| Critical | 4 | 10,000,000 | Correct | 0.3238 |
| Atomic | 4 | 10,000,000 | Correct | 0.1623 |
| Lock | 4 | 10,000,000 | Correct | 0.3784 |
| None | 8 | 5,671,941 | INCORRECT | 0.0084 |
| Critical | 8 | 10,000,000 | Correct | 0.3260 |
| Atomic | 8 | 10,000,000 | Correct | 0.1633 |
| Lock | 8 | 10,000,000 | Correct | 0.3919 |

### Results interpretation

1. **None:** The one-thread run produced the expected count. The multi-thread runs produced incorrect counts, demonstrating lost updates due to the race condition. Its short time must not be considered the speed of a correct implementation.
2. **Critical:** All tested runs produced the correct count. The time increased substantially with multiple threads because they compete to enter the same critical region.
3. **Atomic:** All tested runs produced the correct count. In these measurements, Atomic was faster than Critical and Lock at each tested thread count.
4. **Lock:** All tested runs produced the correct count. Its measured time increased as more threads competed for the lock.
5. **Barrier and Semaphore:** These were demonstrated in separate programs and are not rows in the comparison benchmark.

The timings are specific to this VM and run. They can vary with CPU allocation, system load, scheduling, and repeated execution. The unsynchronized C data race has undefined behavior, so its result is not guaranteed.

## 7. Observation

The experiment showed that unsynchronized access to a shared counter can produce incorrect results when multiple threads update it concurrently. Critical sections, atomic operations, and OpenMP locks maintained the expected final count. In the measurements collected, atomic operations had lower execution times than critical sections and locks. Increasing the number of threads did not improve the synchronized shared-counter workload because the threads competed to update the same variable.

The barrier program demonstrated how threads can be synchronized between phases of execution. The semaphore program demonstrated mutual exclusion using POSIX thread synchronization.

## 8. Conclusion

The experiment successfully implemented and studied race conditions, critical sections, atomic operations, barriers, semaphores, and OpenMP locks. It demonstrated that synchronization is essential for correctness in shared-memory parallel programs. However, synchronization introduces overhead, and contention can limit scalability when many threads compete for one shared resource. In the measured benchmark, Atomic provided the best execution time among the synchronized counter-update methods tested.

## 9. Screenshots

The `Screenshots/` folder contains the screenshots supplied with this project archive. Add any further terminal screenshots there, such as:
- Race-condition program compilation and output
- Critical-section program output
- Atomic program output
- Barrier output using four threads
- Semaphore output
- Full comparison benchmark output

Use the screenshots as evidence of the commands executed and their results.
