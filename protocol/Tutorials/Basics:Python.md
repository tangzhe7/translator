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
创建address book，并且着手编写.proto文件。
定义.proto文件规则如下：需要序列化的数据都定义为message类型，并且对数据中的每一个属性都定义可被区分的名称、类型。下面是addressbook.proto的示例：
```proto
syntax = "proto2";

package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}


```
注:syntax标明proto的版本
可以看到，proto的语法和C++、java是相似的。接下来逐行解释proto文件中的内容。
.proto文件的起始行是包的申明，用于避免不同项目间的命名冲突。对于Python语言而言，包一般是按照路径定义的，所以在proto文件中定义的包不会对编码造成影响。但是为了谨慎起见（避免命名冲突），在Python中也请按照规则命名。
接下来定义message，message是一系列定义了类型的属性的集合。对于大多数简单结构的数据而言，使用呢bool, int32, float, double, and string等数据类型便已足够。当然可以使用已定义的message作为属性类型，比如在上面的例子中，Person这个message包含了PhoneNumber定义的属性，而Person本身是在AddressBook message中作为一个属性类型。当然这种关系支持嵌套。
例子中PhoneNumber就是被定义在Person中。enum（枚举）类型也是支持的，用于约束属性值必须是已定义的的值中一种，例子中phone number必须是MOBILE, HOME, WORK中的一种。


" = 1", " = 2"是二进制编码过程中的唯一标记"tag"。标记的数字用于优化空间，1-15之间用于不怎么重复的字段，而16或者更大的数字用于重复的字段。重复字段中的每个元素都需要重新编码标签号，因此重复字段是（编码过程中）优化的重点。

每一个属性都必须被以下修饰符修饰：
* required，字段必填，否则message认为是"uninitialized"，在序列化时会产生异常，同样的解析这样的message也会失败。除此之外，required字段和optional字段完全一致。
* optional，可填字段，如果没填会填充默认值。而且可以自定义默认值，就像例子中的phone number。系统的默认值根据类型有所不同：数字类型是0，字符串是空，布尔是false，而对于嵌套的message而言，默认值是字段全是None的message（也叫default instance或者prototype）。如果调用一个没有设置值的必填（选填）字段，那么始终返回默认值。
* repeated，该字段可以是0或者任意长。 repeated字段的顺序将保留在协议缓冲区中。 将repeated字段视为动态大小的数组。

您将找到编写.proto文件的完整指南-包括所有可能的字段类型-[ Protocol Buffer Language Guide](https://developers.google.com/protocol-buffers/docs/proto), protocol buffer不支持继承。

### 编译Protocol Buffers
在写好proto文件后，通过运行protocol buffer编译器（protoc）可以生成可以读写AddressBook (还有 Person 、 PhoneNumber) messages的类：
* 去这个地址下载安装[download the package ](https://developers.google.com/protocol-buffers/docs/downloads.html),记得查看说明。
* 运行编译器，参数有源目录（源代码所在的目录，默认是当前目录）、输出目录（生成文件保存的路径，$SRC_DIR）以及proto文件的路径。
编译命令
```linux
protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/addressbook.proto

```
使用Python语言所以使用--python_out选项，其他语言则用对应的选项。
现在在输出目录生成了 addressbook_pb2.py。

### Protocol Buffer API





