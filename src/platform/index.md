# Platform Configuration

Before benchmarking, you should check whether the platform configuration is optimal for the target benchmark. 
The optimal configuration can vary by benchmark, but there are some common high-level settings of which you should be aware. 

```admonish info
Refer to the [NVIDIA Grace Performance Tuning Guide](https://docs.nvidia.com/grace-performance-tuning-guide.pdf)
and the platform-specific documentation at <https://docs.nvidia.com/grace/> for instructions on how to tune
your platform for optimal performance.
```

## CPU and Memory

* Use the `performance` CPU frequency governor:
  
    ```bash
    echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
    ```

* Disable address space layout randomization (ASLR):

    ```bash
    sudo sysctl -w kernel.randomize_va_space=0
    ```

* Drop the caches:
    
    ```bash
    echo 3 | sudo tee /proc/sys/vm/drop_caches
    ```

* Set the kernel dirty page values to the default values:

    ```bash
    echo 10 | sudo tee /proc/sys/vm/dirty_ratio
    echo 5 | sudo tee /proc/sys/vm/dirty_background_ratio
    ```

* To reduce disk I/O, check for dirty page writeback every 60 seconds:

    ```bash
    echo 6000 | sudo tee /proc/sys/vm/dirty_writeback_centisecs
    ```

* Disable the NMI watchdog:

    ```bash
    echo 0 | sudo tee /proc/sys/kernel/watchdog
    ```

* Optional, allow unprivileged users to measure system events.  Note that this setting has implications for system security.  See <https://www.kernel.org/doc/html/latest/admin-guide/perf-security.html> for additional information.

    ```bash
    echo 0 | sudo tee /proc/sys/kernel/perf_event_paranoid
    ```

## Networking

* Set the networking connection tracking size:

    ```bash
    echo 512000 | sudo tee /proc/sys/net/netfilter/nf_conntrack_max
    ```

* Before starting the test, allow the kernel to reuse TCP ports which may be in a `TIME_WAIT` state:

    ```bash
    echo 1 | sudo tee /proc/sys/net/ipv4/tcp_tw_reuse
    ```

## Device I/O

* For full power for generic devices:

    ```bash
    for i in `find /sys/devices/*/power/control` ; do
        echo 'on' > ${i}
    done
    ```

* For full power for PCI devices:

    ```bash
    for i in `find /sys/bus/pci/devices/*/power/control` ; do
        echo 'on' > ${i}
    done
    ```
