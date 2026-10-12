# ft_popen — Process Communication Lab

**42 Exam Rank 04 | C Programming | Unix Processes**

## Learning Objectives

In this lab, you will implement a simplified version of the standard C library's `popen()` function.

By completing this exercise, you will learn how to:

- Create a unidirectional communication channel using `pipe()`.
- Create a child process using `fork()`.
- Redirect standard input and output using `dup2()`.
- Execute external commands using `execvp()`.
- Manage file descriptors correctly to prevent resource leaks.
- Handle errors in parent and child processes.

### Assignment

```text
Assignment name  : ft_popen
Expected files   : ft_popen.c
Allowed functions: pipe, fork, dup2, execvp, close, exit
```

Implement the following function:

```c
int ft_popen(const char *file, char *const argv[], char type);
```

The function must launch the executable `file` with arguments `argv` using `execvp()`.

- If `type == 'r'`, return a file descriptor connected to the command's standard output.
- If `type == 'w'`, return a file descriptor connected to the command's standard input.
- Return `-1` on invalid parameters or detectable errors.

## 1. Understand the Problem

Before writing code, understand how the parent process communicates with the child process.

### Read Mode (`'r'`)

When the function is called with `'r'`, the parent must be able to read the output produced by the child.

```text
       CHILD PROCESS
       ┌───────────────┐
       │ execvp("ls")  │
       │               │
       │ STDOUT (1)    │
       └──────┬────────┘
              │
              ▼
         PIPE WRITE
           fd[1]
              │
              ▼
         PIPE READ
           fd[0]
              │
              ▼
       ┌───────────────┐
       │ PARENT        │
       │ read(fd, ...) │
       └───────────────┘
```

### Write Mode (`'w'`)

When the function is called with `'w'`, the parent must be able to send input to the child.

```text
       PARENT PROCESS
       ┌────────────────┐
       │ write(fd, ...) │
       └───────┬────────┘
               │
               ▼
          PIPE WRITE
            fd[1]
               │
               ▼
          PIPE READ
            fd[0]
               │
               ▼
       ┌────────────────┐
       │ CHILD PROCESS  │
       │ STDIN (0)      │
       │ execvp("cat")  │
       └────────────────┘
```

**Important:** A pipe is unidirectional. Data always flows from `fd[1]` to `fd[0]`, regardless of whether the function operates in read or write mode.

## 2. Validate Function Parameters

Start by implementing the function declaration and validating its arguments.

### Task

1. Check whether `file` is `NULL` or an empty string.
2. Check whether `argv` or `argv[0]` is `NULL`.
3. Accept only `'r'` and `'w'` as valid modes.
4. Return `-1` for invalid parameters.

For now, return `0` when the parameters are valid. This is only a temporary value used during development.

### Solution

```c
// ft_popen.c

int ft_popen(const char *file, char *const argv[], char type)
{
    if (!file || !file[0] || !argv || !argv[0])
        return (-1);
    if (type != 'r' && type != 'w')
        return (-1);
    return (0);
}
```

### Explanation

The expression:

```c
!file || !file[0]
```

checks two conditions:

- `!file`: The pointer is `NULL`.
- `!file[0]`: The string is empty (`""`).

The order matters because C evaluates `||` from left to right and stops as soon as the result is known. If `file` is `NULL`, `file[0]` is never accessed.

The same principle applies to:

```c
!argv || !argv[0]
```

## 3. Test the Function Prototype

Before implementing process communication, verify that your function can be called correctly.

### Task

Create a separate `main.c` file and test both valid and invalid parameters.

### Solution

```c
// main.c

#include <stdio.h>

int ft_popen(const char *file, char *const argv[], char type);

int main(void)
{
    char *argv[] = {"ls", NULL};

    printf("Invalid: %d\n", ft_popen(NULL, NULL, 'r'));
    printf("Valid: %d\n", ft_popen("ls", argv, 'r'));
    return (0);
}
```

