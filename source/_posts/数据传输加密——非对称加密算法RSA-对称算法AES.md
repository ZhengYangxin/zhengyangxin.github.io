---
title: 数据传输加密——非对称加密算法RSA+对称算法AES
date: 2017-04-29 23:52:12
tags: [安全、Android]
category: Android
---
### 数据传输加密
&emsp;&emsp;在开发应用过程中，客户端与服务端经常需要进行数据传输，涉及到重要隐私信息时，开发者自然会想到对其进行加密，即使传输过程中被“有心人”截取，也不会将信息泄露。对于加密算法，相信不少开发者也有所耳闻，比如MD5加密，Base64加密，DES加密，AES加密，RSA加密等等。在这里我主要向大家介绍一下我在开发过程中使用到的加密算法，RSA加密算法+AES加密算法。简单地介绍一下这两种算法吧。

### RSA

&emsp;&emsp;之所以叫RSA算法，是因为算法的三位发明者RSA是目前最有影响力的公钥加密算法，它能够抵抗到目前为止已知的绝大多数密码攻击，已被ISO推荐为公钥数据加密标准，主要的算法原理就不多加介绍，如果对此感兴趣的话，建议去百度一下RSA算法。需要了解的是RSA算法属于非对称加密算法，非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。简单的说是“公钥加密，私钥解密；私钥加密，公钥解密”。

### AES

 &emsp;&emsp;高级加密标准（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。经过五年的甄选流程，高级加密标准由美国国家标准与技术研究院（NIST）于2001年11月26日发布于FIPS PUB 197，并在2002年5月26日成为有效的标准。2006年，高级加密标准已然成为对称密钥加密中最流行的算法之一。

### 为什么要结合使用这两种算法
&emsp;&emsp;如果不清楚非对称算法和对称算法，也许你会问，为什么要结合使用这两种算法，单纯使用一种算法不行吗？这就要结合不同的场景和需求了。

&emsp;&emsp;客户端传输重要信息给服务端，服务端返回的信息不需加密的情况
&emsp;&emsp;客户端传输重要信息给服务端，服务端返回的信息不需加密，例如绑定银行卡的时候，需要传递用户的银行卡号，手机号等重要信息，客户端这边就需要对这些重要信息进行加密，使用RSA公钥加密，服务端使用RSA解密，然后返回一些普通信息，比如状态码code,提示信息msg,提示操作是成功还是失败。这种场景下，仅仅使用RSA加密是可以的。

&emsp;&emsp;客户端传输重要信息给服务端，服务端返回的信息需加密的情况
&emsp;&emsp;客户端传输重要信息给服务端，服务端返回的信息需加密,例如客户端登录的时候，传递用户名和密码等资料，需要进行加密，服务端验证登录信息后，返回令牌token需要进行加密，客户端解密后保存。此时就需要结合这两种算法了。至于整个流程是怎样的，在下面会慢慢通过例子向你介绍，因为如果一开始就这么多文字类的操作，可能会让读者感到一头雾水。

### 使用RSA加密和解密
产生公钥和私钥：产生RSA公钥和密钥的方法有很多，在这里我直接使用我封装好的方法产生，都最后我会将两个算法的工具类赠送给大家。

````
/**
 * 生成公钥和私钥
 * 
 * @throws Exception
 * 
 */
public static void getKeys() throws Exception {
    KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
    keyPairGen.initialize(1024);
    KeyPair keyPair = keyPairGen.generateKeyPair();
    RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
    RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();

    String publicKeyStr = getPublicKeyStr(publicKey);
    String privateKeyStr = getPrivateKeyStr(privateKey);

    System.out.println("公钥\r\n" + publicKeyStr);
    System.out.println("私钥\r\n" + privateKeyStr);
}

public static String getPrivateKeyStr(PrivateKey privateKey)
        throws Exception {
    return new String(Base64Utils.encode(privateKey.getEncoded()));
}

