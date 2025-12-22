# assignments

{% stepper %}
{% step %}
### Biggest of Three Numbers

Write a program or script that takes three numbers as input and prints the largest one.

* Accept inputs via command-line arguments, stdin, or prompts.
* Handle equal numbers appropriately (e.g., print the value).
{% endstep %}

{% step %}
### File / Directory / Link Check

Given a path, detect whether it's a file, directory, or symbolic link and perform the corresponding action.

If it’s a file:

* Display the file contents.
* Count and show the number of lines.

If it’s a directory:

* List all files inside (optionally recursively or non-recursively; specify behavior in your implementation).

If it’s a symbolic link:

* Indicate that it is a symlink and show the target.

Consider handling errors (non-existent paths, permission issues).
{% endstep %}

{% step %}
### Basic Calculator

Create a calculator that performs operations on two numbers. Support at least:

* Addition
* Subtraction
* Multiplication
* Division (handle division-by-zero safely)

Accept inputs and the requested operation (e.g., via flags, menu, or command-line args) and output the result.
{% endstep %}

{% step %}
### Factorial of a Number

Compute the factorial of a non-negative integer.

* Handle n = 0 (0! = 1).
* Consider iterative and recursive implementations.
* Validate input (non-negative integers).
{% endstep %}

{% step %}
### Sum of a Given Set of Numbers

Sum a list of numbers provided as input arguments.

* Accept numbers via command-line arguments or a single input string.
* Validate inputs and skip or report non-numeric entries as appropriate.
* Output the total sum.
{% endstep %}

{% step %}
### Check Services Running or Stopped

Given one or more service names (or identifiers), report whether each service is running or stopped.

* Use platform-appropriate methods (e.g., systemctl on systemd Linux, service / init scripts, or OS-specific APIs).
* For each service, print status (running/stopped/not found) and optionally exit codes or logs when available.
* Handle permission requirements (may need elevated privileges).
{% endstep %}

{% step %}
### Retain Recent 20 Files, Delete Others

In a given directory, keep the most recent 20 files and delete the rest.

* Define "recent" (e.g., based on modification time).
* Exclude directories (or define desired behavior for directories).
* Provide a dry-run option to list files that would be deleted without deleting them.
* Handle edge cases: fewer than 20 files, permission errors, and symlinks.
{% endstep %}
{% endstepper %}
