# deepequal 和 bytes.equal
Is there any reason why 

    reflect.DeepEqual([]byte(""), []byte(nil)) -> false 

while 

    bytes.Bytes([]byte(""), []byte(nil)) -> true 
    
解释：

reflect.DeepEqual: "An empty slice is not equal to a nil slice."
bytes.Equal: "A nil argument is equivalent to an empty slice."

DeepEqual是Go的==运算符的递归放松。

DeepEqual报告x和y是否“深深相等”，定义如下。如果以下情况之一适用，两个相同类型的值是相等的。不同类型的值永远不会相等。

当它们的相应元素深度相等时，数组值是非常相等的。

如果它们相应的字段（导出和未导出）深度相等，那么结构值是相等的。

如果两者都是零，则Func值是相等的; 否则他们并不是很平等。

如果他们拥有深刻的同等具体价值，那么界面价值是相当相等

如果地图值是相同的地图对象，或者如果它们具有相同的长度及其相应的键（使用Go等同性匹配）映射到深度相等的值，则地图值是相等的。

如果使用Go的==运算符或者指向深度相等的值，则指针值相等。

如果满足以下所有条件，则切片值是相等的：它们都是零，或者都是非零，它们具有相同的长度，并且它们指向相同基础数组的相同初始条目（即＆x [0 ] ==＆y [0]）或它们相应的元素（最大长度）是非常相等的。请注意，非零空片和零片（例如，[] byte {}和[] byte（nil））并不完全相等。

其他值 - 数字，布尔值，字符串和通道 - 如果使用Go的==运算符，则它们是相等的。