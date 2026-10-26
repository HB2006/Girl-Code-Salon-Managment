# Smart Salon Management: Assignment 1
**GROUP Members: Ananya Agarwal, Bhumi Bhalekar, Himanshi Bansal, Saanvi Gopalka** 

 ---

## 1. Project Overview

This project models a salon as a simplified Operating Systems process-scheduling problem. Each customer is represented as a **process**, and the salon has **two identical barbers**, which act like two parallel execution slots. Customers arrive at different times, wait if both barbers are occupied, receive service according to a selected scheduling algorithm and eventually finish.

The program is a **deterministic discrete-event simulation**. It uses a simulated clock; it does not make the computer wait in real time while a haircut is being performed. Given the same valid input file and algorithm, it should produce the same result each time.

Our focus is **process management**. Memory allocation, locks, semaphores, race conditions, deadlocks, and other concurrency or memory-management features are outside this submission's scope.

### What the program supports

- Two parallel barbers.
- Four scheduling algorithms: FCFS, Priority, Preemptive Priority and Round Robin.
- Input from a text file in the assignment's standardized format.
- Per-customer completion, waiting, turnaround and response times.
- Summary metrics: average waiting time, average turnaround time, average response time, throughput and barber utilization.
- Test input files, expected-output files, invalid-input cases, and a shell script that checks the supplied test set against all four algorithms.

---

## 2. Process Model

Each customer is stored as a `Process` structure in `schedule.c`. The structure contains the following information:

| Field        | Meaning |
|--------------|---------|
| `id`         | Unique customer identifier, e.g. `C1` or `V0`. |
| `arrival`    | Simulation time at which the customer enters the salon and becomes eligible for service. |
| `priority`   | Numerical scheduling priority. **A smaller number means a higher priority.** |
| `service`    | Total service time required for the haircut (acts like a CPU burst in an Operating Systems scheduler). |
| `category`   | Descriptive customer category, e.g. `REGULAR` or `VIP`. |
| `start`      | First time the customer starts receiving service. It does not change if the customer is later preempted. |
| `completion` | Time at which the customer finishes all required service. |
| `waiting`    | Total time the customer spends waiting rather than receiving service. |
| `turnaround` | Total time from arrival until completion. |
| `response`   | Time from arrival until the first start of service. |

The category field records scenario information, but **the schedulers use the numerical `priority` field, not the category text, to make priority decisions**. For example, a `VIP` customer is not automatically given higher priority unless the input record assigns that customer a smaller numerical priority.

### Process lifecycle

The simulation follows this conceptual lifecycle:

`NEW → READY → RUNNING → TERMINATED`

- **NEW:** The customer has not yet arrived.
- **READY:** The customer has arrived and is waiting for a barber.
- **RUNNING:** The customer is receiving service from a barber.
- **TERMINATED:** The customer has completed the required service.

In Preemptive Priority and Round Robin, a customer who is interrupted before completing service goes back to the waiting set with the remaining service time preserved. The code does not create a separate operating-system process for each customer; these states are represented through arrays, queue membership, remaining-time values, and the running-barber slots.

### The two barbers

Both barbers are treated as identical. Up to two customers can be served at the same simulated time. If a barber is free and an eligible customer is waiting, the selected scheduling policy determines who receives service.

---

## 3. Input File Format

Every input file follows this structure:

```text
N
Q
ID,ARRIVAL,PRIORITY,SERVICE,CATEGORY
ID,ARRIVAL,PRIORITY,SERVICE,CATEGORY
...
```

Where:

- `N` is the number of customers in the file.
- `Q` is the Round Robin time quantum. It is required in the input format even when the selected algorithm is not Round Robin.
- `ID` is a customer identifier.
- `ARRIVAL` is the customer's arrival time.
- `PRIORITY` is the numerical priority; a smaller value means higher priority.
- `SERVICE` is the total haircut/service duration.
- `CATEGORY` is a label such as `REGULAR` or `VIP`. It must not contain commas.

