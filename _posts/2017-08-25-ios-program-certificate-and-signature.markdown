---
layout:     post
title:      "iOS证书和签名机制"
subtitle:   "数字签名和数字证书的重要性"
date:       2017-08-25 12:59:59
author:     "ElveXu"
header-img: "img/post-ios-program-certificate-and-signature-1.jpg"
tags:
    - Apple
    - iOS
    - Certificate
    - Signature
    - Development
---


<h4>非对称加密和摘要</h4>
<h5>非对称加密的特性和用法<h5>
<blockquote>非对称加密算法可能是世界上最重要的算法，它是当今电子商务等领域的基石。简而言之，非对称加密就是指加密密钥和解密密钥是不同的，而且加密密钥和解密密钥是成对出现。非对称加密又叫公钥加密，也就是说成对的密钥，其中一个是对外公开的，所有人都可以获得，称为公钥，而与之相对应的称为私钥，只有这对密钥的生成者才能拥有。</blockquote>

<p>公私钥具有以下重要特性：</p>

<li>对于一个私钥，有且只有一个与之对应的公钥。生成者负责生成私钥和公钥，并保存私钥，公开公钥
<li>公钥是公开的，但不可能通过公钥反推出私钥，或者说极难反推，只能穷举，所以只要密钥足够长度，要通过穷举而得到私钥，几乎是不可能的
<li>通过私钥加密的密文只能通过公钥解密，公钥加密的密文只有通过私钥解密

<p>由于上述特性，非对称加密具有以下的典型用法：</p>

<li>对信息保密，防止中间人攻击：将明文通过接收人的公钥加密，传输给接收人，因为只有接收人拥有对应的私钥，别人不可能拥有或者不可能通过公钥推算出私钥，所以传输过程中无法被中间人截获。只有拥有私钥的接收人才能阅读。此用法通常用于交换对称密钥。
<li>身份验证和防止篡改：权限狗用自己的私钥加密一段授权明文，并将授权明文和加密后的密文，以及公钥一并发送出来，接收方只需要通过公钥将密文解密后与授权明文对比是否一致，就可以判断明文在中途是否被篡改过。此方法用于数字签名。
<br>
<blockquote>著名的RSA算法就是非对称加密算法，RSA以三个发明人的首字母命名。
非对称加密算法如此强大可靠，却有一个弊端，就是加解密比较耗时。因此，在实际使用中，往往与对称加密和摘要算法结合使用。对称加密很好理解，此处略过1w字。我们再来看一下摘要算法。</blockquote>

<h5>摘要算法</h5>
<blockquote>另一个神奇的算法就是摘要算法。摘要算法是指，可以将任意长度的文本，通过一个算法，得到一个固定长度的文本。这里文本不一定只是文本，可以是字节数据。所以摘要算法试图将世间万物，变成一个固定长度的东西。</blockquote>

<p>摘要算法具有以下重要特性：</p>

<li>只要源文本不同，计算得到的结果，必然不同
<li>无法从结果反推出源（那是当然的，不然就能量不守恒了）
<li>典型的摘要算法，比如大名鼎鼎的MD5和SHA。摘要算法主要用于比对信息源是否一致，因为只要源发生变化，得到的摘要必然不同；而且通常结果要比源短很多，所以称为“摘要”。

<h4>数字签名</h4>
<blockquote>理解了非对称加密和摘要算法，来看一下数字签名。实际上数字签名就是两者结合。假设，我们有一段授权文本，需要发布，为了防止中途篡改文本内容，保证文本的完整性，以及文本是由指定的权限狗发的。首先，先将文本内容通过摘要算法，得到摘要，再用权限狗的私钥对摘要进行加密得到密文，将源文本、密文、和私钥对应的公钥一并发布即可</blockquote>

<p>那么如何验证呢？</p>
<blockquote>验证方首先查看公钥是否是权限狗的，然后用公钥对密文进行解密得到摘要，将文本用同样的摘要算法得到摘要，两个摘要进行比对，如果相等那么一切正常。这个过程只要有一步出问题就视为无效。</blockquote>

