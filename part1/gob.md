# GOB 
golang中使用的二进制序列化的包，类似python pickle,java serialization ,常用于rpc远程方法调用

## basic
* 为各种数据类型实现了自定义编码
* 对于数据流，流的最前面是对数据类型的描述
* 指针不能被传递，但指针所指可以，that is, the values are flattened（不懂这句话改如何翻译）.空指针不被允许，因为它没有值。Recursive types work fine, but recursive values (data with cycles) are problematic. This may change.
* To use gobs, create an Encoder and present it with a series of data items as values or addresses that can be dereferenced to values. 

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