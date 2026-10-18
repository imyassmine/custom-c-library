# get_next_line

> Written under the 42 Norme, a strict coding standard (e.g. no `for` loops, no `switch`), so the style is deliberately constrained.

## Description
`get_next_line` is a 42 C project where the goal is to implement a function that reads from a file descriptor and returns one line at a time.

The function must:
- Return the next line, including the trailing `\n` when present.
- Keep internal state between calls so reading continues exactly where the previous call stopped.
- Return `NULL` on EOF (when nothing remains) or on error.

This project focuses on low-level I/O (`read`), dynamic memory management, and robust edge-case handling.

This module is part of [custom-c-library](../README.md): the `Makefile` at the repository root builds it into `libft.a`, together with `libft` and `ft_printf`. It reuses `libft` functions (`ft_memcpy`, `ft_strchr`, `ft_strdup`, `ft_strlen`), which `get_next_line.h` includes.

## Function Prototype

```c
char *get_next_line(int fd);
```

Declared in [`include/get_next_line.h`](../include/get_next_line.h).

**Parameters:**
- `fd`: The file descriptor to read from.

**Returns:**
- A dynamically allocated string containing the next line (with `\n` if present). The caller must `free` it.
- `NULL` if there is nothing else to read, if `fd` is negative, if `BUFFER_SIZE` is lower than 1, or if an error occurred.

## Algorithm & Data Structure

The implementation uses a **static remainder buffer**:

1. **Initialization**: A static `char *rem` persists across calls and holds the data that was read but not returned yet. On the first call it is set to an empty string. A heap buffer of `BUFFER_SIZE + 1` bytes is allocated for each call and freed before returning.

2. **Reading loop** (`fill_remainder`):
   - While the remainder does not contain a `\n` and `read()` returns bytes:
     - Read up to `BUFFER_SIZE` bytes into the buffer.
     - Append them to the remainder with `ft_strjoin_with_len()`, a local helper that takes both lengths explicitly and copies with `ft_memcpy`.
     - Free the old remainder and replace it with the concatenated result.
   - A `read()` error or an allocation failure stops the loop and the function returns `NULL` after freeing everything.

3. **End of file**: if the remainder is empty after the loop, there is nothing left to return. It is freed and `NULL` is returned.

4. **Line extraction** (`extract_line`):
   - Measure the line length, up to and including the first `\n`.
   - Allocate the line and copy those characters.
   - Duplicate the rest of the data (`ft_strdup`) as the new remainder, and free the old one. If nothing is left, the remainder is reset to `NULL`.

### Key Design Decisions

- **Static variable**: keeps the state between calls without using a global variable.
- **Heap-allocated read buffer**: `BUFFER_SIZE` can be very large without risking a stack overflow.
- **Explicit lengths when appending**: `ft_strjoin_with_len()` does not rescan the strings, and keeps the length of the remainder up to date during the loop.
- **Reuse of `libft`**: no duplicated string helpers, so the library has a single definition of each function.
- **Error handling**: on a `read()` or `malloc()` failure the function frees the buffer and the remainder, and returns `NULL`, so no memory is leaked.

### Performance note

With a very small `BUFFER_SIZE` and a very long line, `ft_strchr` rescans the remainder after each `read()`, so the cost grows quickly.

## Instructions

### Requirements
- A C compiler (`cc`)
- `make`
- Standard Unix environment with `read()` syscall

### Build
From the repository root:

```bash
make
```

This creates `libft.a`, which contains `get_next_line` and the `libft` functions it relies on.

### Compile a test program

```bash
cc -Wall -Wextra -Werror -I include main.c -L. -lft -o gnl
```

Example `main.c`:

```c
#include <fcntl.h>
#include "get_next_line.h"
#include "ft_printf.h"

int	main(int argc, char **argv)
{
	int		fd;
	char	*line;

	if (argc != 2)
		return (1);
	fd = open(argv[1], O_RDONLY);
	if (fd < 0)
		return (1);
	line = get_next_line(fd);
	while (line)
	{
		ft_printf("%s", line);
		free(line);
		line = get_next_line(fd);
	}
	close(fd);
	return (0);
}
```

### Custom `BUFFER_SIZE`

The default value (`4096`) is defined in `include/get_next_line.h`. Because it is used while building the library, rebuild with another value like this:

```bash
make re CFLAGS="-Wall -Wextra -Werror -I include -D BUFFER_SIZE=42"
```

### Run

```bash
./gnl some_file.txt
```

**Note:** The commands above assume you use your own local test `main.c`.

## Resources
References:
- Manual page (`man read`) for function behavior.
- Static storage duration in C: https://en.cppreference.com/w/c/language/storage_duration
