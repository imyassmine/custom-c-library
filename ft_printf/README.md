# ft_printf

> Written under the 42 Norme, a strict coding standard (e.g. no `for` loops, no `switch`), so the style is deliberately constrained.

## Description
`ft_printf` is a custom implementation of the standard C `printf()` function, developed as part of the 42 curriculum. The goal is to understand variadic functions (functions with a variable number of arguments) and build a modular, easy-to-extend formatter.

This project is designed to:
- Learn how variadic functions work (`va_list`, `va_start`, `va_arg`, `va_end`).
- Implement a modular, easy-to-extend formatting system.
- Practice low-level string manipulation and character output.
- Produce a static library that can be reused in future C projects.

This module is part of [custom-c-library](../README.md): the `Makefile` at the repository root builds it into `libft.a`, together with `libft` and `get_next_line`. `ft_printf` itself does not depend on `libft`: it only uses `write`.

## Public API

```c
int	ft_printf(const char *str, ...);
int	ft_dprintf(int fd, const char *str, ...);
```

- `ft_printf` writes to the standard output.
- `ft_dprintf` does the same on any file descriptor `fd`.
- Both return the number of characters written, like the real `printf`. If `str` is `NULL`, they return `0`.

Both are declared in [`include/ft_printf.h`](../include/ft_printf.h).

## Supported Conversions

| Conversion | Description |
| --- | --- |
| `%c` | Prints a single character. |
| `%s` | Prints a string (or `(null)` if the pointer is `NULL`). |
| `%p` | Prints a `void *` pointer in lowercase hexadecimal, prefixed with `0x` (or `(nil)` if the pointer is `NULL`). |
| `%d` | Prints a signed decimal integer. |
| `%i` | Prints a signed decimal integer (same as `%d`). |
| `%u` | Prints an unsigned decimal integer. |
| `%x` | Prints an unsigned integer in hexadecimal format (lowercase). |
| `%X` | Prints an unsigned integer in hexadecimal format (uppercase). |
| `%%` | Prints a percent sign (`%`). |

### Scope

Only the conversions listed above are implemented. Any other character after `%` produces no output, and a lone `%` at the end of the format string is ignored.

## Algorithm & Data Structure

The implementation uses a **single-pass, character-by-character parsing approach**:

1. **Entry points** (`ft_printf`, `ft_dprintf`):
   - Initialize the `va_list` and call `print_format()` with the target file descriptor (`1` for `ft_printf`).

2. **Format parser** (`print_format`):
   - Iterates through the format string.
   - Ordinary characters are written directly.
   - When a `%` followed by another character is found, the next character is handed to the dispatcher.

3. **Dispatcher** (`dispatch`):
   - Examines the conversion specifier (`c`, `s`, `d`, ...).
   - Extracts the matching argument with `va_arg()`.
   - Routes it to the appropriate output helper (`%p` goes through the static `handle_ptr()`).

4. **Output helpers** (`ft_printf_utils.c`):
   - `putchar_count()`: writes one character and adds the result of `write` to the counter.
   - `putstr_count()`: writes a string character by character (prints `(null)` for `NULL`).
   - `putnbr_count()`: recursively writes a signed decimal number (uses a `long` to handle `INT_MIN`).
   - `putunbr_count()`: recursively writes an unsigned decimal number.
   - `puthex_count()`: recursively writes a hexadecimal number, lowercase or uppercase.

5. **Character counting**:
   - Each helper receives a `*count` pointer to the running total.
   - `print_format()` returns this total, matching the behavior of the real `printf`.

### Key Design Decisions

- **No buffer management**: output goes straight to `write()`, one call per character. This is simple and easy to follow, but not optimized for speed.
- **Recursion for number output**: digit extraction needs no loops, which fits the Norme restrictions.
- **Safe string and pointer handling**: `NULL` strings print `(null)` and `NULL` pointers print `(nil)` (defined by `NULL_PTR` in the header).
- **Explicit argument types**: `%c` reads an `int` and casts it to `char`, `%d`/`%i` read an `int`, `%u`/`%x`/`%X` read an `unsigned int`, and `%p` converts the pointer to `unsigned long` to keep the full address on 64-bit systems.
- **One parser for both entry points**: `ft_printf` and `ft_dprintf` share `print_format()`.

The helper functions (`putchar_count`, `putstr_count`, ...) are declared in the header because they are shared between the two source files. They are not part of the public API.

## Instructions

### Requirements
- A C compiler (`cc`)
- `make`
- Standard Unix environment with `write()` syscall

### Build
From the repository root:

```bash
make
```

This compiles the library and creates `libft.a` at the root of the repository (it also contains `libft` and `get_next_line`).

### Usage

```c
#include "ft_printf.h"

int	main(void)
{
	int	count;

	count = ft_printf("%s has %d items (%x) at %p\n", "list", 42, 42, NULL);
	ft_dprintf(2, "printed %d characters\n", count);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -I include main.c -L. -lft -o demo
```

### Clean build artifacts

```bash
make clean
```

Removes object files.

```bash
make fclean
```

Removes object files and `libft.a`.

```bash
make re
```

Performs a full rebuild (`fclean` then `all`).

## Resources
References used for this project:
- Manual pages (`man printf`, `man stdarg`) for variadic function macros and printf behavior.
- C standard library documentation on type promotion and variadic argument passing.
- Hexadecimal number conversion algorithms:
  - https://www.geeksforgeeks.org/convert-decimal-to-hexadecimal/
