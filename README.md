# get_next_line

*[Version française](README.fr.md)*

Read a file descriptor one line at a time, in C. School 19 (42 network) project.

```c
char *get_next_line(int fd);
```

Each call returns the next line, newline included, or `NULL` at the end of the file or on a read error.
The caller frees each line.

```c
int   fd = open("notes.txt", O_RDONLY);
char *line;

while ((line = get_next_line(fd)) != NULL)
{
    printf("%s", line);
    free(line);
}
close(fd);
```

## How it works

- `read` fills a buffer of `BUFFER_SIZE` bytes, chosen at compile time (`-D BUFFER_SIZE=42`, default 200,
  capped at 1,000,000).
- What has been read but not returned yet is kept in a `static` string between calls. Reading stops as soon
  as that string holds a `\n`.
- The function then splits it: the line is returned, the rest stays for the next call. When nothing is
  left, the static string is freed, so a file read to the end leaves no allocation behind.
- One file descriptor at a time: the bonus version that tracks several descriptors is not part of this one.

## Tested

The output of a small reader program was compared byte for byte with the original file, with
`BUFFER_SIZE` set to 1, 42, 4,096 and 1,000,000, and AddressSanitizer enabled:

| Input | Result |
|---|---|
| empty file, one long line without `\n`, lines with and without a final `\n` | identical |
| 100,000 short lines, a single 27 KB line | identical |
| standard input (`fd` 0), a descriptor that failed to open | lines read / `NULL` |

No memory error or leak was reported.

## Build

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c get_next_line.c get_next_line_utils.c
```

`main.c` reads the sample files in `files/`.
