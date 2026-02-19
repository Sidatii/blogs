---
title: Scripting for DevOps - Bash fundamentals
category: DevOps
tags: Scripting - Bash - automation - Beginner
excerpt: A series of scripting fundamentals using bash for devops engineers.
published_at: 2026-02-20
---

# Bash fundamentals

# Introduction

Bash is a powerful shell and scripting language used in many Unix-like operating systems. It provides a command-line interface for users to interact with the system, execute commands, and automate tasks through scripts. Bash is widely used in automation in complex environments because of it is built in every Unix-like system, it's simplicity and portability. In this blog , we are discovering this powerful tool and how to leverage this power in order to be a better DevOps engineer. Let's start.

## Bash interpreter

The Bash interpreter is the program that reads and executes commands written in the Bash scripting language. It can be invoked in several ways:

- **Interactive mode**: When you open a terminal, the Bash interpreter starts in interactive mode, allowing you to enter commands directly.

- **Script mode**: You can run a Bash script by executing it directly or using the `bash` command followed by the script filename. For example:

  ```bash
  bash myscript.sh
  ```
- **Shebang**: If a script file starts with a shebang (`#!`) followed by the path to the Bash interpreter (e.g., `#!/bin/bash`), you can make the script executable and run it directly:

  ```bash
  chmod +x myscript.sh
  ./myscript.sh
  ```
- **Environment variables**: The Bash interpreter can be configured using environment variables, such as `BASH_ENV`, which specifies the path to a file that contains commands to be executed before the script runs.

## Basic script

A basic Bash script is a text file containing a series of commands that the Bash interpreter can execute. Here’s a simple example:

```bash
#!/bin/bash
# This is a comment
echo "Hello, World!"  # Print a message to the terminal
```

### Notes

> [!NOTE] - The first line (`#!/bin/bash`) is the shebang, indicating that the script should be run using the Bash interpreter.
> - Comments start with `#` and are ignored by the interpreter.


# Variables

Bash variables are used to store data that can be referenced and manipulated within scripts or command-line sessions. Variables in Bash do not require explicit declaration of their type, and they can hold strings, numbers, or other data types.

## Defining Variables

To define a variable in Bash, you simply assign a value to a name without spaces around the `=` sign. For example:

```bash
my_variable="Hello, World!"
```
To access the value of a variable, prefix its name with a dollar sign (`$`):

```bash
echo $my_variable
```

This will output: Hello, World!

### Variable types

Variables in Bash can be categorized into several types:

- **String Variables**: Store text data.

  ```bash
  name="Alice"
  echo $name
  ```
- **Integer Variables**: Store numeric data.

  ```bash
  count=10
  echo $count
  ```
- **Array Variables**: Store multiple values in a single variable.

  ```bash
  fruits=("apple" "banana" "cherry")
  echo ${fruits[0]}  # Outputs: apple
  ```

- **Environment Variables**: Special variables that affect the behavior of the shell and processes. They are typically defined in uppercase.

  ```bash
  # the value assigned to PATH is a list of directories where the shell looks for executable files
  # The path is assigned recursively, meaning that the directories are separated by colons (:)
  export PATH="/usr/local/bin:$PATH"
  echo $PATH
  ```

## Using Variables

You can use variables in various contexts, such as in commands, scripts, or conditional statements. Here are some examples:

```bash
echo "The value of my_variable is: $my_variable"
```
This will output:

```
The value of my_variable is: Hello, World!
```

```bash
if [ $count -gt 5 ]; then
    echo "Count is greater than 5"
else
    echo "Count is 5 or less"
fi
```

This will output:

```
Count is greater than 5
```

## Special Variables

Bash has several special variables that provide information about the script or the shell environment:

- `$?`: The exit status of the last command executed.
```bash
echo "Exit status of the last command: $?"
```
This will output the exit status, which is `0` for success or a non-zero value for failure.

- `$#`: The number of positional parameters passed to the script or function.
```bash
echo "Number of arguments passed: $#"
```
This will output the count of arguments passed to the script.

- `$@`: All positional parameters passed to the script or function.
```bash
echo "All arguments: $@"
```
This will output all arguments passed to the script, separated by spaces.

```
All arguments: arg1 arg2 arg3
```

## Predefined Variables

Bash provides several predefined variables that can be used to access information about the script or the shell environment:

### Table representation

