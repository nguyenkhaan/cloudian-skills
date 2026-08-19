# C — AGENTS Protocol

## C

### Minimum Required Linters/Analyzers

- **clang-tidy** — static analysis and modernization
- **cppcheck** — deep static analysis for C/C++
- **gcc/clang with `-Wall -Wextra -Werror -pedantic`** — treat all warnings as errors

Install if missing:
```bash
# Debian/Ubuntu
apt install clang-tidy cppcheck

# macOS
brew install llvm cppcheck
```

### Memory: Allocate, Check, Free

Every `malloc` is paired with a `free`. Every allocation is checked. No
exceptions. If you allocate memory and don't check the return, you deserve
the segfault you'll get — but the operator doesn't.

```c
/* GOOD: Allocate, check, use, free */
char *buf = malloc(BUF_SIZE);
if (!buf) {
    fprintf(stderr, "malloc(%zu) failed in %s\n", (size_t)BUF_SIZE, __func__);
    return -ENOMEM;
}
/* ... use buf ... */
free(buf);
buf = NULL;  /* prevent use-after-free */

/* BAD: No check, no cleanup, dangling pointer */
char *buf = malloc(BUF_SIZE);
strcpy(buf, input);  /* segfault if malloc failed */
/* buf is never freed */
```

### Set Pointers to NULL After Free

```c
/* GOOD */
free(ctx->buffer);
ctx->buffer = NULL;

/* BAD: Use-after-free waiting to happen */
free(ctx->buffer);
/* ctx->buffer still points to freed memory */
```

### Use Sized String Functions

```c
/* GOOD */
strncpy(dest, src, sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';

/* BETTER: Use snprintf for formatted strings */
snprintf(buf, sizeof(buf), "user=%s action=%s", user, action);

/* BAD: Buffer overflow */
strcpy(dest, src);
sprintf(buf, "user=%s action=%s", user, action);
```

### Function Length

If your function doesn't fit on two screenfuls (roughly 50 lines of logic),
it's doing too much. Split it. Name the parts. A function called
`validate_header()` is worth ten comments explaining what lines 147-203 do.

### Goto for Cleanup Is Not Evil

In C, `goto` for centralized cleanup is the correct pattern. It is not a
hack. It is not "bad practice". It is how the Linux kernel handles cleanup,
and it works.

```c
/* GOOD: Centralized cleanup */
int process_data(const char *path)
{
    int ret = -1;
    FILE *fp = NULL;
    char *buf = NULL;

    fp = fopen(path, "r");
    if (!fp) {
        fprintf(stderr, "%s: fopen(%s) failed: %s\n",
                __func__, path, strerror(errno));
        goto out;
    }

    buf = malloc(BUF_SIZE);
    if (!buf) {
        fprintf(stderr, "%s: malloc(%zu) failed\n", __func__, (size_t)BUF_SIZE);
        goto out_close;
    }

    /* ... work with fp and buf ... */
    ret = 0;

    free(buf);
out_close:
    fclose(fp);
out:
    return ret;
}

/* BAD: Duplicated cleanup, easy to get wrong */
int process_data(const char *path)
{
    FILE *fp = fopen(path, "r");
    if (!fp)
        return -1;

    char *buf = malloc(BUF_SIZE);
    if (!buf) {
        fclose(fp);  /* duplicated cleanup */
        return -1;
    }

    if (some_condition) {
        free(buf);   /* duplicated again */
        fclose(fp);  /* and again */
        return -1;
    }

    free(buf);
    fclose(fp);
    return 0;
}
```

### Integer Overflow

```c
/* GOOD: Check before arithmetic */
if (count > SIZE_MAX / sizeof(struct item)) {
    fprintf(stderr, "%s: allocation size overflow: count=%zu\n",
            __func__, count);
    return NULL;
}
struct item *items = malloc(count * sizeof(struct item));

/* BAD: Overflow wraps to small allocation, then buffer overrun */
struct item *items = malloc(count * sizeof(struct item));
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