public static String getPublicKeyStr(PublicKey publicKey) throws Exception {
    return new String(Base64Utils.encode(publicKey.getEncoded()));
}
````
公匙
````
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCRQZ5O/AOAjeYAaSFf6Rjhqovws78I716I9oGF7WxCIPmcaUa1YuyLOncCCuPsaw69+RMWjdbOBp8hd4PPM/d4mKTOVEYUE0SfxhhDTZaM5CzQEUXUyXy7icQTGR5wBjrbjU1yHCKOf5PJJZZQWB06husSFZ40TdL7FdlBpZ1u1QIDAQAB
````
私钥
````
MIICdgIBADANBgkqhkiG9w0BAQEFAASCAmAwggJcAgEAAoGBAJFBnk78A4CN5gBpIV/pGOGqi/CzvwjvXoj2gYXtbEIg+ZxpRrVi7Is6dwIK4+xrDr35ExaN1s4GnyF3g88z93iYpM5URhQTRJ/GGENNlozkLNARRdTJfLuJxBMZHnAGOtuNTXIcIo5/k8klllBYHTqG6xIVnjRN0vsV2UGlnW7VAgMBAAECgYBMoT9xD8aRNUrXgJ7YyFIWCzEUZN8tSYqn2tPt4ZkxMdA9UdS5sFx1/vv1meUwPjJiylnlliJyQlAFCdYBo7qzmib8+3Q8EU3MDP9bNlpxxC1go57/q/TbaymWyOk3pK2VXaX+8vQmllgRZMQRi2JFBHVoep1f1x7lSsf2TpipgQJBANJlO+UDmync9X/1YdrVaDOi4o7g3w9u1eVq9B01+WklAP3bvxIoBRI97HlDPKHx+CZXeODx1xj0xPOK3HUz5FECQQCwvdagPPtWHhHx0boPF/s4ZrTUIH04afuePUuwKTQQRijnl0eb2idBe0z2VAH1utPps/p4SpuT3HI3PJJ8MlVFAkAFypuXdj3zLQ3k89A5wd4Ybcdmv3HkbtyccBFALJgs+MPKOR5NVaSuF95GiD9HBe4awBWnu4B8Q2CYg54F6+PBAkBKNgvukGyARnQGc6eKOumTTxzSjSnHDElIsjgbqdFgm/UE+TJqMHmXNyyjqbaA9YeRc67R35HfzgpvQxHG8GN5AkEAxSKOlfACUCQ/CZJovETMmaUDas463hbrUznp71uRMk8RP7DY/lBnGGMeUeeZLIVK5X2Ngcp9nJQSKWCGtpnfLQ==
````

&emsp;&emsp;很明显，公钥字符串长度比较短，私钥的比较长。生成完密钥后，公钥可以存放在客户端，即使被别人知道公钥，也是没有问题的；私钥则一定要保存在服务端。如果到时公司面临人事变动，避免私钥被离职人员泄露，可以重新生成公钥和密钥。

