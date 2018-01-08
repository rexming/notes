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

```