Compile and execute:

```sh
cc -Wall -Wextra -Werror main.c ft_popen.c
./a.out
```

Expected output:

```text
Invalid: -1
Valid: 0
```

### Passing Arrays as Function Arguments

You cannot directly write:

```c
ft_popen("ls", {"ls", NULL}, 'r');
```

The initializer `{...}` cannot be used as a standalone function argument in C.

There are two valid approaches.

**Method 1: Declare an array**

```c
char *argv[] = {"ls", NULL};

ft_popen("ls", argv, 'r');
```

**Method 2: Use a compound literal (C99)**

```c
ft_popen("ls", (char *const []){"ls", NULL}, 'r');
```

The expression `(char *const []){"ls", NULL}` creates an unnamed array that can be passed directly to the function.

Both approaches are valid.

## 4. Create a Pipe

Now we need a communication channel between two processes.

### Understanding `pipe()`

```c
#include <unistd.h>

int pipe(int pipefd[2]);
```

The function creates two file descriptors:

| Descriptor | Purpose |
|---|---|
| `fd[0]` | Read data from the pipe |
| `fd[1]` | Write data into the pipe |

Return values:

- `0`: Success
- `-1`: Failure

### Task

1. Declare an integer array of size two.
2. Call `pipe()` to initialize the descriptors.
3. Return `-1` if pipe creation fails.

### Code to Add

After parameter validation:

```c
int fd[2];

if (pipe(fd) == -1)
    return (-1);
```

At this point, the process owns two open file descriptors.

**Important:** Every successfully opened descriptor must eventually be closed when no longer needed.

## 5. Create a Child Process

The next step is creating a second process using `fork()`.

### Understanding `fork()`

```c
#include <unistd.h>

pid_t fork(void);
```

`fork()` duplicates the calling process.

The return value determines which process is executing:

| Return value | Meaning |
|---|---|
| `-1` | Process creation failed |
| `0` | Current process is the child |
| Positive PID | Current process is the parent |

After a successful `fork()`, both processes continue executing from the instruction immediately after the call.

### Task

1. Declare a `pid_t` variable.
2. Call `fork()` after creating the pipe.
3. Handle process creation failure.
4. Separate parent and child execution.

### Solution

```c
pid_t pid;

pid = fork();
if (pid == -1)
{
    close(fd[0]);
    close(fd[1]);
    return (-1);
}

if (pid == 0)
{
    // Child process
    exit(0);
}
else
{
    // Parent process
}
```

### Why Close Both Descriptors on Fork Failure?

The pipe was created before calling `fork()`.

If `fork()` fails, the parent still owns both pipe descriptors.

Therefore, both must be closed before returning.

### Important Concept: File Descriptors After Fork

Immediately after `fork()`, the parent and child each have descriptors for both pipe ends.

```text
PARENT                  CHILD

fd[0] ────┐       ┌──── fd[0]
          └─ PIPE ┘
fd[1] ────┐       ┌──── fd[1]
          └──────┘
```

There are four descriptor references in total, but they refer to the same underlying pipe.

Closing a descriptor in one process does not automatically close the corresponding descriptor in the other process.

## 6. Close Unnecessary Pipe Ends

Each process should retain only the descriptor required for its operation.

### Read Mode

The child writes to the pipe, and the parent reads from it.

```c
if (pid == 0)
{
    close(fd[0]); // Child doesn't read
    // Child operation
}
else
{
    close(fd[1]); // Parent doesn't write
}
```

### Write Mode

The parent writes to the pipe, and the child reads from it.

```c
if (pid == 0)
{
    close(fd[1]); // Child doesn't write
    // Child operation
}
else
{
    close(fd[0]); // Parent doesn't read
}
```

### Task

Implement the descriptor closing logic based on `type`.

### Solution