使用公钥加密，私钥解密
![Alt text](http://i.imgur.com/EzSYTst.png "Optional title")
这里在客户端模拟加密的情况，对字符串”Beyond黄家驹”使用RSA加密，调用RSAUtils的encryptByPublicKey()方法，输出结果为：
````
密文: BRFjf3tUqRqlwuP5JtzxZinf7lp+AHuHM9JSabM5BNFDxuUe9+uuO6RpCHVH5PibifqQHzGNsyZn1G9QcIENT9Tbm+PZwAbNUlMPZRDBU1FSnOtY8dBdeW/lJdnY9sJVwNvIBnOLQk66hxRh6R2149dwlgdsGUpWMOMBzcP3vsU=
````
在服务端，可以使用RSAUtils的decryptByPrivateKey()方法进行解密，现在模拟服务端解密
![Alt text](http://i.imgur.com/oNox5Ma.png "Optional title")
&emsp;&emsp;在这里虽然没有完全模拟数据传输过程，比如说客户端发起一个网络请求，传递参数给服务端，服务端接收参数并进行处理，也是为了让大家可以更加容易明白，所以这里只是进行简单的模拟。可以看到Android客户端端和Java服务端的RSA加密解密算法是可以互通的，原因是他们所使用到的base64加密类是一致的，所以才可以实现加密和解密的算法互通。
![Alt text](http://i.imgur.com/lLgQbab.png "Optional title")
![Alt text](http://i.imgur.com/lLgQbab.png "Optional title")
&emsp;&emsp;使用到的jar包都是javabase64-1.3.1.jar,相信不少人都知道，java中有自带的Base64算法类，但是安卓中却没有，之前出现的情况是，使用的Base64类不统一，比如在安卓客户端开发使用的Base64算法是使用第三方提供的jar包，而java服务端中使用的是JDK自带的Base64,导致从客户端传过来的密文，服务端解析出错。

&emsp;&emsp;上面的例子展示了客户端使用公钥加密，服务端使用私钥解密的过程。也许你会这么想，既然可以如此，那服务端那边信息也可以通过RSA加密后，传递加密信息过来，客户端进行解密。但是，这样做，显示是不安全的。原因是，由于客户端并没有保存私钥，只有公钥，只可以服务端进行私钥加密，客户端进行公钥解密，但由于公钥是公开，别人也可以获取到公钥，如果信息被他们截取，他们同样可以通过公钥进行解密，那么这样子加密，就毫无意义了，所以这个时候，就要结合对称算法，实现客户端与服务端之前的安全通信了。

使用AES加密解密
加密
![Alt text](http://i.imgur.com/WgX5ss7.png "Optional title")
模拟客户端进行AES加密，我们通过调用AESUtils中的generateKey()方法，随机产生一个密钥，用于对数据进行加密。输出的结果为：
````
密钥: 6446c69c0f914a57
密文: GECDQOsc22yV48hdJENTMg==
````

解密
&emsp;&emsp;模拟服务端进行AES解密，由于AES属于对称算法，加密和解密需要使用同一把密钥，所以，服务端要解密传递过来的内容，就需要密钥 + 密文。这里模拟一下服务端解密。
![Alt text](http://i.imgur.com/NXLPGwG.png "Optional title")
&emsp;&emsp;到这里也许你会问，客户端使用AES进行加密，服务端要进行解密的话，需要用到产生的密钥，那密钥必须从客户端传输到服务端，如果不对密钥进行加密，那加密就没有意义了。所以这里终于谈到了重点，RSA算法+AES算法结合使用。

RSA算法+AES算法的使用
&emsp;&emsp;举一个简单的例子来说明一下吧，例如实名认证功能，需要传递用户真实姓名和身份证号，对于这种重要信息，需要进行加密处理。

客户端使用RSA + AES对重要信息进行加密
客户端加密过程主要分为以下三个步骤：

1. 客户端随机产生AES的密钥；

2. 对身份证信息（重要信息）进行AES加密；

3. 通过使用RSA对AES密钥进行公钥加密。
![Alt text](http://i.imgur.com/8c2YKWN.png "Optional title")

&emsp;&emsp;这样在传输的过程中，即时加密后的AES密钥被别人截取，对其也无济于事，因为他并不知道RSA的私钥，无法解密得到原本的AES密钥，就无法解密用AES加密后的重要信息。

服务端使用RSA + AES对重要信息进行解密

服务端解密过程主要分为以下两个步骤：

1. 对加密后的AES密钥进行RSA私钥解密，拿到密钥原文；

2. 对加密后的重要信息进行AES解密，拿到原始内容。
![Alt text](http://i.imgur.com/kVrEvsu.png "Optional title")

&emsp;&emsp;现实开发中，服务端有时也需要向客户端传递重要信息，比如登录的时候，返回token给客户端，作为令牌，这个令牌就需要进行加密，原理也是差不多的，比上面多一个步骤而已，就是将解密后的AES密钥，对将要传递给客户端的数据token进行AES加密，返回给客户端，由于客户端和服务端都已经拿到同一把AES钥匙，所以客户端可以解密服务端返回的加密后的数据。如果客户端想要将令牌进行保存，则需要使用自己定义的默认的AES密钥进行加密后保存，需要使用的时候传入默认密钥和密文，解密后得到原token。

&emsp;&emsp;上面提及到客户端加密，服务端返回数据不加密的情况，上面说到仅仅使用RSA是可以，但是还是建议同时使用这两种算法，即产生一个AES密钥，使用RSA对该密钥进行公钥加密，对重要信息进行AES加密，服务端通过RSA私钥解密拿到AES密钥，再对加密后的重要信息进行解密。如果仅仅使用RSA，服务端只通过RSA解密，这样会对于性能会有所影响，原因是RSA的解密耗时约等于AES解密耗时的100倍，所以如果每个重要信息都只通过RSA加密和解密，则会影响服务端系统的性能，所以建议两种算法一起使用。

同时还有相应的JS版RSA和AES算法，使用方式也差不多，在这里简单演示一下：

````
<!DOCTYPE html>
<html>
  <head>
    <title>RSA+AES.html</title>

    <meta name="keywords" content="keyword1,keyword2,keyword3">
    <meta name="description" content="this is my page">
    <meta name="content-type" content="text/html; charset=UTF-8">
    <script type="text/javascript" src="./js/rsa.js"></script>
    <script type="text/javascript" src="./js/aes.js"></script>
    <script type="text/javascript">
        var key = getKey();//随机产生AES密钥
        var encryptKey = RSA(key);//对AES密钥进行RSA加密
        console.log("encryptKey: " + encryptKey);

        //测试AES加密和解密
        var cipherText = AESEnc(key,"123456");
        var plainText = AESDec(key,cipherText);
        console.log("密文: " + cipherText);
        console.log("明文: " + plainText);
    </script>
  </head>

  <body>
    This is my HTML page. <br>
  </body>
</html>
````
打开页面后，查看控制台输出：
![Alt text](http://i.imgur.com/PyU0b3r.png "Optional title")
同时，模拟服务端解密，运行结果如下：
![Alt text](http://i.imgur.com/3VrRv3d.png "Optional title")

需要注意的是:

1.RSAUtils中配置公钥和密钥，可以使用getKeys()方法产生。如果是客户端，则无须配置私钥，把没有私钥的RSAUtils放到客户端，因为仅需要用到公钥加密的方法。

2.AESUtils中配置偏移量IV_STRING；

3.rsa.js中最底部配置公钥，须和上面RSAUtils配置的公钥一致；

4.aes.js中的底部var iv = CryptoJS.enc.Utf8.parse(“16-Bytes–String”); //加密向量中，替换里面的字符串，加密向量须和 
是上面的AESUtils中的偏移量一致。

各种语言的加密的处理方式有所差异，所以我们需要因地制宜。了解此加密的思想方法即可
1. [php 和 java RSA 对称加密互通的问题](http://blog.csdn.net/treesky/article/details/49422645)
2. [php与java通用AES加密解密算法](http://www.cnblogs.com/yipu/articles/3871576.html)
3. [Android错误解决：java.lang.NoSuchMethodError: No static method encodeBase64String](http://blog.csdn.net/pbm863521/article/details/54023009)
