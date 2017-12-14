## 提供原子操作，同步，直接操作指针地址
* The swap operation, implemented by the SwapT functions, is the atomic equivalent of:

```
    old = *addr
    *addr = new
    return old

```

*  The compare-and-swap operation, implemented by the CompareAndSwapT functions, is the atomic equivalent of:
```
    if *addr == old {
	*addr = new
	return true
    }
    return false
```
*  The add operation, implemented by the AddT functions, is the atomic equivalent of:
```
    *addr += delta
    return *addr
```

*  The load and store operations, implemented by the LoadT and StoreT functions, are the atomic equivalents of "return *addr" and "*addr = val".

# fds