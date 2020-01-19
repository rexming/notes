### 调整内核参数

参见[Low write performance on SLES 11 servers with large RAM](http://www.novell.com/support/kb/doc.php?id=7010287)和[Better Linux Disk Caching & Performance with vm.dirty_ratio & vm.dirty_background_ratio](http://lonesysadmin.net/2013/12/22/better-linux-disk-caching-performance-vm-dirty_ratio/)。 对于大内存来说，需要考虑调整磁盘写缓存。在`/etc/sysctl.conf`中加入

```
#磁盘写缓存上限（占总内存的百分比）
vm.dirty_ratio = 3
#内核flusher线程开始清理磁盘写缓存的上限
vm.dirty_background_ratio = 2
```

- vm.dirty_ratio默认是10。按目前的标准配置，内存通常都有4GB，这就意味着磁盘的写缓存有400MB（4GB*10%）。对于磁盘的读取操作而言，大缓存并没有问题，但是对于写操作而言，过大的写缓存将造成两个重要风险，一是意外停机时丢失数据，二是累积太多的数据在内存，一旦开始集中写入操作很容易造成长时间卡机。所以，系统内存较大时，应该将这个参数降下来。
- 调整vm.dirty_background_ratio的原理同上。这个参数一般设置为vm.dirty_ratio的1/4～1/2。



## wiki 链接 :

[https://wiki.archlinux.org/index.php/Improving_performance_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)](https://wiki.archlinux.org/index.php/Improving_performance_(简体中文))