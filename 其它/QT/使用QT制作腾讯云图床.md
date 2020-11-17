# 使用QT做腾讯云图床

## 什么是图床

 浏览一个博客时，所看到的图片都是一个链接，而不是真正的一张图片放在博客文章上。
要想获取一张图片的链接，须得将自己的本地图片上传到远端服务器上来获得。
利用腾讯云，可以将自己的本地图片上传到其对象存储，获得图片的访问链接，就可以远程访问图片啦！
而存放你的图片的地方就叫做“图床”。

## 上传方法

将图片上传到腾讯云图床可以有两种方法。
一种是在腾讯云的控制台中直接上传，这种操作优点是简单，缺点是必须每次上传都得登录腾讯云，浪费时间。
二是本机上传，将图片直接上传，这种操作优点是更加简单快捷方便，只需选择本地图片就可以上传。虽然目前有很多软件专门做这一些东西。

## 个人制作上传工具

### 上传工具简单介绍
这里我使用QT制作一个上传工具。
功能是输入上传图片路径，按下上传按钮，即可上传，前提是先得进行基础配置。

![QTui1](https://vnote-1259141668.cos.ap-guangzhou.myqcloud.com/QTui1.PNG)
                                            上传界面

![QTui2](https://vnote-1259141668.cos.ap-guangzhou.myqcloud.com/QTui2.PNG)
                                                配置界面

虽然功能简单，但对于是小白的我来说，实现这么一个小工具，也是大费周章。

## 实现原理

做界面是很简单的，关键技术是如何将本地文件上传服务器。
克服这技术难度花了我很多时间与精力，甚至中途都有想过放弃，还好师兄帮助了我。

### 关键技术

#### 第一步：注册腾讯云
腾讯云给我们提供了一个叫做对象存储的东西，哪里存放我们上传的图片。
首先第一步是注册腾讯云（很简单，上官网操作）
注册之后，在腾讯云对象存储控制台中开通腾讯云对象存储（COS）服务。
然后创建一个bucket(桶），属性选择公有度私有写，其他的不管（我也不懂）
在桶里面就可以上传文件。

#### 第二部：创建密钥

腾讯云对发起HTTP请求的用户会进行身份认定。
匿名请求：求不携带任何身份HTTP 请标识和鉴权信息，通过 RESTful API 进行 HTTP 请求操作。
签名请求：HTTP 请求时携带签名，COS 服务器端收到消息后，进行身份验证，验证成功则可接受并执行请求，否则将会返回错误信息并丢弃此请求。
如果桶的属性是公有度公有写的话，可以发起匿名请求（简单），但是这样你的桶就不是你的了。（肯定不行的）
我们需要的是签名请求。
要想发起签名请求，得要获得做基本的身份信息，这就得创建密钥了。
在API密钥管理界面创建就行（简单）
密钥有两部分
SecretId: 身份ID
SecretKey: 密码

#### 第三部：创造请求（最重要）

由于我们只是简单的上传图片，所以我们选择对象存储中的Put Object接口就行。

官网的请求格式如下：

```
PUT /<ObjectKey> HTTP/1.1
Host: <BucketName-APPID>.cos.<Region>.myqcloud.com
Date: GMT Date
Content-Type: Content Type
Content-Length: Content Length
Content-MD5: MD5
Authorization: Auth String

[Object Content]
```
看不懂是吧，没关系，这是Http请求格式，跟腾讯云无关。
我简单说明一下
第一行那个叫——请求行。说明了请求方式（put请求），请求路径（<objectKey>)，可以理解为图片名。 版本（HTTP/1.1）
第二到第七行叫——请求头。 说明了请求中的请求地址（Host), 时间（Data)，内容格式（Content-type), 请求内容大小（Content-Length),
                                             请求内容格式（Content-MD5），请求签名（Authorization)。
最后一行是请求内容。

其中最最重要的是请求签名（Authorization)。
它包含了用户的信息，通过某种途径将SecretId，IDSecretKey等信息结合起来的一条信息。
腾讯云通过它来识别发起Http签名请求的用户信息。
其它请求头都很简单。

