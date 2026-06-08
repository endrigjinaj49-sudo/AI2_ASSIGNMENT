# D1-V8: Planetary Rover - Memory Constraints and Data Management

## Overview

This project models a planetary rover that navigates through a known environment, collects scientific data, stores it in limited onboard memory, and returns to a base station to offload the collected information.

The project contains two parts:

- a classical PDDL model for navigation, data collection, memory usage, and offloading;
- a PDDL+ extension that adds continuous data corruption, continuous encoding, and automatic data-loss events.

The main objective is to show how limited memory and time-dependent data degradation influence planning decisions.

---

## Project Structure

```
D1-V8-memory-rover/
├── pddl/
│   ├── domain-memory-rover.pddl
│   ├── problem-1-simple.pddl
│   └── problem-2-memory-constrained.pddl
│
├── pddl-plus/
│   ├── domain-memory-rover-plus.pddl
│   ├── problem-plus-1-safe.pddl
│   ├── problem-plus-2-data-loss.pddl
│   ├── problem-plus-3-multiple-data.pddl
│   └── problem-plus-4-multiple-failures.pddl
│
├── plans/
├── raw/
│   ├── raw_pddl+_data_loss_problem.txt
│   ├── raw_pddl+_multiple_failures.txt
│   ├── raw_pddl+_plan_3_multiple_data.txt
│   └── raw_pddl+_safe_problem
│
├── pddl+_data_loss_problem.txt
├── pddl+_multiple_failures.txt
├── pddl+_plan_3_multiple_data.txt
├── pddl+_safe_problem.txt
├── pddl_problem_1.txt
└── pddl_problem_2.txt
│
├── report/
│   └── report.pdf
│
└── README.md
```

---

## Classical PDDL Model

The classical domain models three actions:

- `move`
- `collect-data`
- `offload-data`

The rover memory is represented using numeric fluents:

```lisp
(used-memory ?r - rover)
(memory-capacity ?r - rover)
(data-size ?d - data)
```

The collection action is permitted only when the new dataset fits inside the remaining capacity:

```lisp
(>=
  (memory-capacity ?r)
  (+ (used-memory ?r) (data-size ?d))
)
```

This condition means:

```text
used memory + new dataset size ≤ memory capacity
```

Collecting data increases used memory, while offloading decreases it.

### Classical Problem 1

`problem-1-simple.pddl` contains:

- one rover;
- one base station;
- one collection site;
- one dataset;
- sufficient memory.

The problem validates the basic sequence:

```text
move → collect → return → offload
```

### Classical Problem 2

`problem-2-memory-constrained.pddl` contains:

- four collection sites;
- four datasets with different sizes;
- a memory capacity of 10 units;
- a linear navigation map.

The dataset sizes are:

```text
data1 = 4
data2 = 4
data3 = 10
data4 = 6
```

These values demonstrate several cases:

```text
data1 + data2 = 8
data1 + data4 = 10
data3 = 10
```

The planner must determine which datasets can be stored together and when the rover must return to the base to offload data.

---

## PDDL+ Extension

The PDDL+ domain extends the classical model with continuous processes and automatic events.

### Continuous Memory Corruption

While a dataset is stored onboard, its corruption level increases continuously:

```lisp
(:process memory-corruption ...)
```

Each dataset has its own corruption rate:

```lisp
(corruption-rate ?d - data)
```

The continuous effect is based on elapsed time:

```lisp
(increase
  (corruption ?d)
  (* #t (corruption-rate ?d))
)
```

### Continuous Data Encoding

Stored data must be encoded before it can be offloaded:

```lisp
(:process data-encoding ...)
```

Each dataset has its own encoding rate:

```lisp
(encoding-rate ?d - data)
```

The encoding progress increases continuously:

```lisp
(increase
  (encoding-progress ?d)
  (* #t (encoding-rate ?d))
)
```

### Encoding-Complete Event

When the encoding progress reaches the required threshold, an automatic event marks the data as encoded:

```lisp
(:event encoding-complete ...)
```

Once encoded, the data may be offloaded at the base.

### Data-Loss Event

When corruption reaches the dataset-specific corruption limit, an automatic event marks the data as lost:

```lisp
(:event data-loss ...)
```

The `stored-by` predicate associates each stored dataset with the rover carrying it. This allows the correct rover memory to be freed when data is offloaded or lost.

---

## PDDL+ Problem Instances

### Problem Plus 1 – Safe Basic Case

This problem contains one dataset that can finish encoding before its corruption limit is reached.

Expected result:

```text
Problem Solved
```

The generated plan includes an explicit waiting period while encoding and corruption evolve continuously.

### Problem Plus 2 – Simple Data-Loss Case

This problem contains one dataset whose encoding time is greater than its survival time.

Expected result:

```text
Problem unsolvable
```

The corruption threshold is reached before encoding finishes, so the data-loss event prevents successful offloading.

### Problem Plus 3 – Successful Multi-Data Case

This problem contains several datasets with different:

- sizes;
- corruption rates;
- corruption limits;
- encoding rates;
- encoding requirements.

