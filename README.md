*This project has been created as part of the 42 curriculum by dide-jes.*

# get_next_line

## Description

`get_next_line` is a C function that reads from a file descriptor and returns the next line on each call (including the terminating `\n` when present). The goal is to provide a reliable, memory-safe way to iterate through a stream (file, stdin, etc.) line-by-line without reading the whole file at once.

This repository contains:
- `get_next_line.c`: the `get_next_line` implementation
- `get_next_line_utils.c`: helper functions (`ft_strlen`, `ft_substr`, `ft_strjoin`, `ft_strchr`)
- `get_next_line.h`: public prototypes and `BUFFER_SIZE` default

## Instructions

### Compile

You can compile and run it directly:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c -o gnl
```

### Run

- Read from a file:

```bash
./gnl test_input.txt
```

- Read from standard input (interactive):

```bash
./gnl
```

## Algorithm (detailed)

The selected approach is the classic **“stash + buffered reads”** algorithm:

1. **Persistent remainder (stash)**
   - A `static char *` (named `left_c` in this code) stores leftover characters that were read previously but not returned yet.
   - Persistence across function calls is necessary because a single `read()` call may return more than one line, or a line may span multiple reads.

2. **Read chunks of size `BUFFER_SIZE`**
   - Each `get_next_line(fd)` call allocates a temporary `buffer` of size `BUFFER_SIZE + 1`.
   - The code repeatedly calls `read(fd, buffer, BUFFER_SIZE)` and null-terminates `buffer`.

3. **Accumulate into a line buffer**
   - After each read, the freshly read chunk is appended to the stash using `ft_strjoin`.
   - This builds a “line buffer” (`left_c`) containing: previous remainder + newly read bytes.

4. **Stop condition**
   - Reading stops when either:
     - a newline is observed in the last chunk read (`'\n'` in `buffer`), or
     - `read()` returns `0` (EOF).

5. **Split into “returned line” and “new remainder”**
   - Once enough content exists, the function splits at the first newline:
     - The returned `line` is truncated right after `\n` (or ends at `\0` at EOF).
     - The part after that is copied into a new stash (`left_c`) for the next call.

### Why this algorithm

- **Correctness for arbitrary line lengths**: lines longer than `BUFFER_SIZE` are handled by repeatedly reading and appending until the newline arrives.
- **Streaming-friendly**: memory usage grows only with the longest in-progress line and remainder, not the whole file.
- **Good performance characteristics**:
  - Fewer syscalls than reading byte-by-byte (reads happen in `BUFFER_SIZE` chunks).
  - Time is linear in the amount of data processed per returned line, i.e., $O(n)$ for each produced line.

### Implementation constraints / behavior notes

- This implementation uses a **single static stash**, so it is intended for **one file descriptor at a time**. (Supporting multiple FDs typically requires an array/map of stashes indexed by `fd`.)
- Returned lines are dynamically allocated; the caller must `free()` each returned line.

## Resources

### Classic references

- `man 2 read`, `man 2 open`, 
- `man 3 malloc`, `man 3 free`
- https://www.geeksforgeeks.org/
- https://www.youtube.com/watch?v=-Mt2FdJjVno

