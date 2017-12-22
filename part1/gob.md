# GOB 
golang中使用的二进制序列化的包，类似python pickle,java serialization ,常用于rpc远程方法调用

## basic
* 为各种数据类型实现了自定义编码
* 对于数据流，流的最前面是对数据类型的描述
* 指针不能被传递，但指针所指可以，that is, the values are flattened（不懂这句话改如何翻译）.空指针不被允许，因为它没有值。Recursive types work fine, but recursive values (data with cycles) are problematic. This may change.
* To use gobs, create an Encoder and present it with a series of data items as values or addresses that can be dereferenced to values.

struct { A, B int }
can be sent from or received into any of these Go types:

struct { A, B int }	// the same
*struct { A, B int }	// extra indirection of the struct
struct { *A, **B int }	// extra indirection of the fields
struct { A, B int64 }	// different concrete value type; see below
It may also be received into any of these:

struct { A, B int }	// the same
struct { B, A int }	// ordering doesn't matter; matching is by name
struct { A, B, C int }	// extra field (C) ignored
struct { B int }	// missing field (A) ignored; data will be dropped
struct { B, C int }	// missing field (A) ignored; extra field (C) ignored.
Attempting to receive into these types will draw a decode error:

struct { A int; B uint }	// change of signedness for B
struct { A int; B float }	// change of type for B
struct { }			// no field names in common
struct { C, D int }		// no field names in common
  
# Type and Value
send | receive
---- | -------
任意符号整数（没有int8,int16 etc） | int,int16...
unsigned integers|unsigned integer variable
 floating point | floating point
 struct(仅适用于导出字段)|...
 arrays/slice  (字符串数组和字节数组是特别的)| ...
 Functions and channels will not be sent in a gob。 A struct field of chan or func type is treated exactly like an unexported field and is ignored.


## 用gob实现结构体的深拷贝
```
func deepCopy(dst, src interface{}) error {
    var buf bytes.Buffer
    if err := gob.NewEncoder(&buf).Encode(src); err != nil {
        return err
    }
    return gob.NewDecoder(bytes.NewBuffer(buf.Bytes())).Decode(dst)
}
```
同理，用json也可以：先把struct序列化保存到一个buffer里面，然后将buffer里面的Byte()数据进行反序列化到指定对象里面即可