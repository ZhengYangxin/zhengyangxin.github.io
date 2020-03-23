---
title: protocol buffer 使用和原理
copyright: true
date: 2020-03-01 19:08:10
tags: [数据存储, Protocol Buff, Android]
category: [Android]
img: https://cn.bing.com/th?id=OHR.SeussianLandscape_ZH-CN0785428057_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
summary: Protocol Buffer是什么，在Android中怎么使用，为什么轻便高效的原理
---
### Protocol Buffer 介绍
Protocol Buffer 是一种轻便高效的结构化数据格式，可以用于结构化数据的序列号，适合做数据存储和RPC数据交换格式，他是平台无关，语言无关，可扩展的序列号结构数据格式

#### 结构化数据格式
1. XML, 通过标签定义的
2. JSON，通过键值对定义的
3. DB，数据库

#### 序列化
1. Serilizable
2. Parseable

#### 用途
1. RPC数据交换格式 即网络通讯
2. 数据存储

#### 跨平台，语言
如json，不限操作平台和编程语言同可以使用json

#### 优点
1. 序列化后的体积相比json和xml很小，适合网络传输  40M的json数据  17M Protobuffer
2. 支持跨平台，多语言
3. 消息格式升级和兼容性不错
4. 序列化反序列化快   40M的json数据  10s, Protobuffer 0.8s


### 使用
Android studio 环境配置
#### 配置应用build
```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.0'
        classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.10'
    }
}
```
#### 配置module build.gradle
```
apply plugin: 'com.android.application'
apply plugin: 'com.google.protobuf'
protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.8.0'
    }
    generateProtoTasks {
        all().each { task ->
            task.builtins {
                java {
                    option "lite"
                }
            }
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.google.protobuf:protobuf-javalite:3.9.1'
    ...
}

```

#### 构建消息
消息由至少一个字段组合而成：字段 = 字段修饰符 + 字段类型 + 字段名 + 标识符
标识符TAG：每个字段的唯一标识数字，用于说明二进制文件的对应关系，不能修改，
```
syntax = "proto3";

//option java_package = "www.dcf.com.vo";
package tutorial;

option java_package = "com.zyx.proto";
option java_outer_classname = "TestProto";

message test {
     int32     id = 1;  // ID
     string    str = 2;  // str
     int32     opt = 3;  //optional field
}

```
#### Studio 自动生成代码

通过Android Studio build，Protobuf插件会帮助我们自动生成TestProto类，类结构如下

![](/Users/zhengyangxin/Documents/Blog/zhengyangxin.github.io/source/images/proto_generate_code.png)

Protobuf帮助我们自动生成了testOrBuilder接口，主要定义了个字段的get，set方法，并生成了test类，核心逻辑，通过writeTo（CodedOutputStream）接口序列化到CodedOutputStream，通过parseFrom(InputStream) 接口从InputStream中反序列化

![](/Users/zhengyangxin/Documents/Blog/zhengyangxin.github.io/source/images/protobuffer_struct.png)

### ProtoBuffer 原理

ProtoBuffer不管在时间还是空间上更加高效，是怎么做到的？

消息经过ProtoBuffer序列化后会成为二进制数据流，通过key-Value组成方式写入到二进制数据流。

#### 编码机制

##### Base 128 Varints

是一种可变字节序列化整形的方法

1. 每个byte的最高位是标志位(msb), 如果是1，则表示后面还有byte，否则为结束byte
2. 每个byte的低7位用来存储数值的位
3. Varints方法用Litte-Endian(小端)字节序列

#### 消息结构

##### 编码类型

| Type |     Meaning      |                      Used For                      |
| :--: | :--------------: | :------------------------------------------------: |
|  0   |      Varint      | int32,int64,uinit32,uint64,sint32,sint64,bool,enum |
|  1   |      64-bit      |              fixed64,sfixed64,double               |
|  2   | Length-delimited |           string,bytes,embedded messages           |
|  3   |                  |                                                    |
|  4   |                  |                                                    |
|  5   |      32-bit      |               fixed32,sfixed32,float               |

