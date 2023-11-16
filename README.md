# Hash Hash Hash
This is a basic thread safety and speed-up testing program. It is a hash table
that uses a linked list to handle collisions. When run with multiple threads,
it will use a mutex to ensure safety between threads. There are two versions
of the implementation. The first uses only one mutex. The second uses a mutex
for every hash table entry to help with speed-ups.

## Building
```shell
make
```

## Running
```shell
./hash_table_tester -t <num_threads> -s <num_elem_per_thread>
```

<num_threads> is the number of threads to run. 
<num_elem_per_thread> is the number of elements each thread will add to the hash table.

This will run the base test (single thread) and both versions of the multi-threaded test.
It will output the time for each test and the number of missing elements from the hash table.

Example Output:
```shell
$ ./hash-table-tester -t 4 -s 50000

Generation: 25545 usec
Hash table base: 113714 usec
  - 0 missing
Hash table v1: 486003 usec
  - 0 missing
Hash table v2: 37243 usec
  - 0 missing
```

## First Implementation
In the `hash_table_v1_add_entry` function, I added (2) pthread lock-unlock pairs to lock
and unlock the mutex that is initialized at the beginning of the program. The first pair
surrounds the function that searches for the entry in the hash table. The second lock
handles the linked list macro that adds to the linked list.

### Performance
```shell
./hash_table_tester -t <num_threads> -s <num_elem_per_thread>
```
Version 1 is slower than the base version despite being multi-threaded. Considering the limitation
of only one mutex, this is expected. The mutex is used by all threads, so the threads are all
waiting for the resource to be freed. The individual threads run almost in serial, but there is the
added overhead of the mutex, making it slower overall.

## Second Implementation
In the second version, the `hash_table_v2_add_entry` function, I added a mutex to the 
hash table entry structure. This meant that I had 4096 mutexes instead of 1. This allowed
me to lock each hash table entry separtely so that I wouldn't have to wait for other
threads to finish when they're not accessing the same resources.

### Performance
```shell
./hash_table_tester -t <num_threads> -s <num_elem_per_thread>
```

Version 2 is much faster than the base implementation (depending on the number of threads used).
Because the threads are now mostly not waiting for each other, the threads can run in parallel
and the work can be done much faster.

I executed the following implementation on my WSL (Windows Subsystem for Linux) machine (6 cores):

```shell
./hash_table_tester -t 4 -s 50000
```

My average output for 5 runs was:
- Base:         117431.6 usec
- Version 1:    374122.4 usec
- Version 2:     39871.8 usec

My average speedup was 2.95x.

When I upped the number of threads to 6, I got the following results:

```shell
./hash_table_tester -t 6 -s 50000
```

Average for 5 runs:
- Base:          318352.8 usec
- Version 1:    1129983.4 usec
- Version 2:      93029.8 usec

Average speedup: 3.42x

Even though I had 6 cores, the speedup was not proportional to the thread count. I'm
anticipating that this is because it's a WSL machine and not a true Linux machine, 
and there are other background tasks running on my Windows base OS.
Also, however, there would likely be more mutex contention with more threads, so
the speedup would not be as great.

## Cleaning up
```shell
make clean
```