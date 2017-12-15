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

# atomic.swappoint()
```
var arr = make([]int, 0)

func main() {
	//atomic.LoadPointer()
	f1()
}
//错误用法
func f1() {
	arr = append(arr, 10)
	arr = append(arr, 11)
	arr = append(arr, 12)
	arr = append(arr, 13)
	arr = append(arr, 14)
	
	fmt.Printf("%p,%v\n", &arr, arr)
	add := unsafe.Pointer(&arr)
	//addradd := &add
	new := 34
	snew := unsafe.Pointer(&new)
	old := atomic.SwapPointer(&add, snew)
	fmt.Println((*[]int)(old))
	fmt.Printf("%p\n", &old)
	fmt.Println(arr)
	fmt.Println(*(*int)(add))
	
}

//正确用法
直接将需要原子操作的变量定义为unsafe.pointer类型
var arr2 = new()
func New() unsafe.Pointer {
	a := make([]int, 0)
	return unsafe.Pointer(&a)
}

func Get() []int  {
	
	a := (*[]int)(atomic.LoadPointer(&arr2))
return *a
}

func Swap() []int  {
	old :=(*[]int)(atomic.SwapPointer(&arr2,new()))
	return *old
}
```