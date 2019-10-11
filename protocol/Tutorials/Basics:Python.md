## Protocol Buffer Basics: Python
[原文](https://developers.google.com/protocol-buffers/docs/pythontutorial)

译者：schopenhauerzhang（schopenhauerzhang@gmail.com）

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
Python protocol buffer 编译器并不会像C++、JAVA（编译器）那样直接生成可调用代码，相反生成的是针对所有的messages, enums, 字段, 和一些类的描述。看一个message的例子：

```
class Person(message.Message):
  __metaclass__ = reflection.GeneratedProtocolMessageType

  class PhoneNumber(message.Message):
    __metaclass__ = reflection.GeneratedProtocolMessageType
    DESCRIPTOR = _PERSON_PHONENUMBER
  DESCRIPTOR = _PERSON

class AddressBook(message.Message):
  __metaclass__ = reflection.GeneratedProtocolMessageType
  DESCRIPTOR = _ADDRESSBOOK
```
每个类中最重要的描述是 __metaclass__ = reflection.GeneratedProtocolMessageType。每一个metaclass就像是创建类的模板。在加载过程中，GeneratedProtocolMessageType会通过(proto文件中的）描述创建一系列需要用到的message的方法，并将这些方法和对应的类关联起来，以便于接下来的使用。
以上工作的最终结果是你可以使用Person类的字段就像使用messages基类一样。比如：
 ```
import addressbook_pb2
person = addressbook_pb2.Person()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"
phone = person.phones.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.PhoneType.HOME
```
注意这些修改并不是轻易的在Python 对象中添加字段。如果使用了在proto文件中没有定义的字段就会抛出AttributeError异常。如果设置字段的值不符合proto文件中的定义则会抛出TypeError异常。所以建议给需要读取的字段设置默认值。
```
person.no_such_field = 1  # raises AttributeError
person.id = "1234"        # raises TypeError
```
如果需要深入了解protocol 编译器相关的东西比如一些特殊字段的定义请移步[Python generate code reference](https://developers.google.com/protocol-buffers/docs/reference/python-generated)

#### Enums
枚举Enums是用于符号常量的数字类型值的限定，比如常量addressbook_pb2.Person.PhoneType.WORK 的值是2。

#### Standard Message Methods
每个message class 都包含许多方法使得可以检查、使用message：
* IsInitialized（）检查是否所有的必填字段都填充了；
* __str__（）返回人类可读的message描述，适用于debug，常见触发方式是str(message) 和 print message；
* CopyFrom（other_msg）用其他的message更新现有message的值；
* Clear（）重置所有的字段为空

以上方法实现于Message接口。更多信息请查看 [complete API documentation for Message](https://developers.google.com/protocol-buffers/docs/reference/python/google.protobuf.message.Message-class).

#### Parsing and Serialization
每一个protocol buffer 都有方法从messages的二进制格式读写数据：
* SerializeToString()序列化message为二进制数据，但是返回的是（包含二进制数据的）字符串
* ParseFromString()从（包含二进制数据的）字符串中解析出message

这里只列出了一对解析和编码的方法，完整列表请看这里[Message API reference](https://developers.google.com/protocol-buffers/docs/reference/python/google.protobuf.message.Message-class).

### 写入Message
现在可以使用protocol buffer类了。首先保证address book能够写入数据到address book文件，接着创建address book的protocol buffer实例，并将其写入输出流中。
下面是一段从AddressBook文件读取数据的代码，根据用户输入创建一个新的Person实例，再将AddressBook写回文件。高亮代码是protocol buffer编译器的调用和说明。

```
#! /usr/bin/python

import addressbook_pb2
import sys

# This function fills in a Person message based on user input.
def PromptForAddress(person):
  person.id = int(raw_input("Enter person ID number: "))
  person.name = raw_input("Enter name: ")

  email = raw_input("Enter email address (blank for none): ")
  if email != "":
    person.email = email

  while True:
    number = raw_input("Enter a phone number (or leave blank to finish): ")
    if number == "":
      break

    phone_number = person.phones.add()
    phone_number.number = number

    type = raw_input("Is this a mobile, home, or work phone? ")
    if type == "mobile":
      phone_number.type = addressbook_pb2.Person.PhoneType.MOBILE
    elif type == "home":
      phone_number.type = addressbook_pb2.Person.PhoneType.HOME
    elif type == "work":
      phone_number.type = addressbook_pb2.Person.PhoneType.WORK
    else:
      print "Unknown phone type; leaving as default value."

# Main procedure:  Reads the entire address book from a file,
#   adds one person based on user input, then writes it back out to the same
#   file.
if len(sys.argv) != 2:
  print "Usage:", sys.argv[0], "ADDRESS_BOOK_FILE"
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
try:
  f = open(sys.argv[1], "rb")
  address_book.ParseFromString(f.read())
  f.close()
except IOError:
  print sys.argv[1] + ": Could not open file.  Creating a new one."

# Add an address.
PromptForAddress(address_book.people.add())

# Write the new address book back to disk.
f = open(sys.argv[1], "wb")
f.write(address_book.SerializeToString())
f.close()
```

### 读取 Message
当然，如果无法从中获取任何信息，address book也没用。本示例读取由以上示例创建的文件，并打印其中的所有信息。
```
#! /usr/bin/python

import addressbook_pb2
import sys

# Iterates though all people in the AddressBook and prints info about them.
def ListPeople(address_book):
  for person in address_book.people:
    print "Person ID:", person.id
    print "  Name:", person.name
    if person.HasField('email'):
      print "  E-mail address:", person.email

    for phone_number in person.phones:
      if phone_number.type == addressbook_pb2.Person.PhoneType.MOBILE:
        print "  Mobile phone #: ",
      elif phone_number.type == addressbook_pb2.Person.PhoneType.HOME:
        print "  Home phone #: ",
      elif phone_number.type == addressbook_pb2.Person.PhoneType.WORK:
        print "  Work phone #: ",
      print phone_number.number

# Main procedure:  Reads the entire address book from a file and prints all
#   the information inside.
if len(sys.argv) != 2:
  print "Usage:", sys.argv[0], "ADDRESS_BOOK_FILE"
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
f = open(sys.argv[1], "rb")
address_book.ParseFromString(f.read())
f.close()

ListPeople(address_book)
```
### 扩展 Protocol Buffer
不久之后可能需要改善此前protocol buffer的（数据）定义，并且需要新的定义文件向后兼容，此前的定义文件向前兼容，要达到以上要求，所以新版本的protocol buffer定义需要遵守以下规则：
* 不能改变已有的标记数字tag numbers 
* 不能添加删除必填字段（required ）
* 可以删除optional 或者repeated 字段.
* 如果添加了新的optional 或者repeated 字段，必须使用新的标记数字（tag number）（可以是全新的，也可以是以前使用过但是已删除）

[异常](https://developers.google.com/protocol-buffers/docs/proto.html#updating)

只要严格遵守规则，旧版本可以完美先前兼容，遇到新的字段会自动忽略。旧版本中的optional字段被删除后会用默认值填充，repeated字段填充为空。自然新版本也可以读老版本的messages。但还是需要检查可选字段是否被设置了值或者在proto文件中有默认值，默认值的设置方式是在tag后面添加 *[default = value]* 。正如前面提到的，如果字段的默认值没有设置，那么会用类型的默认值填充，比如字符串的默认值是空，布尔的默认值是false，数字的默认值是0。而对于新增的repeated字段如果没有设置has_ 标记，那么就无法知道它是没有（被新版本）赋值还是根本没有（被旧版本设置）值。

### 进阶用法
Protocol buffers不知用于访问和序列化，还有其他用处[ Python API reference](https://developers.google.com/protocol-buffers/docs/reference/python/index.html)。

此外protocol message 还提供反射类（reflection）。通过它，可以遍历消息的字段并修改它们的值，而无需针对任何特定的消息类型编写代码。使用反射的一种非常有用的场景是将protocol messages与其他编码进行相互转换比如 XML 、 JSON。
反射的一种更高级的用法可能是查找同一类型的两个消息之间的差异，或者开发一种“protocol messages的正则表达式”，在其中可以编写与某些消息内容匹配的表达式。 只有你想不到，没有你通过Protocol Buffers做不到的！

反射类的文档[Reflection](https://developers.google.com/protocol-buffers/docs/reference/python/google.protobuf.message.Message-class)