Further assignment rules:
- `N`, `Q`, service times, and priorities must be positive integers;
- arrival times must be non-negative integers;
- IDs must be unique within a file.
- Multiple customers may share an arrival time, priority, or service time.

### Included input files

| File                           | Main purpose |
|--------------------------------|--------------|
| `01_sample.txt`                | General example used to compare the algorithms. |
| `02_single.txt`                | One customer who arrives after the simulation's initial time. |
| `03_all_same_arrival.txt`      | Multiple customers arriving at the same time; tests deterministic tie-breaking. |
| `04_idle_gap.txt`              | A gap with no customers waiting, followed by later arrivals. |
| `05_equal_priority.txt`        | Equal numerical priorities and non-preemptive behaviour. |
| `06_vip_preempts.txt`          | A high-priority customer arriving while lower-priority customers are being served. |
| `07_rr_arrival_at_expiry.txt`  | New arrivals occurring when a Round Robin quantum expires. |
| `08_rr_service_vs_quantum.txt` | Service times shorter than, equal to, and longer than the Round Robin quantum. |
| `09_starvation.txt`            | A low-priority customer competing with a stream of higher-priority customers. |

The `expected/` directory contains the expected full output for each of the nine valid inputs under each of the four algorithms. The `invalid/` directory contains malformed inputs that should be rejected. `run_all.sh` uses these directories during automated testing.

---

## 4. How to Compile and Run

### Requirements

You need a C compiler such as GCC and a terminal. The source code uses standard C library functionality. To run the full test script, you also need Bash and Python 3.

### Compile

From the directory containing `schedule.c` and the input files, run:

```bash
gcc -std=c11 -Wall -Wextra -O2 schedule.c -o out
```

This compiles `schedule.c` and creates an executable named `out`. The compiler flags select the C11 language standard, enable common warnings, and apply optimization.

> **Submission note:** The project relies on the source code rather than a prebuilt executable. Compile `schedule.c` on the machine where you intend to run the program. The test script builds its own temporary executable named `schedule_bin` and removes it when the script finishes.

### Run one algorithm

The command-line interface is:

```bash
./out INPUT_FILE ALGORITHM
```

For example:

```bash
./out 01_sample.txt FCFS
./out 01_sample.txt PRIORITY
./out 01_sample.txt PRIORITY_PREEMPTIVE
./out 01_sample.txt RR
```

Use these algorithm names:

| Argument | Algorithm |
|---|---|
| `FCFS` | First-Come, First-Served |
| `PRIORITY` | Non-preemptive Priority Scheduling |
| `PRIORITY_PREEMPTIVE` | Preemptive Priority Scheduling |
| `RR` | Round Robin |

Note: The algorithm name is case-sensitive. `SJF` is not implemented because it is not one of the algorithms required for the Smart Salon scenario.

Note: If the number of command-line arguments is incorrect or the input cannot be read, the program prints an error to standard error and exits with a non-zero status. If an unsupported algorithm name is supplied, the program also prints an error.

### Windows note

If using GCC through MinGW on Windows, compile with:

```powershell
gcc -std=c11 -Wall -Wextra -O2 schedule.c -o out.exe
```

Then run, for example:

```powershell
.\out.exe 01_sample.txt FCFS
```

### Run the full test set

The provided `run_all.sh` script compiles `schedule.c` into a temporary executable named `schedule_bin`, runs every numbered input against all four supported algorithms, and checks the results. Run it from the project directory:

```bash
chmod +x run_all.sh
./run_all.sh
```

The script checks all 36 valid input/algorithm combinations against the corresponding files in `expected/`. It also runs the 15 files in `invalid/` with all four algorithm names and checks that each invalid input is rejected with a non-zero exit status and no standard output. The script exits with a non-zero status if any check fails.

To print the summary metrics for each valid run as well as the final pass/fail count, use:

```bash
./run_all.sh -v
```

