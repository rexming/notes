# 官方说法
从一个正确的解释点来说，不需要知道。
准确地说，你并不需要知道。Golang 中的变量只要被引用就一直会存活，存储在堆上还是栈上由内部实现决定而和具体的语法没有关系。

知道变量的存储位置确实和效率编程有关系。如果可能，Golang 编译器会将函数的局部变量分配到函数栈帧（stack frame）上。然而，如果编译器不能确保变量在函数 return 之后不再被引用，编译器就会将变量分配到堆上来避免指针错误。而且，如果一个局部变量非常大，那么它也应该被分配到堆上而不是栈上。

当前情况下，如果一个变量被取地址，那么它就有可能被分配到堆上。然而，还要对这些变量做逃逸分析，如果函数 return 之后，变量不再被引用，则将其分配到栈上。
# 例子1（来源）
```
func main() {
	var a [1]int
	c := a[:]
	fmt.Println(c)
}

```
通过命令查看其汇编代码：
go tool compile -S demo1.go
```
"".main STEXT size=200 args=0x0 locals=0x60
        0x0000 00000 (demo1.go:5)       TEXT    "".main(SB), $96-0
        0x0000 00000 (demo1.go:5)       MOVQ    TLS, CX
        0x0009 00009 (demo1.go:5)       MOVQ    (CX)(TLS*2), CX
        0x0010 00016 (demo1.go:5)       CMPQ    SP, 16(CX)
        0x0014 00020 (demo1.go:5)       JLS     190
        0x001a 00026 (demo1.go:5)       SUBQ    $96, SP
        0x001e 00030 (demo1.go:5)       MOVQ    BP, 88(SP)
        0x0023 00035 (demo1.go:5)       LEAQ    88(SP), BP
        0x0028 00040 (demo1.go:5)       FUNCDATA        $0, gclocals·69c1753bd5f81501d95132d08af04464(SB)
        0x0028 00040 (demo1.go:5)       FUNCDATA        $1, gclocals·57cc5e9a024203768cbab1c731570886(SB)
        0x0028 00040 (demo1.go:5)       LEAQ    type.[1]int(SB), AX
        0x002f 00047 (demo1.go:6)       MOVQ    AX, (SP)
        0x0033 00051 (demo1.go:6)       PCDATA  $0, $0
        0x0033 00051 (demo1.go:6)       CALL    runtime.newobject(SB)
        0x0038 00056 (demo1.go:6)       MOVQ    8(SP), AX
        0x003d 00061 (demo1.go:8)       MOVQ    AX, ""..autotmp_4+64(SP)
        0x0042 00066 (demo1.go:8)       MOVQ    $1, ""..autotmp_4+72(SP)
        0x004b 00075 (demo1.go:8)       MOVQ    $1, ""..autotmp_4+80(SP)
        0x0054 00084 (demo1.go:8)       MOVQ    $0, ""..autotmp_3+48(SP)
        0x005d 00093 (demo1.go:8)       MOVQ    $0, ""..autotmp_3+56(SP)
        0x0066 00102 (demo1.go:8)       LEAQ    type.[]int(SB), AX
        0x006d 00109 (demo1.go:8)       MOVQ    AX, (SP)
        0x0071 00113 (demo1.go:8)       LEAQ    ""..autotmp_4+64(SP), AX
        0x0076 00118 (demo1.go:8)       MOVQ    AX, 8(SP)
        0x007b 00123 (demo1.go:8)       PCDATA  $0, $1
        0x007b 00123 (demo1.go:8)       CALL    runtime.convT2Eslice(SB)
        0x0080 00128 (demo1.go:8)       MOVQ    24(SP), AX
        0x0085 00133 (demo1.go:8)       MOVQ    16(SP), CX
        0x008a 00138 (demo1.go:8)       MOVQ    CX, ""..autotmp_3+48(SP)
        0x008f 00143 (demo1.go:8)       MOVQ    AX, ""..autotmp_3+56(SP)
        0x0094 00148 (demo1.go:8)       LEAQ    ""..autotmp_3+48(SP), AX
        0x0099 00153 (demo1.go:8)       MOVQ    AX, (SP)
        0x009d 00157 (demo1.go:8)       MOVQ    $1, 8(SP)
        0x00a6 00166 (demo1.go:8)       MOVQ    $1, 16(SP)
        0x00af 00175 (demo1.go:8)       PCDATA  $0, $1
        0x00af 00175 (demo1.go:8)       CALL    fmt.Println(SB)
        0x00b4 00180 (demo1.go:9)       MOVQ    88(SP), BP
        0x00b9 00185 (demo1.go:9)       ADDQ    $96, SP
        0x00bd 00189 (demo1.go:9)       RET
        0x00be 00190 (demo1.go:9)       NOP
        0x00be 00190 (demo1.go:5)       PCDATA  $0, $-1
        0x00be 00190 (demo1.go:5)       CALL    runtime.morestack_noctxt(SB)
        0x00c3 00195 (demo1.go:5)       JMP     0

```
注意runtime.newobject！其中demo1.go:6说明变量a的内存是在堆上分配的

