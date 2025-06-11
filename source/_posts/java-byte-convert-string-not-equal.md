---
title: java byte array 转String在转回byte array不相等
tags:
  - java
  - encode
categories:
  - java
date: 2023-09-21 20:39:21
abstracts: java byte array 转String在转回byte array不相等
feature: true
toc: true
---


# 背景

最近在搞微软的`NBFS`协议,这个协议实际上也是基于`WebService`,只不过对`xml`进行了压缩,按照他自己的编码规则进行压缩
网上搜罗一圈后发现有个大佬写好的`burp`的`NBFS`插件[WCF-Binary-SOAP-Plug-In](https://github.com/GDSSecurity/WCF-Binary-SOAP-Plug-In)
这个插件会将传入的经过`base64`编码的`xml`转换成`NBFS`协议的`base64`编码的字符串

测试那边要使用`jmeter`对这个`NBFS`接口性能测试
基本思路:
- 新建`http request`输入原始的`xml`
- 搞一个`PreProcessor`,调用大佬写的`NBFS.exe`获取压缩后的`XML`base64编码的字符串



PreProcessor 代码如下
<!--more-->

```groovy
import org.apache.commons.codec.binary.Base64;

// 获取请求体数据
def requestBody = sampler.getArguments().getArgument(0).getValue();

// 检查请求体是否存在
if (requestBody != null) {
    log.info("获取到请求体:{}",requestBody)
    // 进行加密操作，这里使用Base64编码作为示例
    def reqBase64Str = Base64.encodeBase64String(requestBody.getBytes());
    def process = Runtime.getRuntime().exec("./NBFS.exe","encode",reqBase64Str)
    process.waitFor()
    def nbfsEncodeBase64Str = process.inputStream.text
    def result = Base64.decodeBase64(nbfsEncodeBase64Str)
    // 将加密后的数据设置回请求体
    sampler.getArguments().getArgument(0).setValue(new String(result));
} else {
    log.warn("请求体不存在");
}
```

这时候就会发现一个神奇的东西,接口会返回`400`错误


# 问题分析


先说明下`NBFS`编码的原理,为了压缩`xml`,`NBFS`会预先定义一些字典,比如:`0x90`代表`Reason`这个字符,
也就是说,经过`NBFS.exe`返回的`Base64`字符串经过`Base64.decodeBase64(nbfsEncodeBase64Str)`这个方法返回的byte数组中可能出现`0x80,0xAA`等字节

众所周知,java默认的字符串编码是`UTF-8`

另一个常识是:java字符串是由`Char[]`表示的,而`Char`是由`unicode`表示

ok,有了上面的已知条件,返回400的问题,就是解释为什么下面代码输出是`false`就行了


```java
      byte[] byteArray = new byte[]{0x01, (byte)0x81, (byte)0xAA, 0x44, 0x45};
      String str = new String(byteArray);
      byte [] revertByteArray = str.getBytes();
      System.out.println(Arrays.equals(byteArray,revertByteArray));
```

其实断点调试下,会发现其实`revertByteArray`这个数组其实是:`[1, -17, -65, -67, -17, -65, -67, 68, 69]`而原数组是:`[1, -128, -86, 68, 69]`
区别就是多出了6个byte,其中`0x81`,`0xAA`对应变成`[-17,-65,-67]`,对应10进制的65533,对应的unicde就是:`\uFFFD`


接下来就分析一下为什么`0x81`为什么会变成`[-17,-65,-67]`这玩意就行了


# debug大法


对`new String(byteArr)`这段代码开始debug,
就会发现最终会进入一个超长的方法:

`sun.nio.cs.UTF_8.Decoder#decode`:

```java
// da 就是字符串找保存的char[]
// sa 就是String构造函数传进来的byte 数组
// 就是这个方法将byte [] -> char []
// sp 是0,应该是偏移量,      String str = new String(byteArray,1,1,"UTF-8");这么创建字符串才会有,这个不重要,
// len就是byte数组的长度
public int decode(byte[] sa, int sp, int len, char[] da) {
            final int sl = sp + len;
            int dp = 0;
            int dlASCII = Math.min(len, da.length);
            ByteBuffer bb = null;  // only necessary if malformed

            // ASCII only optimized loop
            //性能优化,>= 0的意思就是byte没有溢出,也就是:0~128,对应就是0x00 到 0x00 0x80
            while (dp < dlASCII && sa[sp] >= 0)
                da[dp++] = (char) sa[sp++];

            //下面就是范围判断,判断byte是否在utf8的编码范围内
            while (sp < sl) {
                int b1 = sa[sp++];
                if (b1 >= 0) {
                    // 1 byte, 7 bits: 0xxxxxxx
                    da[dp++] = (char) b1;
                } else if ((b1 >> 5) == -2 && (b1 & 0x1e) != 0) {
                    // 2 bytes, 11 bits: 110xxxxx 10xxxxxx
                    .
                    .
                    .
                } else if ((b1 >> 4) == -2) {
                    // 3 bytes, 16 bits: 1110xxxx 10xxxxxx 10xxxxxx
                    .
                    .
                    .
                } else if ((b1 >> 3) == -2) {
                    // 4 bytes, 21 bits: 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
                    .
                    .
                    .
                } else {
                    if (malformedInputAction() != CodingErrorAction.REPLACE)
                        return -1;
                    da[dp++] = replacement().charAt(0);
                }
            }
            return dp;
        }
```



上面代码翻译成人话就是:
循环byte数组里面的每一个byte
1. 如果>=0,就是ASCII
2. 如果不是,判断是否符合条件:
    1. 1 byte, 7 bits: 0xxxxxxx
    2. 2 bytes, 11 bits: 110xxxxx 10xxxxxx
    3. 3 bytes, 16 bits: 1110xxxx 10xxxxxx 10xxxxxx
    4. 4 bytes, 21 bits: 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
3. 如果都不符合,就返回`replacement()`第0个char(打断点看,这个字符其实就是main类启动时创建的,其实这就是'65533'也就是`\uFFFD`)


至此,真相大白

` 0x80 `->`0b10000000`
`0x88`->`0b10010000`

都不在上面的范围,所以他们最终的`char`最终都会变成`65533`(`\uFFFD`)

其实上面的规则就是`UTF-8`规则,参考[维基百科](https://zh.wikipedia.org/wiki/UTF-8):

>   在ASCII码的范围，用一个字节表示，超出ASCII码的范围就用字节表示，这就形成了我们上面看到的UTF-8的表示方法，这样的好处是当UNICODE文件中只有ASCII码时，存储的文件都为一个字节，所以就是普通的ASCII文件无异，读取的时候也是如此，所以能与以前的ASCII文件兼容。
>   大于ASCII码的，就会由上面的第一字节的前几位表示该unicode字符的长度，比如110xxxxx前三位的二进制表示告诉我们这是个2BYTE的UNICODE字符；1110xxxx是个三位的UNICODE字符，依此类推；xxx的位置由字符编码数的二进制表示的位填入。越靠右的x具有越少的特殊意义。只用最短的那个足够表达一个字符编码数的多字节串。注意在多字节串中，第一个字节的开头"1"的数目就是整个串中字节的数目。



# 太长不看

一句话总结:因为类似`0x80`,`0x88`等字节不在`utf-8`编码范围内,所以会返回一个默认字符,`\uFFFD`(65533),这个字符占3个字节,所以就造成的两个byte数组不相等

# 解决方法

解决方法很简单:
找一个编码覆盖`0x00` 到 `0xff`,并且只用一个字节编码的编码格式就行,
也就是`ISO-8859-1`

所以下面代码输出就是`true`

```java
      byte[] byteArray = new byte[]{0x01, (byte)0x81, (byte)0xAA, 0x44, 0x45};
      String str = new String(byteArray,"ISO-8859-1");
      byte [] revertByteArray = str.getBytes("ISO-8859-1");
      System.out.println(Arrays.equals(byteArray,revertByteArray));
```