```c
if (pid == 0)
{
    if (type == 'r')
        close(fd[0]);
    else
        close(fd[1]);

    // Child operation

    if (type == 'r')
        close(fd[1]);
    else
        close(fd[0]);
    exit(0);
}
else
{
    if (type == 'r')
        close(fd[1]);
    else
        close(fd[0]);
}
```

The child closes its remaining descriptor after the placeholder operation.

Later, we will replace that placeholder with standard input/output redirection and command execution.

### Returning the Correct Descriptor

The parent must return the descriptor it retained.

```c
if (type == 'r')
    return (fd[0]);
return (fd[1]);
```

## 7. Redirect Standard Input and Output

We now have two processes connected through a pipe.

However, external commands normally communicate using standard input and standard output.

We need to connect those standard descriptors to our pipe.

### Understanding `dup2()`

```c
#include <unistd.h>

int dup2(int oldfd, int newfd);
```

`dup2()` makes `newfd` refer to the same open file or pipe endpoint as `oldfd`.

Common standard descriptors:

| Descriptor | Constant | Purpose |
|---|---|---|
| `0` | `STDIN_FILENO` | Standard input |
| `1` | `STDOUT_FILENO` | Standard output |
| `2` | `STDERR_FILENO` | Standard error |

For example:

```c
dup2(fd[1], STDOUT_FILENO);
```

This redirects standard output to the pipe's write end.

After a successful `dup2()`, the original descriptor can be closed because the duplicated descriptor remains valid.

### Task

In the child process:

1. For `'r'`, redirect standard output to the pipe's write end.
2. For `'w'`, redirect standard input to the pipe's read end.
3. Handle `dup2()` errors.
4. Close the original pipe descriptor after redirection.

### Solution

```c
if (pid == 0)
{
    if (type == 'r')
    {
        close(fd[0]);
        if (dup2(fd[1], STDOUT_FILENO) == -1)
        {
            close(fd[1]);
            exit(1);
        }
        close(fd[1]);
    }
    else
    {
        close(fd[1]);
        if (dup2(fd[0], STDIN_FILENO) == -1)
        {
            close(fd[0]);
            exit(1);
        }
        close(fd[0]);
    }

    // Execute command here
    exit(0);
}
```

### Why Use `exit()` Instead of `return`?

After `fork()`, the child process is executing independently.

If the child simply returns from `ft_popen()`, it may continue executing the caller's code.

Instead, we want the child to terminate completely when an error occurs.

```c
exit(1);
```

A zero exit status conventionally means success, while a nonzero status indicates failure.

## 8. Execute the Command

The final implementation step is replacing the child process's program with the requested executable.

### Understanding `execvp()`

```c
#include <unistd.h>

int execvp(const char *file, char *const argv[]);
```

Arguments:

- `file`: Executable name or path.
- `argv`: NULL-terminated array of arguments.

For example:

```c
char *argv[] = {"ls", "-l", NULL};

execvp("ls", argv);
```

Unlike `execv()`, `execvp()` searches directories listed in the `PATH` environment variable when the executable name does not contain a slash.

### Important Behavior

If `execvp()` succeeds, it replaces the current process's program image.

The process retains its PID, but the old program code does not continue executing.

If `execvp()` fails, it returns `-1`.

### Task

1. Call `execvp()` inside the child.
2. Pass the original `file` and `argv`.
3. Terminate the child if execution fails.

### Solution

```c
execvp(file, argv);
exit(1);
```

There is no need to check the return value explicitly because `execvp()` returns only on failure.

## 9. Complete Implementation

Here is the complete `ft_popen.c` implementation.

