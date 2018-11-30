---
layout:     post
title:      Java使用Protocol Buffers
subtitle:   高效数据压缩格式
date:       2018-11-20
author:     zhanghaichao
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - 数据压缩
---

### 1、介绍
pb是用于将结构化的数据序列化、反序列化。经经常使用于网络传输。
- 效率高
- 生成的是字节码文件
- 与json、xml等类似

### 2、编译器安装
github下载地址（v2.5.0）
https://github.com/protocolbuffers/protobuf/tree/v2.5.0
releases版本i地址（v2.5.0）
https://github.com/protocolbuffers/protobuf/releases/tag/v2.5.0

或者下载源码自行编译
- 源码下载：
wget https://github.com/protocolbuffers/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
- 解压：tar -zxvf v2.5.0.tar.gz
- 进入目录：cd protobuf-2.5.0/
- $ ./configure
- $ make
 - $ make check
 - $ sudo make install
编译完成之后，运行protoc --version,查看版本号，如果能正常打印版本号则说明安装成功了
![image.png](https://cdn.nlark.com/lark/0/2018/png/164731/1542187623655-8c6f231a-ab42-4e76-b116-acacbb2ba6e0.png)

### 3、使用
- 编写.proto 文件
	语法与java类似，参考：https://link.jianshu.com/?t=https://developers.google.com/protocol-buffers/docs/proto
```
package common;

option java_package = "com.alimama.blink.streaming";
option java_outer_classname = "AddressBookProtos";

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

  repeated PhoneNumber phone = 4;
}

message AddressBook {
  repeated Person person = 1;
}
```
- 使用之前已经安装好的编译器程序生成类文件
`protoc --java_out=./ addressbook.proto`
或者
`protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto`
![image.png](https://cdn.nlark.com/lark/0/2018/lspng/164731/1542188747745-504b2a4b-5349-439f-8e5b-3e851f218dc6.png)

- 生成类的使用
假设是maven项目则增加：
```
<dependency>
	<groupId>com.google.protobuf</groupId>
	<artifactId>protobuf-java</artifactId>
	<version>2.5.0</version>
</dependency>
```
1.序列化
![image.png](https://cdn.nlark.com/lark/0/2018/png/164731/1542192461780-4c53a76f-0f97-4d38-91a8-6c1ecf970d4e.png)
2.反序列化
![image.png](https://cdn.nlark.com/lark/0/2018/png/164731/1542192507034-e11aeb85-5d75-4563-a60b-58523b84d409.png)

### 4、一些消息方法
- isInitialized() : 检查是否所有的required字段都已经被设置了。
- toString() : 返回一个人类可读的消息表示，对调试特别有用。
- mergeFrom(Message other): (只有builder可用) 将 other 的内容合并到这个消息中，覆写单数的字段，附接重复的。
- clear(): (只有builder可用) 清空所有的元素为空状态。
- byte[] toByteArray();: 序列化消息并返回一个包含它的原始字节的字节数组。
- static Person parseFrom(byte[] data);: 从给定的字节数组解析一个消息。
- void writeTo(OutputStream output);: 序列化消息并将消息写入 OutputStream。
- static Person parseFrom(InputStream input);: 从一个 InputStream 读取并解析消息。

### 参考资料
- https://developers.google.com/protocol-buffers/docs/downloads
