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
(目前感觉没什么用。。。)
```
s := reflect.ArrayOf(10, reflect.TypeOf("st"))
fmt.Println(s)
>>> [10]string
```

传入对象区分基类型和指针类型
```
	x := 100
	tx,tp := reflect.TypeOf(x),reflect.TypeOf(&x)
	
	fmt.Println(tx,tp,tx==tp)
	fmt.Println(tx.Kind(),tp.Kind())
	fmt.Println(tx == tp.Elem())
     >>int *int false
       int ptr
       true  
       ------------------
       	x := make([]chan *struct{A string},0)
	tx,tp := reflect.TypeOf(x),reflect.TypeOf(&x)
	
	fmt.Println(tx,tp,tx==tp)
	fmt.Println(tx.Kind(),tp.Kind())
	fmt.Println(tx == tp.Elem())
     >>[]chan *struct { A string } *[]chan *struct { A          string } false
       slice ptr
       true
```
方法Elem()返回指针，数组，切片，字典或通道的基类型
```
// Elem returns a type's element type.
// It panics if the type's Kind is not Array, Chan, Map, Ptr, or Slice.

```	

只有获取结构体指针的基类型后，才能遍历它的字段
```
type Person struct {
	Name string
	age  int `json:"age"`
}
func main() {
	x :=  Person{Name:"x"}
	tx,tp := reflect.TypeOf(x),reflect.TypeOf(&x)
	
	fmt.Println(tx,tp,tx==tp)
	fmt.Println(tx.Kind(),tp.Kind())
	fmt.Println(tx == tp.Elem())
	fmt.Println("----")
	
	for i:=0;i<tx.NumField();i++ {
		fmt.Printf("第%d个字段",i)
		fmt.Println(tx.Field(i).Name)
		fmt.Println(tx.Field(i).Type)
		fmt.Println(tx.Field(i).Anonymous)
		fmt.Println(tx.Field(i).Index)
		fmt.Println(tx.Field(i).Offset)
		fmt.Println(tx.Field(i).PkgPath)
		fmt.Println(tx.Field(i).Tag)
	}
	
}
>>
	main.Person *main.Person false
	struct ptr
	true
	----
	第0个字段Name
	string
	false
	[0]
	0


	第1个字段age
	int
	false
	[1]
	16
	main
	json:"age"
	
```
访问匿名字段
1.通过多级索引直接访问
```
	// FieldByIndex returns the nested field corresponding
	// to the index sequence. It is equivalent to calling Field
	// successively for each index i.
	// It panics if the type's Kind is not Struct.
	FieldByIndex(index []int) StructField
```
2.通过字段进行访问

同样的输出方法集时，也区分基类型和指针类型

反射能探知当前包的非导出结构成员


# 值
Value 专注于对象实例数据的读写
接口变量会复制对象，而且是unaddressable的，所以要想修改目标对象，必须要使用指针
就算传入指针，一样需要Elem获取目标对象，因为被接口存储的指针本身是不能寻址和进行设置操作的。
```
// Elem returns the value that the interface v contains
// or that the pointer v points to.
// It panics if v's Kind is not Interface or Ptr.
// It returns the zero Value if v is nil.
```
不能对非导出字段直接进行设置操作，无论是当前包还是外包
```
type Person struct {
	Name string
	age  int `json:"age"`
}
func main() {
	p := Person{}
	va,vp := reflect.ValueOf(p),reflect.ValueOf(&p)
	//fmt.Println(va.Kind(),vp.Kind())
	//fmt.Println(va.Elem())
	_ = va
	vp = vp.Elem()
	
	for i:=0;i<vp.NumField();i++ {
		fmt.Println(vp.Field(i).CanAddr())
		fmt.Println(vp.Field(i).CanSet())
	}
	
	name := vp.FieldByName("Name")
	age := vp.FieldByName("age")
	value := reflect.ValueOf("ai")
	name.Set(value)
	fmt.Println(p)
	name.SetString("oi")
	fmt.Println(p)
	
	px := (*string)(unsafe.Pointer(age.UnsafeAddr()))
	fmt.Println(px,*px=="")
	*px = "12"
	fmt.Printf("%v",p)
	
	pint := (*int)(unsafe.Pointer(age.UnsafeAddr()))
	fmt.Println(pint,*pint)
	*pint = 12
	fmt.Printf("%v",p)
	
	age.Pointer() //返回point
	fmt.Println(age.Int()) //返回12
	
}
>>
	true
	true
	true
	false
	{ai 0}
	{oi 0}
	0xc0420023f0 true
	{oi 5098856}0xc0420023f0 5098856
	{oi 12}
```

PS：重要：可以通过Interface方法进行类型推断和转换

复合类型对象设置示例
```
chan
 	c := make(chan int)
	v := reflect.ValueOf(c)
	
	if v.TrySend(reflect.ValueOf(100)) {
		fmt.Println(v.TryRecv())
	}
map
	c := make(map[int]string)
	v := reflect.ValueOf(c)
	v.SetMapIndex(reflect.ValueOf(1),reflect.ValueOf("s"))
	fmt.Println(c)  //1:s
slice
	v.slice() 方法
```