案例：
```
PUT /exampleobject HTTP/1.1
Host: examplebucket-1250000000.cos.ap-beijing.myqcloud.com
Date: Fri, 21 Jun 2019 09:24:28 GMT
Content-Type: image/jpeg
Content-Length: 13
Content-MD5: ti4QvKtVqIJAvZxDbP/c+Q==
Authorization: q-sign-algorithm=sha1&q-ak=AKID8A0fBVtYFrNm02oY1g1JQQF0c3JO****&q-sign-time=1561109068;1561116268&q-key-time=1561109068;1561116268&q-header-list=content-length;content-md5;content-type;date;host&q-url-param-list=&q-signature=998bfc8836fc205d09e455c14e3d7e623bd2****
Connection: close
```
注：****是省列的内容。
利用QT中的QNetworkAccessManager、QNetWorkRequest、QNetWorkReply可以发起put请求。（自行了解，在此不做介绍）
通过QNetWorkRequest的seHeader可以设置一部分请求头：Content-Type、Content-Length。
```
    request.setHeader(QNetworkRequest::ContentTypeHeader, "image/jpeg");
    request.setHeader(QNetworkRequest::ContentLengthHeader, fdata.size());
```
其中"image/jpeg"是图片的请求类型。
fdata是QByteArray类型的变量，内容是图片。

而Authorization可以通过QNetWorkRequest的seRawHeader自定义。
```
    request.setRawHeader(QByteArray("Authorization"), str.toUtf8());
```
str是QString型变量，内容是Authorization。
其它请求头可以选择忽视。

接下来是最重要的生成签名部分：生成Authorization。
官方文档中有介绍如何生成请求签名

##### 准备工作
1.  APPID、SecretId 和 SecretKey。
    在访问管理控制台的 **API 密钥管理** 页面中获取。
2.  获得时间（转到秒）
     通过QT提供的API，可以获得时间
```
    QDateTime::currentDateTime().toTime_t();
```
3. 确定对应的 HMAC-SHA1、SHA1 函数。
    这两个函数很重要，它们是一种网络请求加密函数（自行了解），这里需要它们将信息加密生成Authorization。
    每种语言都有他的
    SHA1函数QT有提供API,如下：
```
    QCryptographicHash::hash(str, QCryptographicHash::Sha1).toHex(); //str是QByteArray型变量，返回将str加密后的结果。
```
   注：因为结果要以16进制的形式显示，所以转到16进制。
   而HMAC-SHA1本质也是一种SHA1，只不过他要经过多次SHA1加密。QT不提供相应的API，所以只能从网上找，
```
   QByteArray hmacSha1(QByteArray key, QByteArray baseString)
    {
        int blockSize = 64;
        if (key.length() > blockSize) {
            key = QCryptographicHash::hash(key, QCryptographicHash::Sha1);
        }
        QByteArray innerPadding(blockSize, char(0x36));
        QByteArray outerPadding(blockSize, char(0x5c));
        for (int i = 0; i < key.length(); i++) {
            innerPadding[i] = innerPadding[i] ^ key.at(i);
            outerPadding[i] = outerPadding[i] ^ key.at(i);
        }
        QByteArray total = outerPadding;
        QByteArray part = innerPadding;
        part.append(baseString);
        total.append(QCryptographicHash::hash(part, QCryptographicHash::Sha1));
        QByteArray hashed = QCryptographicHash::hash(total, QCryptographicHash::Sha1);
        return hashed.toHex();
    }
```
   注：这原本是返回Base64形式的，我当时也是搞得一脸懵逼，后来经师兄的稍微修改就行了。

##### 签名流程
1. 生成 KeyTime：
    生成例子：1557902800;1557910000。 
2. 生成 SignKey：
    使用 HMAC-SHA1 以 SecretKey 为密钥，以 KeyTime 为消息，计算消息摘要（哈希值），即为 SignKey。
    生成例子：36bcd76dbb8c9f066472fec403df8a34cab34c77
    这里调用如下API来获取
