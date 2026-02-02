---
title: Netty项目实践（一）
date: 2017-03-19 22:14:45
tags: [Netty,JAVA]
category: 后端
---
### 为什么要学
需要要做一个中间件，接受由硬件通过wifi和手机反复传递过来的数据（温湿度），然后再通过中间件将数据写入数据库。由于数据是时时接收的，所以用普通的http请求难以完全实现，所以考虑建立长链接实现功能。</br>
#### Netty的使用场景非常吻合：
* 构建高性能、低时延的各种Java中间件，例如MQ、分布式服务框架、ESB消息总线等，Netty主要作为基础通信框架提供高性能、低时延的通信服务；
* 公有或者私有协议栈的基础通信框架，例如可以基于Netty构建异步、高性能的WebSocket协议栈；
* 各领域应用，例如大数据、游戏等，Netty作为高性能的通信框架用于内部各模块的数据分发、传输和汇总等，实现模块之间高性能通信。
#### 参考:
[通俗地讲，Netty 能做什么？](https://www.zhihu.com/question/24322387)
[Netty那些不得不说的事](http://www.kuqin.com/shuoit/20150709/346994.html);
[Netty系列之Netty高性能之道](http://www.infoq.com/cn/articles/netty-high-performance/)
### 学习过程
1. 创建Maven 项目BabyNetty
```
 <modelVersion>4.0.0</modelVersion>

    <groupId>com.zyx.baby</groupId>
    <artifactId>BabyNetty</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
```
GroupID:是项目组织唯一的标识符，实际对应JAVA的包的结构，是main目录里java的目录结构。
ArtifactID:就是项目的唯一的标识符，实际对应项目的名称，就是项目根目录的名称。

2. 创建两个模块JavaClient，NettyCore
JavaClient:打算用于对服务器的一些数据请求
NettyCore：数据接收，分发处理
3. 依赖jar和插件
```
<!--“打包“这个词听起来比较土，比较正式的说法应该是”构建项目软件包“，具体说就是将项目中的各种文件，
比如源代码、编译生成的字节码、配置文件、文档，按照规范的格式生成归档，最常见的当然就是JAR包和WAR包了，-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>org.origin.netty.Start</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

<dependencies>
        <!--是一个能够将Java bean/map/collection/Java array/xml转换成JSON并且反过来将JSON转换成java对象的类库-->
        <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.4</version>
            <classifier>jdk15</classifier>
        </dependency>
        <!--Netty包-->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>5.0.0.Alpha2</version>
        </dependency>
        <!--Apache的开源项目log4j是一个功能强大的日志组件,提供方便的日志记录-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
</dependencies>
```
4. 配置Log4j日志组件
日志是应用软件中不可缺少的部分，Apache的开源项目log4j是一个功能强大的日志组件,提供方便的日志记录。在apache网站：jakarta.apache.org/log4j 可以免费下载到Log4j最新版本的软件包。
参考：[最详细的Log4j使用教程](http://www.codeceo.com/article/log4j-usage.html)
5. 核心NettyCore编写
    * 常量数据集定义
``` JAVA
package com.zyx.baby.domain;

/**
 * Created by 三金Sir on 2017/3/19.
 */
public class Constant {

    public static final String ENCODING="UTF-8";


    public static final String CLIENT_SERVER="JAVASERVER";
    //not used
    public static final String CLIENT_IOS_SECRET ="BABY_IOS";
    public static final String CLIENT_ANDROID_SECRET ="BABY_ANDROID";
    public static final String CLIENT_HARDWARE="BABY_HARDWARE";

    public enum Type{
        SERVER_CONNECT(1000), //Java client connect
        LOGIN(2000),
        LOGINOUT(2001),
        RECONNECT(1020), // app client reconnect
        SUCCESS_AWARE(4000), // notify client the operation is apply successful.
        INTERNAL_ERROR(4001), // middleware error
        DATA_ERROR(4002); // json data parse error

        public int value;

        Type(int value){
            this.value = value;
        }

        public int value() {
            return value;
        }

        public static Type parse(int value){
            switch (value){
                case 1000:
                    return SERVER_CONNECT;
                case 2000:
                    return LOGIN;
                case 2001:
                    return LOGINOUT;
                case 4000:
                    return SUCCESS_AWARE;
                case 4001:
                    return INTERNAL_ERROR;
                case 1020:
                    return RECONNECT;
                default:
                    return DATA_ERROR;
            }
        }

    }

}

```
知识点 Java 枚举enum
        - 在实际编程中，往往存在着这样的“数据集”，它们的数值在程序中是稳定的，而且“数据集”中的元素是有限的。
        - 例如星期一到星期日七个数据元素组成了一周的“数据集”，春夏秋冬四个数据元素组成了四季的“数据集”。
        - 在Java中如何更好的使用这些“数据集”呢？因此枚举便派上了用场，以下代码详细介绍了枚举的用法。
        #### 参考 ：
    [Java 枚举enum 使用详解](http://blog.csdn.net/zcback1/article/details/51014229?locationNum=4&fps=1).</br>[java enum(枚举)使用详解 + 总结](http://blog.csdn.net/zhushuai1221/article/details/51775811?locationNum=7&fps=1).
* 数据格式定义
``` JAVA
package com.zyx.baby.domain;

import io.netty.buffer.ByteBuf;
import net.sf.json.JSONObject;

import java.nio.charset.Charset;

/**
 * Created by 三金Sir on 2017/3/19.
 */
public class DataPacket {
    private Constant.Type type;
    // SERVER、BLUETOOTH、WIFI
    private String from = "SERVER";
    private String to="";
    private String data = "";

    public DataPacket(Constant.Type type,String data){
        this.type = type;
        this.from = "SERVER";
        this.data = data;
    }

    public DataPacket(String to){
        this.type = Constant.Type.SUCCESS_AWARE;
        this.from = "SERVER";
        this.to = to;
        this.data = "";
    }

    public DataPacket(Constant.Type type,  String to, String data) {
        this.type = type;
        this.to = to;
        this.data = data;
    }

    public DataPacket(Constant.Type type, String from, String to, String data) {
        this.type = type;
        this.from = from;
        this.to = to;
        this.data = data;
    }

    // Getter&Setter begin

    public Constant.Type getType() {
        return type;
    }

    public void setType(Constant.Type type) {
        this.type = type;
    }

    public String getFrom() {
        return from;
    }

    public void setFrom(String from) {
        this.from = from;
    }

    public String getTo() {
        return to;
    }

    public void setTo(String to) {
        this.to = to;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }

    // Getter&Setter end

    public static DataPacket parse(Object object){
        ByteBuf receive = (ByteBuf)object;
        JSONObject json=JSONObject.fromObject(receive.toString(Charset.forName(Constant.ENCODING)));
        receive.release();
        return new DataPacket(Constant.Type.parse(json.getInt("type")),json.getString("from"),json.getString("to"),json.getString("data"));
    }

    @Override
    public String toString() {
        return "{" +
                "\"type\":" + type.value() +
                ", \"from\":\"" + from + '"' +
                ", \"to\":\"" + to + '"' +
                ", \"data\":\"" + data + '"' +
                '}';
    }
}
```
知识点：netty中ByteBuf部分
[netty中ByteBuf部分的分析](http://www.tuicool.com/articles/FFb6Zr)
[Netty之ByteBuf](http://blog.csdn.net/alex_bean/article/details/51251015?locationNum=1&fps=1)
[ Netty中的ByteBuf原理分析]http://blog.csdn.net/u012832964/article/details/50899511
* 配置拦截器，连接设备身份认证
    过程：
        - verify = 设备标识字段+时间戳
        - MD5加密后与客户端发送code匹配
        - 成功,返回标识字段;否则null,Access denied
```JAVA
package com.zyx.baby.interceptor;


import com.zyx.baby.domain.Constant;
import com.zyx.baby.domain.DataPacket;
import com.zyx.baby.utils.Util;
import io.netty.channel.ChannelHandlerContext;

import static org.apache.log4j.Logger.*;

/**
 * 连接设备拦截器
 * Created by 三金Sir on 2017/3/19.
 */
public class Interceptor {
    private static final org.apache.log4j.Logger log = getLogger(Interceptor.class);

    public static String authIntercept(DataPacket pkt, ChannelHandlerContext ctx){
        String secret = Constant.CLIENT_SERVER;
        try {
            String[] data = pkt.getData().split("&");
            String code = data[0];
            String timestamp = data[1];

            String verify;
            if(pkt.getType() == Constant.Type.SERVER_CONNECT)
                verify = Constant.CLIENT_SERVER + timestamp;
            else
                verify = Constant.CLIENT_HARDWARE + timestamp;

            String validateCode = Util.md5(verify);

            boolean isLogin = pkt.getType() == Constant.Type.LOGIN || pkt.getType() == Constant.Type.RECONNECT;

            if(isLogin && !code.equals(validateCode)){
                verify = Constant.CLIENT_HARDWARE + timestamp;
                validateCode = Util.md5(verify);
                secret = Constant.CLIENT_HARDWARE;
            }

            if(isLogin && !code.equals(validateCode)){
                verify = Constant.CLIENT_ANDROID_SECRET + timestamp;
                validateCode = Util.md5(verify);
                secret = Constant.CLIENT_ANDROID_SECRET;
            }

            if(isLogin && !code.equals(validateCode)){
                verify = Constant.CLIENT_IOS_SECRET + timestamp;
                validateCode = Util.md5(verify);
                secret = Constant.CLIENT_IOS_SECRET;
            }

            if (!code.equals(validateCode)) {
//                SystemService.sendMessage(new DataPacket(Constant.Type.DATA_ERROR, "Access denied."), ctx);
                ctx.close();
                log.warn("[ socket interceptor ] Access denied.");
                return null;
            }

        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }

        return secret;
    }

}
```
未完待续~~~