To save this verbose output to `results.txt`, use `./run_all.sh -v > results.txt`. The existing `results.txt` is a saved summary snapshot; the script does not rewrite it unless you redirect the output yourself.

---

## 5. Scheduling Algorithms

The Smart Salon scenario requires the following four algorithms. The numerical priority, arrival time, and customer ID are used according to the rules below.

### 5.1 First-Come, First-Served (FCFS)

**Idea:** Customers are served in arrival order.

1. Choose the customer with the earliest arrival time.
2. If two customers arrived at the same time, choose the lexicographically smaller customer ID first. Lexicographic order is string order, so it is not necessarily numeric order (`C10`, for example, sorts before `C2`).
3. Assign that customer to the barber who becomes free earliest.
4. Once service begins, the customer continues until completion.

FCFS is **non-preemptive**. A later arrival cannot interrupt a customer who has already started service.

**In the code:** `cmp_fcfs()` defines the arrival-time and ID ordering; `run_fcfs()` assigns customers to the earliest-free barber.

### 5.2 Non-preemptive Priority Scheduling

**Idea:** Among customers who are ready when a barber becomes available, serve the highest-priority customer first.

The selection order is:

1. Smaller numerical priority value first.
2. If priorities are equal, earlier arrival first.
3. If arrival times are also equal, lexicographically smaller ID first.

A customer already receiving service is not interrupted, even if a higher-priority customer arrives. If no customer is ready when a barber becomes free, the simulation advances to the next arrival.

**In the code:** `better_priority()` implements the priority tie-breaking order and `run_priority()` performs the non-preemptive scheduling.

### 5.3 Preemptive Priority Scheduling

**Idea:** A newly ready customer with a strictly higher priority may interrupt a customer who is already receiving service.

- A smaller priority number means higher priority.
- Preemption only occurs if the waiting customer's priority number is strictly smaller than the running customer's priority number.
- Equal-priority arrivals do not cause preemption.
- A preempted customer returns to the waiting set with its remaining service time preserved.
- When both barbers are occupied, the customer selected for interruption is the running customer with the lowest priority. If the running priorities are equal, select the customer with the later arrival; if they are still tied, select the lexicographically larger ID.
- The simulation reevaluates the schedule as time advances and relevant arrivals or completions occur.

**In the code:** `run_priority_preemptive()` tracks remaining service time, the customers currently occupying the two barber slots, and whether each customer has started before. It advances the clock in one-unit steps, which is suitable for the integer time values used by the input format.

### 5.4 Round Robin (RR)

**Idea:** Each customer receives up to a fixed number of service time units, called the **time quantum**, before the scheduler considers another customer.

- The quantum is read from the second line of the input file.
- Customers who arrive are added to the back of the ready queue in arrival order; simultaneous arrivals are ordered by lexicographically smaller ID.
- When a customer finishes before the quantum expires, that customer leaves the system.
- If the quantum expires and service remains, the customer returns to the back of the ready queue.
- When new arrivals occur at exactly the same time as a quantum expiration, new arrivals are enqueued before the expired customer is reinserted.
- If both barbers' quantum periods expire at the same time, this implementation reinserts the expired customers in lexicographic ID order to make the simulation deterministic. The specification does not prescribe a tie-break for this particular case, so this is an implementation assumption.

**In the code:** `run_rr()` uses a circular array as the ready queue, tracks each customer's remaining service time, and tracks the remaining quantum for each running barber.

### Non-preemptive vs. preemptive scheduling

- **Non-preemptive:** Once service starts, the customer keeps the barber until the service completes.
- **Preemptive:** The scheduler may interrupt a customer, return them to the ready set, and let another customer use the barber. The interrupted customer's remaining service time is saved.
- **Round Robin:** This is also preemptive, but interruption happens because the time quantum expires, not because a higher-priority customer arrives.

---

## 6. Simulation Design and Assumptions

### Discrete-event simulation

The program uses simulated time rather than real delays. It does not call a sleep function to represent the duration of a haircut. Non-preemptive algorithms keep track of when each barber is free. Preemptive Priority and Round Robin advance time in one-unit increments and update the running customers' remaining service time.

