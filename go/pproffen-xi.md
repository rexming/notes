#主要使用go tool pprof命令来交互式的访问概要文件的内容
# CPU分析
CPU的主频，即CPU内核工作的时钟频率（CPU Clock Speed）。CPU的主频的基本单位是赫兹（Hz），但更多的是以兆赫兹（MHz）或吉赫兹（GHz）为单位。时钟频率的倒数即为时钟周期。时钟周期的基本单位为秒（s），但更多的是以毫秒（ms）、微妙（us）或纳秒（ns）为单位。在一个时钟周期内，CPU执行一条运算指令。也就是说，在1000 Hz的CPU主频下，每1毫秒可以执行一条CPU运算指令。在1 MHz的CPU主频下，每1微妙可以执行一条CPU运算指令。而在1 GHz的CPU主频下，每1纳秒可以执行一条CPU运算指令。
在默认情况下，Go语言的运行时系统会以100 Hz的的频率对CPU使用情况进行取样。也就是说每秒取样100次，即每10毫秒会取样一次。为什么使用这个频率呢？因为100 Hz既足够产生有用的数据，又不至于让系统产生停顿。并且100这个数上也很容易做换算，比如把总取样计数换算为每秒的取样数。实际上，这里所说的对CPU使用情况的取样就是对当前的Goroutine的堆栈上的程序计数器的取样。由此，我们就可以从样本记录中分析出哪些代码是计算时间最长或者说最耗CPU资源的部分了
```
	var cpuprofile = flag.String("cpuprofile", "", 				   "write cpu profile to file")
	if *cpuprofile != "" {
		f, err := os.Create(*cpuprofile)
		if err != nil {
			log.Fatal(err)
		}
		pprof.StartCPUProfile(f)
		defer pprof.StopCPUProfile()
	}
	
	通过命令行参数决定是否产生cpu概要文件
	pprof.StopCPUProfile函数通过把CPU使用情况取样的频率设置为0来停止取样操作。并且，只有当所有CPU使用情况记录都被写入到CPU概要文件之后，pprof.StopCPUProfile函数才会退出。从而保证了CPU概要文件的完整性。
```
# 内存分析
内存概要文件用于保存在用户程序执行期间的内存使用情况。这里所说的内存使用情况，其实就是程序运行过程中堆内存的分配情况。Go语言运行时系统会对用户程序运行期间的所有的堆内存分配进行记录。不论在取样的那一时刻、堆内存已用字节数是否有增长，只要有字节被分配且数量足够，分析器就会对其进行取样。开启内存使用情况记录的方式如下
```
func startMemProfile() {
    if *memProfile != "" && *memProfileRate > 0 {
        runtime.MemProfileRate = *memProfileRate
    }
}
```
开启内存使用情况记录的方式非常简单。在函数startMemProfile中，只有在\*memProfile和\*memProfileRate的值有效时才会进行后续操作。\*memProfile的含义是内存概要文件的绝对路径。\*memProfileRate的含义是分析器的取样间隔，单位是字节。当我们将这个值赋给int类型的变量runtime.MemProfileRate时，就意味着分析器将会在每分配指定的字节数量后对内存使用情况进行取样。实际上，即使我们不给runtime.MemProfileRate变量赋值，内存使用情况的取样操作也会照样进行。此取样操作会从用户程序开始时启动，且一直持续进行到用户程序结束。runtime.MemProfileRate变量的默认值是512 * 1024，即512K个字节。只有当我们显式的将0赋给runtime.MemProfileRate变量之后，才会取消取样操作

在默认情况下，内存使用情况的取样数据只会被保存在运行时内存中，而保存到文件的操作只能由我们自己来完成:
```
func stopMemProfile() {
    if *memProfile != "" {
        f, err := os.Create(*memProfile)
        if err != nil {
            fmt.Fprintf(os.Stderr, "Can not create mem profile output file: %s", err)
            return
        }
        if err = pprof.WriteHeapProfile(f); err != nil {
            fmt.Fprintf(os.Stderr, "Can not write %s: %s", *memProfile, err)
        }
        f.Close()
    }
}
```
# 程序阻塞