| Variable | Description | Example Usage |
| :--- | :--- | :--- |
| **&#36;0** | The name of the script or shell. | `echo "Script name: &#36;0"` |
| **&#36;1, &#36;2** | Positional parameters (arguments). | `echo "First argument: &#36;1"` |
| **&#36;#** | Number of positional parameters. | `echo "Number of args: &#36;#"` |
| **&#36;?** | Exit status of the last command. | `echo "Exit status: &#36;?"` |
| **&#36;@** | All parameters (separate words). | `echo "All args: &#36;@"` |
| **&#36;*** | All parameters (single word). | `echo "All args: &#36;*"` |
| **&#36;&#36;** | Process ID (PID) of current shell. | `echo "PID: &#36;&#36;"` |
| **&#36;!** | PID of last background command. | `echo "Last BG PID: &#36;!"` |
| **&#36;-** | Current shell options. | `echo "Options: &#36;-"` |
| **&#36;IFS** | Internal Field Separator. | `echo "IFS: &#36;IFS"` |
| **&#36;PS1** | Primary prompt string. | `echo "PS1: &#36;PS1"` |
| **&#36;PS2** | Secondary prompt string. | `echo "PS2: &#36;PS2"` |
| **&#36;RANDOM** | Random number (0-32767). | `echo "Random: &#36;RANDOM"` |
| **&#36;SECONDS** | Seconds since shell started. | `echo "Seconds: &#36;SECONDS"` |
| **&#36;UID** | User ID of current user. | `echo "UID: &#36;UID"` |
| **&#36;USER** | Username of current user. | `echo "User: &#36;USER"` |
| **&#36;HOME** | Home directory of current user. | `echo "Home: &#36;HOME"` |
| **&#36;PWD** | Current working directory. | `echo "PWD: &#36;PWD"` |
| **&#36;OLDPWD** | Previous working directory. | `echo "Old PWD: &#36;OLDPWD"` |
| **&#36;MAIL** | Path to the user's mailbox. | `echo "Mailbox: &#36;MAIL"` |

### Detailed representation

---
- `$0`: The name of the script or shell.

```bash
echo "Script name: $0"
```
---

- `$1`, `$2`, ...: Positional parameters representing the first, second, etc., arguments passed to the script.

```bash
echo "First argument: $1"
```
---

- `$#`: The number of positional parameters passed to the script.

```bash
echo "Number of arguments: $#"
```
---

- `$?`: The exit status of the last command executed.

```bash
echo "Exit status of the last command: $?"
```
---

-`$@`: All positional parameters passed to the script.

```bash
echo "All arguments: $@"
```
---

-`$*`: Similar to `$@`, but treats all arguments as a single word.

```bash
echo "All arguments as a single word: $*"
```
---

```
-`$$`: The process ID of the current shell.
```

```bash
echo "Current shell process ID: $$"
```
---

- `$!`: The process ID of the last background command.

```bash
echo "Process ID of the last background command: $!"
```
---

-`$-`: The current options set for the shell.

```bash
echo "Current shell options: $-"
```
---

-`$IFS`: The Internal Field Separator, which defines how Bash splits words.

```bash
echo "Internal Field Separator: $IFS"
```
---

-`$PS1`: The primary prompt string, which defines the appearance of the command prompt.

```bash
echo "Primary prompt string: $PS1"
```
---

-`$PS2`: The secondary prompt string, used for multi-line commands.

```bash
echo "Secondary prompt string: $PS2"
```
---

-`$RANDOM`: A random number between 0 and 32767.

```bash
echo "Random number: $RANDOM"
```
---

-`$SECONDS`: The number of seconds since the shell was started.

```bash
echo "Seconds since shell started: $
```
---

- `$UID`: The user ID of the current user.

```bash
echo "Current user ID: $UID"
```
---

-`$USER`: The username of the current user.

```bash
echo "Current user: $USER"
```
---

-`$HOME`: The home directory of the current user.

```bash
echo "Home directory: $HOME"
```
---

-`$PWD`: The current working directory.

```bash
echo "Current working directory: $PWD"
```
---

-`$OLDPWD`: The previous working directory.

```bash
echo "Previous working directory: $OLDPWD"
```
---

-`$MAIL`: The path to the user's mailbox.

```bash
echo "User's mailbox: $MAIL"
```

# Conclusion

Bash is a powerful tool for devops engineers and mastering it opens doors to the world of automation and managing complex infrastructure. The ability to use bash either directly in terminal or via scripts is a fundamental skill that distinguishes an implicated engineers from ones who just know the basic. In the coming blogs we are diving deeper in more complex components of the world of bash. Keep learning!
