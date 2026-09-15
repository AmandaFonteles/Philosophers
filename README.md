*This project has been created as part of the 42 curriculum by afontele.*

# Philosophers

## Description

**Philosophers** is an introduction to multithreading and concurrency in C, based on the classic [Dining Philosophers problem](https://en.wikipedia.org/wiki/Dining_philosophers_problem) formulated by Edsger Dijkstra.

One or more philosophers sit around a round table with a bowl of spaghetti in the middle. There are as many forks as philosophers, one between each pair. To eat, a philosopher must hold **both** the fork on their left and the fork on their right. Each philosopher cycles through eating, sleeping and thinking. If a philosopher goes too long without starting a meal, they starve and the simulation ends.

The goal is to keep every philosopher alive as long as possible while guaranteeing:

- **no data races**: all shared state is protected by mutexes,
- **no deadlocks**: philosophers never end up waiting on each other forever,
- **accurate logging**: messages never overlap, and a death is reported within 10 ms.

Each philosopher runs in its own thread (`pthread`), and each fork is a `pthread_mutex_t`.

### Technical choices

**Deadlock prevention (asymmetric fork order).** Odd-numbered philosophers pick up their left fork first. Even-numbered philosophers pick up their right fork first. This breaks the circular wait condition, so a deadlock cannot occur.

**Staggered start.** Even-numbered philosophers wait `time_to_eat / 2` ms before their first meal. This lets the odd ones grab forks first and smooths out contention.

**Synchronized start time.** The start time is set slightly in the future, and all threads busy-wait until it is reached. Every thread therefore begins the simulation at the same moment, regardless of how long thread creation took.

**Monitor thread.** A dedicated thread checks every philosopher roughly every 0.5 ms. It stops the simulation when:
- a philosopher's time since their last meal exceeds `time_to_die`, or
- every philosopher has eaten at least `number_of_times_each_philosopher_must_eat` times, if that argument is given.

**Mutexes used.**

| Mutex | Protects |
|-------|----------|
| `forks[i]` | Each fork's state |
| `m_last_meal` (per philosopher) | Timestamp of the start of the last meal |
| `m_count_meal` (per philosopher) | Number of meals eaten |
| `m_dead` | The shared end-of-simulation flag |
| `print` | Standard output, so log lines never overlap |

**Interruptible sleep.** `ft_usleep` sleeps in small 500 µs steps and checks the end flag between steps. Threads therefore exit quickly once the simulation stops.

**Single philosopher.** With only one fork available, the lone philosopher takes it, waits `time_to_die` ms, and dies.

## Instructions

### Compilation

From the `philo/` directory:

```bash
make        # build the philo executable
make clean  # remove object files
make fclean # remove object files and the executable
make re     # rebuild from scratch
```

The project is compiled with `cc` and the flags `-Wall -Wextra -Werror`.

### Usage

```bash
./philo number_of_philosophers time_to_die time_to_eat time_to_sleep [number_of_times_each_philosopher_must_eat]
```

| Argument | Description |
|----------|-------------|
| `number_of_philosophers` | Number of philosophers (and forks) |
| `time_to_die` (ms) | Max time since the start of the last meal (or the simulation) before dying |
| `time_to_eat` (ms) | Time spent eating, holding two forks |
| `time_to_sleep` (ms) | Time spent sleeping |
| `number_of_times_each_philosopher_must_eat` | *(optional)* Simulation stops once every philosopher has eaten this many times |

All arguments must be positive integers that fit in an `int`.

### Output format

```
timestamp_in_ms X has taken a fork
timestamp_in_ms X is eating
timestamp_in_ms X is sleeping
timestamp_in_ms X is thinking
timestamp_in_ms X died
```

### Examples

```bash
./philo 1 800 200 200       # the philosopher takes one fork and dies at 800 ms
./philo 5 800 200 200       # no one should die
./philo 5 800 200 200 7     # stops once everyone has eaten 7 times
./philo 4 410 200 200       # no one should die
./philo 4 310 200 100       # a philosopher should die
```

### Checking for data races

```bash
# ThreadSanitizer (add -fsanitize=thread -g to CFLAGS and rebuild)
./philo 5 800 200 200

# Valgrind tools
valgrind --tool=helgrind ./philo 5 800 200 200
valgrind --tool=drd ./philo 5 800 200 200
```

## Resources

- [Dining philosophers problem – Wikipedia](https://en.wikipedia.org/wiki/Dining_philosophers_problem)
- [POSIX Threads Programming – LLNL tutorial](https://hpc-tutorials.llnl.gov/posix/)
- `man pthread_create`, `man pthread_join`, `man pthread_mutex_lock`, `man gettimeofday`, `man usleep`
- [Valgrind Helgrind manual](https://valgrind.org/docs/manual/hg-manual.html)
- [ThreadSanitizer documentation](https://github.com/google/sanitizers/wiki/ThreadSanitizerCppManual)

### AI usage

<!-- Edit this to match what you actually did -->
AI (Claude) was used to draft this README from the existing source code and the project subject. [Describe any other use here, e.g. explaining mutex/thread concepts, reviewing edge cases, or debugging data races, and which parts of the project it concerned.] All code was written, tested, and is fully understood by the author.