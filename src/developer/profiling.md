# Profiling

If you are not getting the performance you expect, one of the best ways to understand what is
going on in the system is to compare profiles of execution and understand where the CPU cores are
spending time. This will frequently point to a hot function that could be optimized. 

1. Install the Linux perf tool.

        ```bash
        # Redhat
        sudo yum install perf

        # Ubuntu
        sudo apt-get install linux-tools-$(uname -r)
        ```

2. Record a profile.

        ```bash
        # If the program is run interactively
        sudo perf record -g -F99 -o perf.data ./your_program

        # Sample all cpus (-a) and run for 60 seconds while the system is loaded
        sudo perf record -ag -F99 -o perf.data  sleep 60
        ```

3. Examine the profile.

        ```bash
        perf report
        ```