The planner can store multiple datasets concurrently, allow their continuous processes to evolve at the same time, selectively offload data, and complete every goal.

Expected result:

```text
Problem Solved
```

### Problem Plus 4 – Multiple-Failure Case

This problem contains four datasets:

- two datasets are temporally feasible;
- two datasets cannot finish encoding before reaching their corruption limits.

Because the goal requires all datasets to be safely offloaded, the complete mission is infeasible.

Expected result:

```text
Problem unsolvable
```

---

## Requirements

The project was tested using:

- Windows Subsystem for Linux;
- Java;
- ENHSP.

The ENHSP executable is expected at:

```text
~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar
```

---

## Running the Classical PDDL Problems

### Problem 1 – Simple Case

```bash
java -jar ~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar \
  -o /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl/domain-memory-rover.pddl \
  -f /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl/problem-1-simple.pddl \
  -planner opt-hrmax
```

### Problem 2 – Memory-Constrained Case

```bash
java -jar ~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar \
  -o /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl/domain-memory-rover.pddl \
  -f /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl/problem-2-memory-constrained.pddl \
  -planner opt-hrmax
```

---

## Running the PDDL+ Problems

### Problem Plus 1 – Safe Case

```bash
java -jar ~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar \
  -o /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/domain-memory-rover-plus.pddl \
  -f /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/problem-plus-1-safe.pddl
```

### Problem Plus 2 – Data-Loss Case

```bash
java -jar ~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar \
  -o /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/domain-memory-rover-plus.pddl \
  -f /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/problem-plus-2-data-loss.pddl
```

### Problem Plus 3 – Successful Multi-Data Case

```bash
java -jar ~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar \
  -o /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/domain-memory-rover-plus.pddl \
  -f /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/problem-plus-3-multiple-data.pddl \
  -planner opt-hrmax
```

### Problem Plus 4 – Multiple-Failure Case

```bash
java -jar ~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar \
  -o /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/domain-memory-rover-plus.pddl \
  -f /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl-plus/problem-plus-4-multiple-failures.pddl
```

---

## Saving Solver Output

The complete output produced by ENHSP can be redirected to a text file using the `>` operator.

Example:

```bash
java -jar ~/enhsp/ENHSP-Public/enhsp-dist/enhsp.jar \
  -o /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl/domain-memory-rover.pddl \
  -f /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/pddl/problem-1-simple.pddl \
  -planner opt-hrmax \
  > /mnt/c/Users/Endri/Desktop/D1-V8-memory-rover/plans/raw/raw_pddl+_safe_problem.txt
```

The raw output contains:

- parser messages;
- grounding information;
- search statistics;
- the generated plan;
- the solved or unsolvable result.

---

## Results Summary

| Problem | Model | Result |
|---|---|---|
| Problem 1 | Classical PDDL | Solved |
| Problem 2 | Classical PDDL | Solved |
| Problem Plus 1 | PDDL+ | Solved |
| Problem Plus 2 | PDDL+ | Unsolvable |
| Problem Plus 3 | PDDL+ | Solved |
| Problem Plus 4 | PDDL+ | Unsolvable |

---

## Information-Storage Abstraction

The model abstracts real information storage into symbolic objects and numeric resources.

Each dataset is represented as a PDDL object:

```lisp
data1
data2
data3
```

Each dataset has a numeric storage requirement:

```lisp
(data-size ?d)
```

The rover memory is represented using:

```lisp
(used-memory ?r)
(memory-capacity ?r)
```

The state of each dataset is represented using predicates:

```lisp
(collected ?d)
(stored ?d)
(encoded ?d)
(offloaded ?d)
(lost ?d)
```

This abstraction focuses on high-level planning decisions, including:

- whether a dataset fits in memory;
- which datasets may be stored together;
- when data must be offloaded;
- whether encoding can complete before corruption causes data loss.

It deliberately avoids modelling low-level memory hardware and file-system implementation details.

---

## Limitations

The model does not represent:

- real file systems;
- memory addresses;
- storage fragmentation;
- file compression;
- communication bandwidth;
- data-transfer duration;
- probabilistic corruption;
- partial data quality;
- uncertain dataset sizes;
- geometric motion planning;
- terrain difficulty;
- battery constraints;
- multi-rover coordination.

Movement is instantaneous in the current PDDL+ abstraction. Therefore, corruption evolves during planner-generated waiting periods but not during realistic travel durations.

The model also assumes:

- perfect knowledge of the map and data;
- deterministic actions;
- fixed dataset sizes;
- constant dataset-specific corruption rates;
- constant dataset-specific encoding rates.

The generated plans are valid within the symbolic abstraction, but they do not directly guarantee physical executability on a real rover.

---

## Report

The complete report is available in:

```text
report/report.pdf
```
---

## Conclusion

The project demonstrates how symbolic planning can represent:

- limited onboard memory;
- data collection;
- data offloading;
- partial and full memory usage;
- continuous corruption;
- continuous encoding;
- automatic encoding completion;
- automatic data loss.

The classical PDDL problems focus on capacity-constrained planning, while the PDDL+ problems show how autonomous time-dependent processes can make a mission either feasible or impossible.
````
