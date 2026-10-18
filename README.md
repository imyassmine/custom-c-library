# custom-c-library

> A C library built from scratch: libc re-implementations, `ft_printf` and `get_next_line`, compiled into a single static archive (`libft.a`).

> Written under the 42 Norme, a strict coding standard (e.g. no `for` loops, no `switch`), so the style is deliberately constrained.

## Description

`custom-c-library` gathers three foundation projects of the 42 curriculum into one reusable library:

- **`libft`**: 43 re-implemented libc functions plus extra utilities (strings, memory, conversions, file-descriptor output, linked lists).
- **`ft_printf`**: a re-implementation of `printf` (and `dprintf`) built on variadic functions.
- **`get_next_line`**: a function that reads a file descriptor one line at a time.

A single `Makefile` compiles the three modules into one static archive that can be linked into any other C project. The archive keeps the traditional name `libft.a`, so it stays compatible with other 42 projects.

## Modules

| Module | What it provides | Documentation |
| --- | --- | --- |
| `libft` | Character checks, string and memory utilities, conversions, `*_fd` output helpers, linked-list functions | [libft/README.md](libft/README.md) |
| `ft_printf` | `ft_printf` and `ft_dprintf` with the conversions `c s p d i u x X %` | [ft_printf/README.md](ft_printf/README.md) |
| `get_next_line` | `get_next_line(int fd)`: returns the next line, including its `\n` when present | [get_next_line/README.md](get_next_line/README.md) |

`ft_printf` is self-contained (it only needs `write`). `get_next_line` reuses `libft` functions (`ft_memcpy`, `ft_strchr`, `ft_strdup`, `ft_strlen`) through `get_next_line.h`, so the modules are meant to be linked together as one archive.

### `libft` at a glance

| Category | Functions |
| --- | --- |
| Character checks | `ft_isalpha` `ft_isdigit` `ft_isalnum` `ft_isascii` `ft_isprint` |
| Strings | `ft_strlen` `ft_strlcpy` `ft_strlcat` `ft_strchr` `ft_strrchr` `ft_strncmp` `ft_strnstr` `ft_strdup` `ft_substr` `ft_strjoin` `ft_strtrim` `ft_split` `ft_strmapi` `ft_striteri` |
| Memory | `ft_memset` `ft_bzero` `ft_memcpy` `ft_memmove` `ft_memchr` `ft_memcmp` `ft_calloc` |
| Conversions | `ft_atoi` `ft_itoa` `ft_toupper` `ft_tolower` |
| Output | `ft_putchar_fd` `ft_putstr_fd` `ft_putendl_fd` `ft_putnbr_fd` |
| Linked lists | `ft_lstnew` `ft_lstadd_front` `ft_lstadd_back` `ft_lstsize` `ft_lstlast` `ft_lstdelone` `ft_lstclear` `ft_lstiter` `ft_lstmap` |

## Repository structure

```
custom-c-library/
├── Makefile
├── README.md
├── .gitignore
├── include/
│   ├── libft.h
│   ├── ft_printf.h
│   └── get_next_line.h
├── libft/
│   ├── README.md
│   └── src/
├── ft_printf/
│   ├── README.md
│   └── src/
└── get_next_line/
    ├── README.md
    └── src/
```

## Instructions

### Requirements

- A C compiler (`cc`)
- `make`
- A standard Unix environment (`read` and `write` system calls)

### Build

```bash
git clone https://github.com/Yassmine-ab/custom-c-library.git
cd custom-c-library
make
```

This compiles every source file (with `-Wall -Wextra -Werror`) into `obj/` and creates `libft.a` at the repository root. Only the files that changed are recompiled on later builds.

### Makefile targets

| Target | Description |
| --- | --- |
| `make` / `make all` | Build `libft.a` |
| `make clean` | Remove the object files (`obj/`) |
| `make fclean` | Remove the object files and `libft.a` |
| `make re` | Full rebuild (`fclean` then `all`) |
| `make help` | Show the available targets |

### Usage

Include the headers you need and link against the archive:

```c
#include <fcntl.h>
#include "libft.h"
#include "ft_printf.h"
#include "get_next_line.h"

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

```bash
cc -Wall -Wextra -Werror -I include main.c -L. -lft -o demo
./demo some_file.txt
```

The default `BUFFER_SIZE` of `get_next_line` (4096) is defined in `include/get_next_line.h`. See the [get_next_line README](get_next_line/README.md) to change it.

## What this project demonstrates

- **Manual memory management**: allocations are checked and released on every failure path (`ft_split`, `ft_lstmap`, `get_next_line`).
- **Defensive edge-case handling**: overflow-safe `ft_calloc`, `INT_MIN` support in `ft_putnbr_fd` and `ft_itoa`, `NULL` handling in `ft_printf`.
- **Variadic functions**: `va_list`-based argument dispatch in `ft_printf` / `ft_dprintf`.
- **Low-level I/O**: direct use of the `read` and `write` system calls, with state kept between calls through a static buffer.
- **Build tooling**: a single Makefile that compiles several source trees into one static library, with out-of-tree objects and incremental rebuilds.

## Resources

- Manual pages (`man`) for libc functions, `printf`, `stdarg` and `read`.
- Static storage duration in C: https://en.cppreference.com/w/c/language/storage_duration
- Linked-list fundamentals: https://www.geeksforgeeks.org/data-structures/linked-list/

### AI Usage Disclosure

AI assistance was used for code review and debugging guidance, for restructuring the Makefile, and for drafting the documentation. It was not used to replace manual implementation work on the C functions.