If no customer is available and no barber is running a customer, the simulation can skip forward to the next arrival instead of waiting in real time.

### Deterministic behaviour

Determinism means that identical valid input and the same algorithm should produce identical output. The implementation supports this using explicit tie-breaking rules. The one additional assumption is the lexicographic ordering of two customers whose Round Robin quantum expires simultaneously, described in Section 5.4.

### Arrival and completion timing

Arrival times and service times are integer simulation units. A service time of `4` means four units of simulated service, not four seconds of real time. A customer can start only after arriving. A customer is complete when the remaining required service reaches zero.

### Categories and numerical priority

`REGULAR` and `VIP` are descriptive categories. The code does not convert these labels into priorities. The value in the `PRIORITY` column controls Priority and Preemptive Priority Scheduling.

### Fixed-size text fields

The current `Process` structure uses fixed-size character arrays for IDs and categories. The arrays are 32 characters long, including space for the terminating null character, so input values can contain up to 31 characters. `read_input()` rejects empty or overlong IDs and categories rather than silently truncating them.

---

## 7. Scheduling Metrics

The simulation reports metrics for every customer and averages selected metrics across all customers. Let:

- `Aᵢ` = arrival time of customer *i*.
- `Sᵢ` = the first time customer *i* starts service.
- `Cᵢ` = completion time of customer *i*.
- `Bᵢ` = required service time of customer *i*.
- `N` = number of completed customers.
- `M` = number of barbers. In this project, `M = 2`.
- `B` = total busy time summed across both barbers.
- `E` = elapsed simulation time from the first arrival until the last completion.

### Completion time

The time at which a customer finishes all required service. For a customer who is interrupted, this is the time service finally finishes, not the time of the first interruption.

### Turnaround time

The total time the customer spends in the system, from arrival to completion:

`Turnaroundᵢ = Cᵢ − Aᵢ`

It includes both waiting and service time.

### Response time

How long the customer waits between arriving and first receiving service:

`Responseᵢ = Sᵢ − Aᵢ`

For a preempted customer, response time is still calculated using the **first** start time. It does not increase each time the customer is interrupted.

### Waiting time

The total time the customer spends in the system without receiving service:

`Waitingᵢ = Turnaroundᵢ − Bᵢ`

For Preemptive Priority and Round Robin, this includes all waiting periods between service slices as well as the initial wait. It does not include time actually spent receiving service.

### Average waiting, turnaround, and response times

Each average is calculated by adding the metric for every customer and dividing by the number of customers. Lower values generally mean less time spent waiting or in the system, but the best policy depends on the metric and workload being considered.

### Throughput

Throughput measures how many customers finish per unit of simulated time:

`Throughput = N / E`

The unit is **customers per simulation time unit**. A larger value means more customers completed per unit of elapsed simulation time for that input.

### Barber utilization

Utilization measures the fraction of the total available barber time that was spent serving customers:

`Utilization (%) = B / (M × E) × 100`

There are two barbers, so the total available service capacity over an elapsed interval `E` is `2 × E`. Utilization is expressed as a percentage. Idle time lowers utilization. The calculation uses the period from the first arrival to the final completion, consistent with the assignment specification.

---

## 8. Program Output

For each customer, the program prints a comma-separated row containing:

`PROCESS_ID,ARRIVAL,PRIORITY,SERVICE,START,COMPLETION,WAITING,TURNAROUND,RESPONSE`

It then prints a `SUMMARY` section with:

- `AVERAGE_WAITING`
- `AVERAGE_TURNAROUND`
- `AVERAGE_RESPONSE`
- `THROUGHPUT`
- `UTILIZATION`

The customer rows are printed in lexicographic ID order. The category is not printed because it is not part of the assignment's standardized output row.

