# 5.1 Network Layer

# 5.1 网络层

The network layer is a fairly straight-forward NIO server, and will not be described in great detail. The sendfile implementation is done by giving the ```MessageSet``` interface a ```writeTo``` method. This allows the file-backed message set to use the more efficient ```transferTo``` implementation instead of an in-process buffered write. The threading model is a single acceptor thread and N processor threads which handle a fixed number of connections each. This design has been pretty thoroughly tested [elsewhere](https://engineering.linkedin.com/teams/data) and found to be simple to implement and fast. The protocol is kept quite simple to allow for future implementation of clients in other languages.

网络层是一个非常直接的NIO服务器，这里不会做详细描述。sendfile的实现是通过给```MessageSet```接口实现一个```writeTo```方法来完成的。这允许文件支持(file-backed)的消息集使用更高效的```transferTo```实现，而不是利用程序中的缓冲写入。线程模型是单个接收者线程和N个处理器线程，每个处理器线程处理固定数量的连接。这个设计已经在[其它地方](https://engineering.linkedin.com/teams/data)进行了相当完整的测试，并且其实施起来很简单，速度也很快。该协议相当简单，以便将来实现其他语言的客户端。
