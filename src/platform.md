# Platform Configuration

Before benchmarking, it's important to check the platform configuration is optimal for the target benchmark.
The optimal configuration may vary by benchmark, but there are some common high-level settings to be aware of.
For a more comprehensive guide to platform tuning, see the [NVIDIA Grace Hopper and Grace CPU Platform Tuning Guide](FIXME).


## CPU and Memory

Use the `performance` CPU frequency governor:
```bash
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

Disable address space layout randomization (ASLR):
```bash
sudo sysctl -w kernel.randomize_va_space=0
```

Drop caches:
```bash
echo 3 > /proc/sys/vm/drop_caches
```

 Set kernel dirty page values to defaults:
```bash
echo 10 > /proc/sys/vm/dirty_ratio
echo 5 > /proc/sys/vm/dirty_background_ratio
```

Check for dirty page writeback every 60 seconds to reduce disk I/O (the default is 5 seconds):
```bash
echo 6000 > /proc/sys/vm/dirty_writeback_centisecs
```

Disable the NMI watchdog
```bash
echo 0 > /proc/sys/kernel/watchdog
```


## Networking

Set networking connection tracking size:
```bash
echo 512000 | sudo tee /proc/sys/net/netfilter/nf_conntrack_max
```

Allow the kernel to reuse TCP ports which may be in a TIME_WAIT state before kicking off the test:
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_tw_reuse
```


## Device I/O

Full power for generic devices:
```bash
for i in `find /sys/devices/*/power/control` ; do
    echo 'on' > ${i}
done
```

Full power for PCI devices:
```bash
for i in `find /sys/bus/pci/devices/*/power/control` ; do
    echo 'on' > ${i}
done
```