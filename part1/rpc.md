# 官方RPC库
包rpc提供了通过网络访问一个对象的输出方法的能力。

服务器需要注册对象， 通过对象的类型名暴露这个服务。注册后这个对象的输出方法就可以远程调用，这个库封装了底层传输的细节，包括序列化(默认GOB序列化器)。

服务器可以注册多个不同类型的对象，但是注册相同类型的多个对象的时候会出错。

同时，如果对象的方法要能远程访问，它们必须满足一定的条件，否则这个对象的方法会被忽略。
五个条件为：
    * 方法的类型是可输出的(the method's type is exported)
    * 方法本身也是可输出的 （the method is exported）
    * 方法必须由两个参数，必须是输出类型或者是内建类型 (the method has twoarguments, both exported or builtin types)
    * 方法的第二个参数必须是指针类型 (the method's second argument is apointer)
    * 方法返回类型为 error (the method has return type error)所以一个输出方法的格式如下
    func (t *T) MethodName(argType T1, replyType *T2) error
这里的 T 、 T1 、 T2 能够被 encoding/gob 序列化，即使使用其它的序列化框架，将来这个需求可能回被弱化。这个方法的第一个参数代表调用者(client)提供的参数，第二个参数代表要返回给调用者的计算结果，方法的返回值如果不为空， 那么它作为一个字符串返回给调用者。
如果返回error，则reply参数不会返回给调用者。
服务器通过调用 ServeConn 在一个连接上处理请求，更典型地， 它可以创建一个network listener然后accept请求。对于HTTP listener来说，可以调用 HandleHTTP 和 http.Serve 。

客户端可以调用 Dial 和 DialHTTP 建立连接。 客户端有两个方法调用服务:Call 和 Go ，可以同步地或者异步地调用服务。
当然，调用的时候，需要把服务名、方法名和参数传递给服务器。异步方法调用 Go 通过 Done channel通知调用结果返回。
除非显示的设置 codec ,否则这个库默认使用包 encoding/gob 作为序列化框架。

# 示例