> **Standardized column header**  
> The current `print_results()` function in `schedule.c` prints the required column header before the customer rows:
>
> ```c
> printf("PROCESS_ID,ARRIVAL,PRIORITY,SERVICE,START,COMPLETION,WAITING,TURNAROUND,RESPONSE\n");
> ```
>
> The automated test script checks that this header is present and spelled correctly.

### Output example structure

The following shows the required structure; the values here are illustrative of the format rather than a complete run:

```text
PROCESS_ID,ARRIVAL,PRIORITY,SERVICE,START,COMPLETION,WAITING,TURNAROUND,RESPONSE
C1,0,3,8,0,8,0,8,0
C2,1,1,4,1,5,0,4,0
SUMMARY
AVERAGE_WAITING,0.00
AVERAGE_TURNAROUND,6.00
AVERAGE_RESPONSE,0.00
THROUGHPUT,0.2500
UTILIZATION,75.00
```

**Note:** The illustrative example above is not intended to be the expected result for `01_sample.txt`. Use the output generated by the program for actual results.

---

## 9. Testing and Results

The supplied test files exercise common scheduling conditions and edge cases. The provided `results.txt` records saved summary metrics for each algorithm on each input. The metrics below are reproduced from those results for `01_sample.txt`. The current `run_all.sh` also checks complete per-customer outputs against `expected/` and verifies that files in `invalid/` are rejected.

### 9.1 Comparison on `01_sample.txt`

| Metric                  | FCFS   | Priority | Preemptive Priority | Round Robin |
|-------------------------|--------|----------|---------------------|--------------|
| Average waiting time    | 2.80   |   2.40   |       2.60          | 4.20 |
| Average turnaround time | 8.00   |   7.60   |       7.80          | 9.40 |
| Average response time   | 2.80   |   2.40   |       1.60          | 0.60 |
| Last completion time    | 14     |   14     |        15           | 14 |
| Throughput              | 0.3571 |  0.3571  |       0.3333        | 0.3571 |
| Barber utilization      | 92.86% | 92.86%   |       86.67%        | 92.86% |

   ![Comparison of the four algorithms](comparison.png)

**Observations:**

- For this particular input, non-preemptive Priority has the lowest average waiting and turnaround times among the four algorithms.
- Round Robin has the lowest average response time in this example. Customers get an early opportunity to start service, but repeated time slices can increase the total time spent waiting and the time until completion.
- FCFS and non-preemptive Priority have the same throughput and utilization here, but their individual customer order and timing can differ.
- Preemptive Priority has a lower average response time than FCFS and non-preemptive Priority for this input, but its last completion time is later in this run. This is a workload-specific observation, not a general guarantee about preemptive scheduling.
- These differences arise from the interaction between arrival times, service durations, priorities, and the fact that two barbers can serve customers in parallel.

### 9.2 Fairness and delayed service on `09_starvation.txt`

The test file contains a low-priority regular customer `L` arriving at time 1, while several priority-1 customers arrive over time. The output gives the following results for `L`:

| Algorithm | First start | Completion | Waiting time | Response time |
|---|---:|---:|---:|---:|
| FCFS | 4 | 7 | 3 | 3 |
| Priority | 14 | 17 | 13 | 13 |
| Preemptive Priority | 14 | 17 | 13 | 13 |
| Round Robin | 2 | 9 | 5 | 1 |

**Observation:** Priority-based scheduling strongly favours the higher-priority customers in this workload. Customer `L` waits much longer under both Priority algorithms than under FCFS or Round Robin. This demonstrates a **starvation risk**: if higher-priority customers keep arriving, a low-priority customer may keep being postponed. Because this is a finite input and `L` eventually receives service, the test demonstrates the risk of starvation rather than proving that the customer would wait forever.

Round Robin gives `L` an earlier first start because customers take turns receiving time slices. It does not guarantee that every customer's turnaround time will be lower; its results depend on the quantum and arrivals.

### 9.3 What the other tests check

