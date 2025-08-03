# Shell Commands

This project includes a series of shell scripts demonstrating the use of basic shell commands, environment variables, and shell scripting techniques.

---

## 0. Create an Alias

**Script File:** `0-alias`

- **Purpose:** Creates a new command alias.
- **Keywords:**
  - `alias`: Used to define a command shortcut.
  - `<variable_name>`: The name of the alias.
  - `<command>`: The command the alias should execute.
- **Example:**
  ```bash
  alias ll="ls -la"
  ```

---

## 1. Print "Hello User"

**Script File:** `1-hello_you`

- **Purpose:** Prints "hello <username>" where the username is the currently logged-in Linux user.
- **Keywords:**
  - `echo`: Prints text to the terminal.
  - `$USER`: Built-in environment variable for the current user.
- **Example:**
  ```bash
  echo "hello $USER"
  ```

---

## 2. Add `/action` to the PATH

**Script File:** `2-path`

- **Purpose:** Adds `/action` as the last directory in the shell's `$PATH`.
- **Keywords:**
  - `$PATH`: Environment variable storing directories for command lookups.
- **Example:**
  ```bash
  export PATH="$PATH:/action"
  ```

---

## 3. Count Directories in PATH

**Script File:** `3-paths`

- **Purpose:** Counts the number of directories listed in `$PATH`.
- **Keywords & Commands:**
  - `echo "$PATH"`: Displays the full path.
  - `tr ':' '\n'`: Replaces colons with newlines to list each directory on a new line.
  - `wc -l`: Counts the number of lines (directories).
- **Example:**
  ```bash
  echo "$PATH" | tr ':' '\n' | wc -l
  ```

---

## 4. List Environment Variables

**Script File:** `4-global_variables`

- **Purpose:** Lists all current environment variables.
- **Command:**
  ```bash
  printenv
  ```

---

## 5. List Local Variables, Environment Variables, and Functions

**Script File:** `5-local_variables`

- **Purpose:** Lists all defined local variables, environment variables, and functions.
- **Command:**
  ```bash
  set
  ```

---

## 6. Create a Local Variable

**Script File:** `6-create_local_variable`

- **Purpose:** Creates a new local variable within the current shell session.
- **Example:**
  ```bash
  MY_VAR="Hello"
  ```

---

## 7. Create a Global Variable

**Script File:** `7-create_global_variable`

- **Purpose:** Creates and exports a global environment variable.
- **Example:**
  ```bash
  export MY_VAR="Hello"
  ```

---

## 8. Add 128 to the Value of `TRUEKNOWLEDGE`

**Script File:** `8-true_knowledge`

- **Purpose:** Adds 128 to the value stored in the `TRUEKNOWLEDGE` environment variable and prints the result.
- **Example:**
  ```bash
  echo $(($TRUEKNOWLEDGE + 128))
  ```

---
