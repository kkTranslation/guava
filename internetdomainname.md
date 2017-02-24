# Introduction

[InternetDomainName](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/net/InternetDomainName.html) 是一个用于解析和操纵域名的有用工具。 它可以用作验证器，组件提取器和用于以类型安全方式传递域名的值类型。

然而，`InternetDomainName`特性的一些方面可能是令人惊讶的，这可能导致调用代码中的错误。 本文件解决了这些问题。

# 细节

## Public suffixes and private domains

根据相关的RFC规范，`InternetDomainName`对象保证在语法上有效，但不能保证对应于Internet上的实际可寻址域。 这是不可能做到这一点，没有做域名的净查找并试图与之联系，这是不可接受的开销大多数常见的情况。

但是，确定给定的域名是否可能表示Internet上的实际域通常是非常有用的。 为此，我们使用来自 [Public Suffix List \(PSL\)](http://publicsuffix.org/)的数据，该列表由Mozilla Foundation维护。 有`InternetDomainName`上的方法来确定给定域与PSL的关系。 要把它放在最基本的术语，如果`domain.hasPublicSuffix()`返回`true`，那么域可能对应一个真实的互联网地址; 否则，它几乎肯定不。

在这一点上，我们需要备份和定义一些术语。 有三个感兴趣的术语：

* [Top-Level Domain \(TLD\)](http://en.wikipedia.org/wiki/Top_level_domain): 由ICANN管理的single-label，例如`com`或`au`。
* [Public suffix](http://en.wikipedia.org/wiki/Public_Suffix_List): 人们可以在其中注册子域名，并且不应在其上设置Cookie的域。
  * Effective Top-Level Domain:  "public suffix"的已弃用同义词。

在继续之前，请仔细阅读链接的文章。



混乱的主要原因是人们说“TLD”，当他们的意思是"public suffix"。 这些是独立的概念。 所以，例如，

* `uk`是TLD，但不是public suffix
* `co.uk`是public suffix，但不是TLD
  * `squerf`既不是TLD也不是public suffix
  * `com`是TLD和public suffix

这种混乱是特别危险的，因为TLD有一个清晰，正式的定义，而不是public suffix。 最后，public suffix是一个可信源，要求PSL维护者添加到列表。 可信来源包括ICANN和国家域管理员，但也包括提供具有（模糊地）定义public suffix（独立的子域和超级脚本 [supercookie](http://en.wikipedia.org/wiki/HTTP_cookie#supercookie) 抑制）特征的服务的私人公司。 因此，例如，许多Google拥有的域名（例如`blogspot.com`）都包含在PSL中。

回到`InternetDomainName`，只要我们限制自己使用`hasPublicSuffix()`来验证域是一个合理的互联网域，一切都很好。 危险来自识别或提取“top private domain”的方法。 从技术角度来看，top private domain仅仅是public suffix之前最右边的超级域。 例如，`www.foo.co.uk`有一个公共后缀`co.uk`，和一个顶级私人域的`foo.co.uk`。

如`isUnderPublicSuffix()`，`isTopPrivateDomain()`和`topPrivateDomain()`的文档中所述，这些方法（主要是）唯一可靠的事情是确定在哪里可以设置cookie。 然而，许多人实际上试图做的是从子域中找到“真实”域或“所有者”域。 例如，在`mail.google.com`中，他们希望将`google.com`标识为所有者域。 所以他们写

```java
InternetDomainName owner = InternetDomainName.from("mail.google.com").topPrivateDomain();
```

...并且确定，`owner`最终与域`google.com`。 事实上，这个惯用语（和它喜欢的）工作了很多时间。 看起来直观的是“公共后缀下的域”在语义上应该等同于“所有者域”。

但它不是，这就是问题。 考虑`blogspot.com`，它出现在PSL中。 虽然它有公共后缀的特点 - 人们可以注册域下（对于他们的博客），并且不应该允许cookie设置它（防止跨博客cookie shenanigans），在互联网上它本身是一个可寻址域（恰好在我写这篇重定向到`blogger.com`，但可能很容易地改变）。

所以如果我使用上面的`foo.blogspot.com`上的惯用语，所有者将是同一个域，`foo.blogspot.com`。 这是我们讨论的术语中的正确答案，但对于许多人来说显然是令人惊讶的。

这里的主要课程是：

* TLD和public suffix不是一回事。
* Public suffixes 由人类定义，严格限制目的（主要是域验证和supercookie预防），并且不可预测地改变。
  * 在给定域与公共后缀的关系，该域响应Web请求的能力和该域的“所有权”之间没有定义的映射。
  * 您可以使用`InternetDomainName`来确定给定字符串是否表示Internet上合理可寻址的域，以及确定域的哪个部分可能允许设置Cookie。
  * 您不能使用`InternetDomainName`来确定某个域是否作为可寻址主机存在于互联网上，也不能使用超级域在业务或管理意义上“拥有”该域。

请记住，如果你不听从这个建议，您的代码将出现在一个巨大的各种投入工作......但失败的案例，所有的错误会发生，和失败案例的设置将为PSL更新纳入基本`internetdomainname`代码变更。