接口的两种状态，解决方法是用IsNil判断值是否为nil
```
	var a interface{} = nil
	var b interface{} = (*int)(nil)
	fmt.Println(a == nil )  // true
	fmt.Println(b == nil,reflect.ValueOf(b).IsNil() ) //false,true  

```

# 反射使用示例

1.常用于ORM

创建一个QueryBuilder 结构体，包含用于生成SQL查询的类型的元数据信息


```
type QueryBuilder struct { Type reflect.Type }

```
定义一个员工类

```
type Employee struct {
	ID        uint32
	FirstName string
	LastName  string
	Birthday  time.Time
}
```
用reflect.typeof获取元信息,
注意因为是是指针类型，所以我们需要用elem()方法获取底层真实类型。如果type's kind不是array,map,ptr或者slice,elem方法会panic
```
t := reflect.TypeOf(&Employee{}).Elem() builder := &QueryBuilder{Type: t}
```
我们来实现生成一个查询语句的方法

```
func (qb *QueryBuilder) CreateSelectQuery() string {
	buffer := bytes.NewBufferString("")

	for index := 0; index < qb.Type.NumField(); index++ {
		field := qb.Type.Field(index)

		if index == 0 {
			buffer.WriteString("SELECT ")
		} else {
			buffer.WriteString(", ")
		}
		column := field.Name
		buffer.WriteString(column)
	}

	if buffer.Len() > 0 {
		fmt.Fprintf(buffer, " FROM %s", qb.Type.Name())
	}

	return buffer.String()
}
```
最终我们会生成这样的sql 语句对于查询Employee 类
```
SELECT ID, FirstName, LastName, Birthday FROM Employee
```
但是如何解觉数据库中真实字段和Employee不同的情况，比如我们想用id字段代替id_pk,firstName代替first_name;所以我们需要使用field tags字段实现映射。
标签的使用强烈取决于结构体的使用，典型用途是为持久化或序列化添加规范或约束。如json的编码和解码；
让我们更改Employee结构声明，以使用携带有关字段如何映射到基础数据库的附加信息的标记

```
type Employee struct {
	ID        uint32 `orm:"id_pk"`
	FirstName string `orm:"first_name"`
	LastName  string `orm:"last_name"`
	Birthday  time.Time
}
```
我们通过使用field.Tag中Get方法获取tag

```
column := field.Name
if tag := field.Tag.Get("orm"); tag != "" {
	column = tag
}

buffer.WriteString(column)
```
这样产生的sql 命令就是
SELECT id_pk, first_name, last_name, Birthday FROM Employee

2.用于字段校验
假设我们这样一个结构体

```
type PaymentTransaction struct {
	Amount      float64 `validate:"positive"`
	Description string  `validate:"max_length:250"`
}
```

像上一个例子一样，我们使用标签注释，然后我们实现验证功能

```
func Validate(obj interface{}) error {
	v := reflect.ValueOf(obj).Elem()
	t := v.Type()

	for index := 0; index < v.NumField(); index++ {
		vField := v.Field(index)
		tField := t.Field(index)

		tag := tField.Tag.Get("validate")
		if tag == "" {
			continue
		}

		switch vField.Kind() {
		case reflect.Float64:
			value := vField.Float()
			if tag == "positive" && value < 0 {
				value = math.Abs(value)
				vField.SetFloat(value)
			}
		case reflect.String:
			value := vField.String()
			if tag == "upper_case" {
				value = strings.ToUpper(value)
				vField.SetString(value)
			}
		default:
			return fmt.Errorf("Unsupported kind '%s'", vField.Kind())
		}
	}

	return nil
}
```
reflect.value 用来获取某一对象的所有成员
通过使用kind方法获取字段类型，然后我们可以获取相对应的值。可以通过set...方法设置值

3.识别接口和调用函数
通过反射来识别是否实现某一特定接口
比如验证的接口

```
type Validator interface {
	Validate() error
}
```
扩展`PaymentTransaction`使实现接口

```
func (p *PaymentTransaction) Validate() error {
	fmt.Println("Validating payment transaction")
	return nil
}
```
为了确定PaymentTransaction是否实现接口，需要调用reflect.type 的Implements方法，

为了调用特定的方法，可以将对象映射到Validator接口或通过MethodByName函数获取方法

```
func CustomValidate(obj interface{}) error {
	v := reflect.ValueOf(obj)
	t := v.Type()

	interfaceT := reflect.TypeOf((*Validator)(nil)).Elem()
	if !t.Implements(interfaceT) {
		return fmt.Errorf("The Validator interface is not implemented")
	}

	validateFunc := v.MethodByName("Validate")
	validateFunc.Call(nil)
	return nil
}
```

参考来源：
1.go语言学习笔记，雨痕
2.http://blog.ralch.com/