```c
// ft_popen.c

#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>

int ft_popen(const char *file, char *const argv[], char type)
{
    int     fd[2];
    pid_t   pid;

    if (!file || !file[0] || !argv || !argv[0])
        return (-1);
    if (type != 'r' && type != 'w')
        return (-1);

    if (pipe(fd) == -1)
        return (-1);

    pid = fork();
    if (pid == -1)
    {
        close(fd[0]);
        close(fd[1]);
        return (-1);
    }

    if (pid == 0)
    {
        if (type == 'r')
        {
            close(fd[0]);
            if (dup2(fd[1], STDOUT_FILENO) == -1)
            {
                close(fd[1]);
                exit(1);
            }
            close(fd[1]);
        }
        else
        {
            close(fd[1]);
            if (dup2(fd[0], STDIN_FILENO) == -1)
            {
                close(fd[0]);
                exit(1);
            }
            close(fd[0]);
        }

        execvp(file, argv);
        exit(1);
    }

    if (type == 'r')
    {
        close(fd[1]);
        return (fd[0]);
    }

    close(fd[0]);
    return (fd[1]);
}
```

**Note:** This is the basic exam-oriented implementation. It assumes the standard descriptors are open when called, as they normally are. Special descriptor-number collisions and reporting child-side execution failures synchronously to the parent require additional consideration.

## 10. Test with Real Commands

Now we can test the function using actual executables.

### Test A: Read Mode

Execute `ls` and read its output from the returned file descriptor.

```c
// main.c

#include <stdio.h>
#include <unistd.h>

int ft_popen(const char *file, char *const argv[], char type);

int main(void)
{
    char    *argv[] = {"ls", NULL};
    char    buffer[128];
    ssize_t bytes;
    int     fd;

    fd = ft_popen("ls", argv, 'r');
    if (fd == -1)
        return (1);

    while ((bytes = read(fd, buffer, sizeof(buffer))) > 0)
        write(STDOUT_FILENO, buffer, bytes);

    close(fd);
    if (bytes == -1)
        return (1);
    return (0);
}
```

Compile:

```sh
cc -Wall -Wextra -Werror main.c ft_popen.c -o test
./test
```

Expected behavior: The program prints the directory contents, just like the `ls` command.

### Why Use `ssize_t` Instead of `size_t`?

The return value of `read()` is a signed integer:

- Positive: Number of bytes read.
- Zero: End of file.
- Negative (`-1`): Error.

Using `size_t` would incorrectly convert `-1` into a large unsigned integer.

This could cause out-of-bounds access if that value is used as an array index.

Also notice that `write()` does not require a null-terminated string. It receives the exact number of bytes to write.

### Test B: Write Mode

Execute `cat` and send data into its standard input.

```c
// main.c

#include <unistd.h>

int ft_popen(const char *file, char *const argv[], char type);

int main(void)
{
    char *argv[] = {"cat", NULL};
    int  fd;

    fd = ft_popen("cat", argv, 'w');
    if (fd == -1)
        return (1);

    write(fd, "Hello from parent!\n", 19);
    close(fd);
    return (0);
}
```

Expected output:

```text
Hello from parent!
```

The `cat` process reads from its standard input, which is connected to the pipe, and writes the received text to its standard output.

### Why Is `close(fd)` Important?

In write mode, the child may wait for more input.

Closing the parent's write descriptor allows the child to receive EOF once no other write descriptors remain open.

Without EOF, a program such as `cat` may continue waiting for input.

## 11. Test Error Cases

Test the following scenarios:

| Test | Expected result |
|---|---|
| `file == NULL` | `-1` |
| `file == ""` | `-1` |
| `argv == NULL` | `-1` |
| `argv[0] == NULL` | `-1` |
| `type == 'x'` | `-1` |
| `pipe()` failure | `-1` |
| `fork()` failure | `-1` |
| Nonexistent executable | Child execution fails; parent may still return an FD |

### Important: `execvp()` Failure

Consider:

```c
char *argv[] = {"nonexistent_command_123", NULL};

int fd = ft_popen(argv[0], argv, 'r');
```

The parent may return a valid descriptor even though the child fails to execute the command.

Why?

After `fork()`, parent and child execute independently.

The parent may return before the child calls `execvp()`.