##### key

key的具体值为   (field_number << 3) | wire_type，

以上面的例子来说，如字段id定义：

```
 int32 id = 2; // 150
```

在序列化时，并不会把字段id写进二进制流中，而是把`field_number=2`通过上述`Key`的定义计算后写进二进制流中，这就是Protobuf可读性差的原因，也是其高效的主要原因。

```
key = (field_number << 3) | wire_type = 2 << 3 | 0 = 10000 |0 = 16  = 0x10
value 150 二进制位1001 0110  
最高位 为msb，将它分为一个一组
1 0010110 进行补齐 0000001 0010110
小端序存储则为0010110  0000001
补齐 表示为msb   10010110 00000001 = 0x96  0x01
则最后的存储为 10 96 01

如果value 300 二进制为 100101100
最高位为msb， 进行7位一组分组
10  0101100  进行补齐0000010 0101100
小端序存储 10101100 00000010  = 0xac 0x2
则最后存储为 10 ac 02
```

key的范围：wire_type只有六种类型，用3bit表示，在一个byte里，去掉mbs，以及3bit的wire_type,只剩下4bit来表示field_number,因此一个Byte里，field_number只能表达0-15，超过15个需要多个byte表示

##### 负数

所谓ZigZag编码即将负数转换成正数，而所有正数都乘2，如0编码成0，-1编码成1，1编码成2，-2编码成3，以此类推，因而它对负数的编码依然保持比较高的效率。

##### 优缺点

* Varint适用于表达比较小的整形，当数字很大时，采用定长编码类型(64bit,32bit)
* 不利于表达负数，负数采用补码表示，会占用更多字节，确定出现负数用sint32,sint64,他会采用ZigZig编码将负数映射成整数，之后再使用Varint编码

Length-delimited

* Length-delimited编码格式会将数据的length也编码进最终的数据，编码格式有string，bytes，自定义消息

* 在消息中将str = "testing", 

  序列化的打印结果 为

  ```
   private void test() {
          TestProto.test.Builder test = TestProto.test.newBuilder();
          TestProto.test test1 = test.setStr("testing").build();
          // 序列化
          byte[] bytes = test1.toByteArray();
  
          for (byte aByte : bytes) {
              System.out.print(aByte +" ");
          }
  }
  // 18 7 116 101 115 116 105 110 103
  str 的field_number = 2; str = "testing"
  key = (field << 3) | wire_type = 2 << 3 | 2 = 10000 | 10 = (十进制) 18 = (16进制)0x12
  7 为 value的长度testing长度为7
  然后计算value的每个字母 t 在ASCII中的 十进制数是 116， e 为101， s为115 以此类推
  发现每个byte存储的是计算后的10进制数字，和网上说16进制 byte存储方式不一致 12 07 74 65 73 74 69 6e 67
  现在按照16进制存储计算
  key = 12 已经确定
  t 的二进制为 0111 0100 16 进制为74
  e 则为65 ..
  所以string的转换方式是 key的16进制 + 字符串长度16进制 + 字符串的每个字母的16进制
  ```

  ##### 结论

  通过原理破解，在通过观察源码Protobuffer的序列化和反序列化同时通过几个位运算实现的，所以他的效率高，体积小

#### 使用指南

* 尽量不要修改tag
* 字段数量不要超过16个，否则会采用2个字节编码, (1个字节最大值为128， key的计算会通过field_number << 3 | wire_type, 当field_number 为16时刚好使用了1个字节计算，否则就需要两个字节计算)
* 如果确定使用负数，采用sint32/sint64

### FastJson对比



### 参考 

[Google Protocol Buffer 的使用和原理](https://www.ibm.com/developerworks/cn/linux/l-cn-gpb/index.html)