运行go tool compile -m demo1.go 
参数-m是打印出编译优化
```
demo1.go:8:13: c escapes to heap
demo1.go:7:8: a escapes to heap
demo1.go:6:6: moved to heap: a
demo1.go:8:13: main ... argument does not escape
```
以上可以明显看出编译器的逃逸分析优化

```
func main() {
    var a [1]int
    c := a[:]
    println(c)
}
```
go tool compile -m demo1.go
```
demo1.go:3:6: can inline main
demo1.go:5:8: main a does not escape

```
可以看出a,c都在栈上分配
这是因为fmt.Println打印slice时会用到reflect.Type.Kind()，这是个接口方法，导致传入fmt.Println的参数被分配在堆上。

# 例2（来源知乎https://zhuanlan.zhihu.com/p/32686933?utm_source=qq&utm_medium=social）

一段会使内存不断增长的代码
```
type Node struct {
	next *Node
	payload [64]byte
}

func main() {
	curr := new(Node)
	for {
		curr.next = new(Node)
		curr = curr.next
	}
}
```
原因是第一个Node的new分配在栈上了，go编译器会对代码做逃逸分析，如果在函数中new分配一个小对象，而且这个小对象不会逃逸出去，则可以直接将其分配到栈上，所以上面代码在优化后相当于：
```
func main() {
	var n Node
	curr := &n
	for {
		curr.next = new(Node)
		curr = curr.next
	}
}
```


而当函数返回后调用GC不能及时时被回收
```
func f() {
	curr := new(Node)
	for i := 0; i < 10000000; i ++ {
		curr.next = new(Node)
		curr = curr.next
	}
}

func main() {
	f()
	runtime.GC()
	time.Sleep(time.Second * 100) //在这里查看进程内存使用
}
```
也就是说，函数返回后，栈上的数据还是会作为不可达来处理的，所以正常业务开发的时候，这个问题看起来也不算严重（从main到main_loop这一段的临时对象容易出事，注意一下即可）

解决办法是用二级指针
```
//用二级指针，new(*Node)会被逃逸优化，但new(Node)就不会了
func main() {
	curr := new(*Node)
	*curr = new(Node)
	for {
		(*curr).next = new(Node)
		*curr = (*curr).next
	}
}

//用return强制逃逸，避免优化
func f() *Node {
	curr := new(Node)
	for {
		curr.next = new(Node)
		curr = curr.next
	}
	return curr
}

func main() {
	f()
}
```
# 自己实践例2
用go trace 工具捕捉trace文件进行分析
```
func main() {
	f, _ := os.Create("trace.out")
	defer f.Close()
	trace.Start(f)
	defer trace.Stop()
	curr := new(Node)
	n := 0
	for {
		curr.next = new(Node)
		curr = curr.next
		n++
		if n == 10000000 {
			break
		}
		if n%1000 == 0 {
			runtime.GC()
		}
	}
	
}
```
当n是1000的倍数时，显示调用GC，效果没什么用

```
func main() {
	f, _ := os.Create("trace.out")
	defer f.Close()
	trace.Start(f)
	defer trace.Stop()
	curr := new(Node)
	n := 0
	for {
		curr.next = new(Node)
		curr = curr.next
		n++
		if n == 10000000 {
			break
		}
		time.Sleep(time.Nanosecond)
	}
	
}
```
当让休眠1纳秒却可以看出GC效果

```
func main() {
	f, _ := os.Create("trace.out")
	defer f.Close()
	trace.Start(f)
	defer trace.Stop()
	curr := new(Node)
	n := 0
	for {
		curr.next = new(Node)
		curr = curr.next
		n++
		if n == 10000000 {
			break
		}
		if n == 10000 || n == 1000000 || n == 2000000 {
			runtime.GC()
			//time.Sleep(time.Millisecond)
		}
	}
	
}
```
当两次GC调用之间存在较大内存扩增时，也可以观察出明显的GC效果

看来还是对内存管理和GC机制不熟