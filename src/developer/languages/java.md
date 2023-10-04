# Java on NVIDIA Grace

Java is well supported and generally performant out-of-the-box on Arm64. While Java 8 is fully supported on Arm64, some customers have not been able to obtain the CPU's full performance benefit until after switching to Java 11.

This section includes specific details about building and tuning Java applications on Arm64.

## Java JVM Options
There are options that control the JVM and might lead to better performance.  Flags `-XX:-TieredCompilation -XX:ReservedCodeCacheSize=64M -XX:InitialCodeCacheSize=64M` have shown large (1.5x) improvements in some Java workloads.  `ReservedCodeCacheSize` and `InitialCodeCacheSize` should be equal and between 64M and 127M. The JIT compiler stores generated code in the code cache. The flags change the size of the code cache from the default 240M to the smaller one. The smaller code cache may help CPU to improve the caching and prediction of JIT'ed code. The flags disable the tiered compilation to make the JIT compiler able to use the smaller code cache. These are helpful on some workloads but can hurt on others so testing with and without them is essential.   

## Java Stack Size
For some JVMs, the default stack size for Java threads (`ThreadStackSize`) is 2MB on Arm64 instead of the 1MB used on x86. You can check the default with the following:
```
$ java -XX:+PrintFlagsFinal -version | grep ThreadStackSize
     intx CompilerThreadStackSize = 2048  {pd product} {default}
     intx ThreadStackSize         = 2048  {pd product} {default}
     intx VMThreadStackSize       = 2048  {pd product} {default}
```
The default can be easily changed on the command line with either `-XX:ThreadStackSize=<kbytes>` or `-Xss<bytes>`. Notice that `-XX:ThreadStackSize` interprets its argument as kilobytes and `-Xss` interprets it as bytes. As a result, `-XX:ThreadStackSize=1024` and `-Xss1m` will both set the stack size for Java threads to 1 megabyte:
```
$ java -Xss1m -XX:+PrintFlagsFinal -version | grep ThreadStackSize
     intx CompilerThreadStackSize                  = 2048                                   {pd product} {default}
     intx ThreadStackSize                          = 1024                                   {pd product} {command line}
     intx VMThreadStackSize                        = 2048                                   {pd product} {default}
```

Typically, you do not have to change the default value because the thread stack will be committed lazily as it grows. Regardless of the default value, the thread will always only commit as much stack as it really uses (at page size granularity). However, there is one exception to this rule. If [Transparent Huge Pages](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html) (THP) are turned on by default on a system, the stack will be completely committed to memory from the start. If you are using hundreds, or even thousands of threads, this memory overhead can be considerable.

To mitigate this issue, you can either manually change the stack size on the command line (as described above) or you can change the default for THP from `always` to `madvise`:
```bash
echo madvise | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

Even if the the default is changed from `always` to `madvise`, if you specify `-XX:+UseTransparentHugePages` on the command line, the JVM can still use THP for the Java heap and code cache.