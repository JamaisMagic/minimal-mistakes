---
title: "记与客户端开发联调一个rsa加密传参的事情"
description: "记与客户端开发联调一个rsa加密传参的事情"
excerpt_separator: "<!--more-->"
last_modified_at: 
comments: true
categories:
  -
tags:
  - rsa encrypt
  - base64
  - encode uri
---

本篇纯属吐槽。

这几日有个需求，背景就不说了。其中一件事情需要实现的东西是，app客户端使用public key和salt通过一定的规则加密一个字符串，放在url参数里传给前端，前端再拿这个参数传给服务端进行校验。

联调的时候，客户端说不行。（这个是最常见的说法）

顺便说一下调试方法。

先是看服务器日志，发现确实有错误信息，解不了密。服务器解析padding出错，以为是客户端加密用得padding不对，问了发现应该是对的。
然后让对方重新发一个加密后的字符串过来试试，发现可以。那就是传到url的时候的问题，没有做uri encode或者做得不对。
然后让对方把做了uri encode之后的字符串发过来看看，他把网址的整个url发过来，发现确实不对的。encode的地方不对，他把链接文件名（.html）后面的所有东西进行了encode，然后又发了一次是对整条url进行encode，然后我说单独对那个参数的值进行encode，对方试了就可以了。但是他已开始是如何encode的我就没兴趣知道了，这种问题其实没什么意义。

但是这只是解决了一端（iOS）的问题，还有另一位同事（Android），那个更让人头大。

Android那边是如何的呢，反正也是不行。

让他发个链接地址看看，用uri decode出来发现有空格，正常不应该有空格的，就让他再检查一下自己的代码，检查之后说没问题。好，然后还是像上面那样让对方把加密后的内容发过来看看，试了下是可以解密成功的。貌似rsa加密方面都没问题的，会不会有点标题党。那肯定又是uri encode的问题了。然而让他看代码还是说代码没问题。问他是怎么做uri encode的，他只说试了没问题，后来我把过程详细说了一遍，rsa加密之后base64，得到的东西再uri encode，再传到网址参数里，我还说了用android.net.Uri.encode 这个方法，然而没什么用，他还是说试了不行。我心想，难道要我上代码，虽然我确实用Android写了个demo并且试过是可以的。然后我继续问他，用了那个方法（android.net.Uri.encode）了吗？答曰，没用，因为其他项目都是base64之后直接拿来用的所以没用。我心想，没用那刚才又说试了不行。对于他说的原因我懒得再吐槽了。然后叫他用来试一下，然后他说用了，还是不行，BASE64Encoder.encode()和Uri.encode()，都不行。我心想，你不会是把uri encode替换了base64把，然后我说，理论上应该是这样 Uri.encode(BASE64Encoder.encode())，然后他再试了下，确实行了。

真是很少会遇到这种情况，首先在url传参数有特殊符号需要uri encode这个应该是常识了，居然说别的项目不需要uri encode就可以用，这个情况跟别的项目不一定相同啊。其次，让他试的东西没试就说试了不行这简直是要上黑名单的行为啊。第三，base64和uri encode只选其一说明他还是不知道uri encode是用来做什么的，或者说不知道为什么要uri encode，关键是说明文档里写了要注意uri encode的问题了，本来我觉得不用写的因为是常识。

联调是个辛苦活，运气不好的话浪费时间还一肚子火，而且是对方的话不一定信得过，例如试了不行，结果发现没试，或者试的方法不对。
这些东西真的要问清楚，问清楚对方是怎么做的，又是甚至需要看对方的代码，或者把自己的例子代码给对方参考。
