## Protocol Buffer Basics: Python
[原文](https://developers.google.com/protocol-buffers/docs/pythontutorial)
此指南提供了用Pythion语言使用protocol buffers的基础教程。下面将通过一些简单的例子展示：
* 在.proto文件中如何定义message.
* 如何使用protocol buffer编译器
* 如何使用Python protocol buffer API读写message

当然，这份指南并不全面。如果需要更深入了解，请查看以下文档[Protocol Buffer Language Guide](https://developers.google.com/protocol-buffers/docs/proto)[ Python API Reference](https://developers.google.com/protocol-buffers/docs/reference/python/index.html)[ Python Generated Code Guide](https://developers.google.com/protocol-buffers/docs/reference/python-generated)[Encoding Reference](https://developers.google.com/protocol-buffers/docs/encoding).

### 为什么要使用Protocol Buffers ？
通过例子"address book"尝试从文件中读取人们的联系信息。例子address book中的每个人（实例）都有属性值name、ID、email地址、phone number.
如何像Protocol Buffers一样序列化或者反序列化结构化数据呢？给出了以下方案：
* 使用Python pickling模块。自从该模块附带在Python语言中之后，它就是几乎是默认的解决方案。但是它的向后兼容性不好，而且不能处理跨语言的数据传输比如C++、JAVA。
* 或者创造一种新的方式：将其编码为字符串比如包含4个数字的字符串"12:3:-23:67"。尽管同时需要编、解码的代码，但是仍然不失为一种简单、灵活的方法。当然仅限于结构简单的数据。
* 序列化数据为XML。XML的优势在于人类可读并且受到许多（开源）语言的支持，特别是当你在多个项目和应用间传输数据时。但是，XML会占用大量空间，进行编码/解码会给应用程序带来巨大的性能损失。此外，XML的DOM树也非常的复杂（较于普通的字段属性）。
Protocol buffers是高效、灵活且自动化的。使用Protocol buffers前，你需要编写.proto文件描述你希望存储的数据的结构。protocol buffer编译器会根据proto文件创建一个类去高效的自动（数据）编码（二进制）、（二进制）解码（数据）。并且该类还给（自己的）每一个属性提供了getter、setter方法用于读取、更新。更为重要的是， protocol buffer 格式数据永远支持向后兼容。


### [实例代码](https://developers.google.com/protocol-buffers/docs/pythontutorial#where-to-find-the-example-code)
实例代码中的examples目录包含了源代码的包。[下载地址](https://developers.google.com/protocol-buffers/docs/downloads.html)

### 定义Protocol 格式