```
    hmacSha1(SecretKey , KeyTime)；
```
3. 生成 UrlParamList 和 HttpParameters：
    1. 遍历 HTTP 请求参数，生成 key 到 value 的映射 Map 及 key 的列表 KeyList，其中 key 转换为小写形式，value 使用 UrlEncode 编码，没有 value 的参数，则认为 value 为    空             字符串。例如请求路径为/?acl，则认为是/?acl=。
    2. 将 KeyList 按照字典序排序。
    3. 将 Map 和 KeyList 中的 key 使用 UrlEncode 编码，并再次转换为小写形式。
    4. 按照 KeyList 的顺序拼接 Map 中的每一个键值对，格式为key1=value1&key2=value2&key3=value3，即为 HttpParameters。
    5. 按照 KeyList 的顺序拼接 KeyList 中的每一项，格式为key1;key2;key3，即为 UrlParamList。
     注：这里只是简单上传图片，无需管这个。UrlParamList为空， HttpParameters为图片名。
4. 生成 HeaderList 和 HttpHeaders
    1. 遍历 HTTP 请求头部，生成 key 到 value 的映射 Map 及 key 的列表 KeyList，其中 key 转换为小写形式，value 使用 UrlEncode 编码。
    2. 将 KeyList 按照字典序排序。
    3. 将 Map 和 KeyList 中的 key 使用 UrlEncode 编码，并再次转换为小写形式。
    4. 按照 KeyList 的顺序拼接 Map 中的每一个键值对，格式为key1=value1&key2=value2&key3=value3，即为 HttpHeaders。
    5. 按照 KeyList 的顺序拼接 KeyList 中的每一项，格式为key1;key2;key3，即为 HeaderList。
    注：这里也无需管这个。
5. 生成 HttpString
    根据 HTTP 方法、HTTP 请求路径、HttpParameters 和 HttpHeaders 生成 HttpString，格式为HttpMethod\nUriPathname\nHttpParameters\nHttpHeaders\n。
    其中：
     HttpMethod 转换为小写，例如 get 或 put。
    UriPathname 为请求路径，例如/或/exampleobject（/图片名）。
    \n为换行符。如果其中有字符串为空，前后的换行符需要保留，例如get\n/exampleobject\n\n\n。
    因为HttpParameters，HttpHeaders都为空，HttpString就是get\n/exampleobject\n\n\n形式了。
6. 生成 StringToSign
    根据 KeyTime 和 HttpString 生成 StringToSign，格式为sha1\nKeyTime\nSHA1(HttpString)\n。
    其中：
    sha1 为固定字符串。
    \n为换行符。
    SHA1(HttpString)使用如下来获取
```
    QCryptographicHash::hash(HttpString, QCryptographicHash::Sha1).toHex();
```
7. 生成 Signature
    使用 HMAC-SHA1 以 SignKey 为密钥，以 StringToSign 为消息，计算消息摘要，即为 Signature。
    这里调用如下API来获取
```
    hmacSha1(SecretKey , StringToSign)；
```
8. 合成Authorization。
    按照以下形式拼接即可。
```
    q-sign-algorithm=sha1&q-ak=SecretId&q-sign-time=KeyTime&q-key-time=KeyTime&q-header-list=HeaderList&q-url-param-list=UrlParamList&q-        signature=Signature
```
    SecretId、KeyTime、HeaderList、UrlParamList 和 Signature 通过上面步骤获取得到。

#### 发送请求
   所有请求信息都准备就绪后，就可以发送请求了。
   注：请求地址要写成http://你的桶域名/你的图片名。
          如：https://vnote-1259141668.cos.ap-guangzhou.myqcloud.com/moon.jpg
   调用QNetworkAccessManager的put API发送put请求
```
   如：manager->put(request, fdata);
```
   具体方法了解QT的网络方面。
   如果请求成之后，你的对象存储中就多了一张你上传上去的图片了。
   通过 https://vnote-1259141668.cos.ap-guangzhou.myqcloud.com/moon.jpg接口浏览到图片了

## 总结
   虽然这是很简单的东西，但我还是希望这篇文章能够帮助那些需要这方面知识的人。