<h5>数字证书：用数字签名实现的证书</h5>
<blockquote>实际上，数字证书就是通过数字签名实现的数字化的证书。在一般的证书组成部分中，还加入了其他的信息，比如证书有效期（好比驾驶证初次申领后6年有效），过了有效期，需要重新签发（驾驶证6年有效后需重新申领）。</blockquote>
<br>
<blockquote>跟现实生活中的签发机构一样，数字证书的签发机构也有若干，并有不同的用处。比如苹果公司就可以签发跟苹果公司有关的证书，而跟web访问有关的证书则是又几家公认的机构进行签发。这些签发机构称为CA（Certificate Authority）。</blockquote>
<br>
<blockquote>对于被签发人，通常都是企业或开发者。比如需要搭建基于SSL的网站，那么需要从几家国际公认的CA去申请证书；再比如需要开发iOS的应用程序，需要从苹果公司获得相关的证书。这些申请通常是企业或者开发者个人提交给CA的。当然申请所需要的材料、资质和费用都各不相同，是由这些CA制定的，比如苹果要求$99或者$299的费用。</blockquote>
<br>
<blockquote>之所以要申请证书，当然是为了被验证。英语6级证书的验证方一般是用人单位；web应用相关的SSL证书的验证方通常是浏览器；iOS各种证书的验证方是iOS设备。我们之所以必须从CA处申请证书，就是因为CA已经将整个验证过程规定好了。对于iOS，iOS系统已经将这个验证过程固化在系统中了，除非越狱，否则无法绕过</blockquote>
<br>

<h4>证书的授权链</h4>
<blockquote>数字证书可能还包括证书链信息(级联信任)</blockquote>
<br>
<blockquote>从苹果MC（Member Center）中获得的证书实际也是一个包含有证书链的证书，其中的根是苹果的CA。我们获得的证书实际上是在告诉iOS设备：我们的证书是被苹果CA签过名的合法的证书。而iOS设备在执行app前，首先要先验证CA的签名是否合法，然后再通过证书中我们的公钥验证程序是否的确是我们发布的，且中途没有对程序进行过篡改。 </blockquote>
<br>

<h5>iOS证书申请和签名打包流程图</h5>

![](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/ios-program-certificate-and-sign/ios-package-flow.png)

<blockquote>这张图阐述了，开发iOS应用程序时，从申请证书，到打包的大致过程</blockquote>


<h4>证书申请</h4>
<p>//TODO</p>

<h4>iOS授权和描述文件</h4>

<p>//TODO</p>


<h4>iOS代码签名</h4>
<h5>IPA包结构</h5>
<blockquote>iOS程序最终都会以.ipa文件导出，ipa文件的结构</blockquote>

<img src="https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/ios-program-certificate-and-sign/payload-st.png">

<br>
<blockquote>事实上，ipa文件只是一个zip包，可以使用如下命令解压：</blockquote>

<br>
<p>
/usr/bin/unzip -q xxx.ipa -d <destination>
</p>

<blockquote>解压后，得到上图的Payload目录，下面是个子目录，其中的内容如下：</blockquote>

<li>资源文件，例如图片、html、等等。
<li>CodeSignature/CodeResources。这是一个plist文件，可用文本查看，其中的内容就是是程序包中（不包括Frameworks）所有文件的签名。注意这里是所有文件。意味着你的程序一旦签名，就不能更改其中任何的东西，包括资源文件和可执行文件本身。iOS系统会检查这些签名。
<li>可执行文件。此文件跟资源文件一样需要签名。
<li>一个mobileprovision文件.打包的时候使用的，从MC上生成的。
<li>Frameworks。程序引用的非系统自带的Frameworks，每个Frameworks其实就是一个app，其中的结构应该和app差不多，也包含签名信息CodeResources文件
<h4>App重新签名的流程</h4>

<img src="https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/ios-program-certificate-and-sign/resign-flow.png">

<blockquote>流程描述</blockquote>
<br>
  <li>首先解压ipa
  <li>如果mobileprovision需要替换，替换
  <li>如果存在Frameworks子目录，则对.app文件夹下的所有Frameworks进行签名，在Frameworks文件夹下的.dylib或.framework
  <li>对xxx.app签名
  <li>重新打包


<h5>EOF</h5>