- `02_single.txt` checks basic calculations when there is one customer and the salon is initially idle.
- `03_all_same_arrival.txt` checks deterministic selection when multiple customers arrive together.
- `04_idle_gap.txt` checks that the clock can advance across a period with no ready customers and that idle time affects utilization.
- `05_equal_priority.txt` checks tie-breaking when priorities are equal. Equal-priority arrivals do not cause preemption in Preemptive Priority.
- `06_vip_preempts.txt` checks the response of Preemptive Priority when a strictly higher-priority customer arrives during service.
- `07_rr_arrival_at_expiry.txt` checks the Round Robin ordering when arrivals coincide with a quantum expiration.
- `08_rr_service_vs_quantum.txt` checks customers whose service time is less than, equal to, or greater than the quantum.
- `09_starvation.txt` illustrates how a low-priority customer can experience a long wait under priority-based scheduling.

The test harness includes `check_schedule.py`, which checks output structure, customer rows, timing relationships, metric formulas, service-capacity constraints, and FCFS ordering properties. The generated output is also compared exactly with the corresponding saved file in `expected/`. The invalid-input tests check that malformed files are rejected. On the supplied project version, `./run_all.sh` completes with `passed: 96  failed: 0` (36 valid runs and 60 invalid-input checks). These checks provide useful regression coverage, but they do not prove correctness for every possible input or replace reviewing the scheduling logic. The assignment also encourages testing equal service times, equal arrival/priority/service combinations, long idle periods, and other boundary cases.

---

## 10. Design Decisions and Assumptions

1. **Two identical barbers:** The model treats the barbers as equivalent parallel service slots; it does not model differences between individual barbers.
2. **Numerical priority controls priority decisions:** Category labels such as `VIP` and `REGULAR` do not override the given priority value.
3. **Deterministic tie-breaking:** Arrival times, numerical priorities, and lexicographic IDs are used to resolve ties as specified by the assignment.
4. **Round Robin quantum-expiration tie:** If both barber slots expire their quantum at the same time, customers are returned to the ready queue in lexicographic ID order. This makes behaviour deterministic where the specification does not give a specific tie-break.
5. **Integer time:** The input and simulation use integer time units. Preemptive Priority and Round Robin update remaining service in one-unit steps.
6. **No real-time delay:** Simulation time is separate from wall-clock time. A service time of 10 means ten simulation units, not a ten-second pause.
7. **Metrics use the first arrival as the start of elapsed time:** Throughput and utilization are calculated from the earliest customer's arrival to the last completion, not from time zero if the first customer arrives later.
8. **Fixed-size ID and category buffers:** The arrays allow up to 31 characters for IDs and categories, excluding the terminating null character. `read_input()` rejects empty or overlong values. Numeric parsing rejects malformed values and values outside the implementation's -1,000,000,000 to 1,000,000,000 range.

---

## 11. Known Limitations

The current source prints the standardized header in `print_results()`. The `read_input()` function validates positive `N` and quantum values, non-negative arrivals, positive priorities and service times, unique IDs, non-empty IDs/categories, field lengths, and malformed or out-of-range numeric values. The test script checks that the header is present and exercises valid and invalid inputs.

Other limitations:

- The program does not draw a Gantt chart or provide a graphical interface. This is optional for the assignment.
- It simulates customers and barbers in one program; it does not create real operating-system processes or model real concurrent execution.
- The implementation does not include priority aging, so it does not actively prevent starvation under Priority Scheduling.
- `check_schedule.py` checks several important output and metric properties, but it is not a formal proof of scheduling correctness for all possible inputs. The saved expected outputs cover the supplied test cases only.
- The parser does not explicitly reject a comma embedded in the category field, or extra process records after the first `N` records. The standardized input format requires category values without commas and exactly `N` process records, so supplied inputs should follow that format.
- The integer parser rejects numeric values outside the implementation's range of -1,000,000,000 to 1,000,000,000.

No features related to memory management or concurrency are implemented, because they are outside the scope of Assignment 1.

---

## 12. Optional Features

 The current project focuses on the required process-management functionality and the supplied test cases. We have added a comparison chart as well. 

---

