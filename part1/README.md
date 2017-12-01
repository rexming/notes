# 反射
作用：让我们在运行期探知对象的类型信息和内存结构，从一定程度上弥补了go在静态语言在动态行为上的不足。同时反射还是实现元编程的重要手段

func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value

+ Type : 表示真实类型（静态类型）
+ Kind : 表示基础结构（底层类型）

底层Kind类型有：

	Bool | Int | Int8 | Int16 | Int32 | Int64
	--- |
	Uint|Uint8|Uint16|Uint32|Uint64|Uintptr
	Float32|Float64	|Complex64|Complex128|Array
	Chan|Func|Interface|Map|Ptr|Slice|String
	Struct|UnsafePointer

除通过实际对象获取类型外，可以通过反射直接构造一些基础复合类型
```
s := reflect.ArrayOf(10, reflect.TypeOf("st"))
fmt.Println(s)
>>> [10]string
```