Even if the child fails immediately, the parent cannot automatically know about that failure.

`WIFEXITED()` and `WEXITSTATUS()` can inspect a child's termination status obtained through functions such as `waitpid()`, but those waiting functions are not permitted by this assignment.

Also, waiting before returning the descriptor could deadlock when the child needs the parent to read from or write to the pipe.

Therefore, the basic implementation reports errors that the parent can detect directly, while child-side execution failures remain a limitation.

The assignment's general error requirement is broader than what this simple implementation can synchronously guarantee.

## 12. Check for File Descriptor Leaks

Every successful `pipe()` call creates two file descriptors.

Every successful `fork()` duplicates references to those descriptors into the child.

Incorrect descriptor management can cause resource leaks or prevent EOF from being delivered.

### Testing with Valgrind

Compile your test program with debugging information:

```sh
cc -Wall -Wextra -Werror -g main.c ft_popen.c -o test
```

Run:

```sh
valgrind --track-fds=yes ./test
```

Inspect the file descriptor information.

Remember that the standard descriptors `0`, `1`, and `2` are normally open.

### Repeated Calls

A useful stress test is to call `ft_popen()` repeatedly:

```c
int i = 0;

while (i < 100)
{
    int fd = ft_popen("true",
        (char *const []){"true", NULL}, 'r');

    if (fd == -1)
        return (1);
    close(fd);
    i++;
}
```

This checks whether the parent keeps accumulating open pipe descriptors.

However, it does not prove that the program has no resource-management issues. In particular, child processes that have terminated but have not been reaped may become zombies while the parent remains alive.

Because `wait()` and `waitpid()` are not allowed in `ft_popen()`, child reaping is outside this exercise's implementation.

## 13. Common Mistakes

| Mistake | Consequence |
|---|---|
| Forgetting to close unused pipe ends | FD leaks or missing EOF |
| Reversing `fd[0]` and `fd[1]` | Incorrect data flow |
| Calling `dup2()` with reversed arguments | Incorrect redirection |
| Calling `return` after child-side failure | Child continues caller execution |
| Forgetting `NULL` at the end of `argv` | Invalid argument array |
| Forgetting to close the returned FD | Caller-side resource leak |
| Using `size_t` for `read()` results | Incorrect error handling |
| Assuming `execvp()` returns on success | Incorrect process logic |

## 14. Final Review

Before moving on to the next exercise, make sure you can answer these questions without looking at the solution.

- [ ] What is the difference between `fd[0]` and `fd[1]`?
- [ ] What does `fork()` return in the parent and child?
- [ ] Why do both processes initially have both pipe ends?
- [ ] Which descriptor should the parent retain in `'r'` mode?
- [ ] Which descriptor should the parent retain in `'w'` mode?
- [ ] Why do we use `dup2()` before `execvp()`?
- [ ] Why can we close the original descriptor after a successful `dup2()`?
- [ ] Why does `execvp()` not return on success?
- [ ] Why should the child use `exit()` instead of `return` on failure?
- [ ] Why is closing the pipe's write end important for EOF?
- [ ] Why can the parent return a valid descriptor even when `execvp()` fails?

### Key Takeaway

The complete workflow can be summarized as:

```text
Validate parameters
        │
        ▼
    Create pipe
        │
        ▼
    Fork process
        │
        ├── CHILD
        │     │
        │     ├── Close unused pipe end
        │     ├── Redirect stdin/stdout
        │     ├── Close original pipe end
        │     ├── Execute command
        │     └── Exit if execution fails
        │
        └── PARENT
              │
              ├── Close unused pipe end
              └── Return remaining FD
```

The most important concept in this exercise is understanding that **`pipe()` creates the communication channel, `fork()` creates the second process, `dup2()` connects the channel to standard input/output, and `execvp()` launches the command.**

Once you understand how these four system calls work together, you can apply the same principles to shell pipelines, process redirection, and inter-process communication.
