# 深入学习SAML2.0

众所周知，安全断言标记语言（SAML）2.0提供了两个重要特性：跨域单点登录（SSO）和身份联合。SAML2.0已被许多企业环境采用，因为它使企业能够让员工、客户和合作伙伴使用的应用程序将用户身份验证委托给集中的企业身份提供者。这为企业提供了一个身份管理和控制中心。如果您正在为大型企业客户编写应用程序，他们可能希望您的应用支持使用SAML2.0进行身份验证[](https://wiki.oasis-open.org/security/FrontPage)。
在本章中，我们将概述SAML2.0，它的设计用来解决什么问题，以及SAML2.0中的跨域单点登录和身份联合特性。我们还将提供一个融合的解决方案，说明如何利用OIDC这样的较新协议，并且仍然有效地实现对SAML的支持。

## 解决的问题

根据我们的经验，使用SAML最常见的用例是跨域单点登录。在此场景中，用户需要访问位于不同域中的多个应用程序，例如application1.com和application2.com。如果没有跨域单点登录，用户可能需要在每个应用程序中建立一个帐户，然后分别登录到每个应用程序。这意味着用户可能需要记住许多不同的用户名和密码。如果用户是公司员工，并且应用程序是SaaS应用程序，那么企业将很难管理其员工可能创建的所有SaaS应用程序帐户。

SAML被设计成一个“基于XML的框架，用于线上业务伙伴之间的安全的描述和交换信息” [ii](https://www.oasis-open.org/committees/download.php/27819/sstc-saml-tech-overview-2.0-cd-02.pdf)。SAML可以让应用程序能够将用户身份验证委托给身份提供者。身份提供者对用户进行身份验证，并向应用程序返回一个断言，其中包含有关已验证用户和身份验证事件的信息。如果用户访问第二个应用，此时用户已经在身份提供者进行了认证，则用户将无需再次提示登录。

SAML还为应用程序和身份提供者提供了一种机制，以便为用户使用公共共享标识符来交换有关用户的信息，即联邦身份。联邦标识可以跨系统使用相同的标识符，也可以使用不透明的内部标识符，该标识符映射到每个系统中用户使用的标识符。我们将在下面的部分中了解有关这些功能如何工作的更多信息。

首先，我们需要指定一些要使用的术语。

## 术语

SAML规范定义了以下术语：

* 主题–将交换安全信息的实体。主题通常是指一个人，但可以是任何能够进行身份验证的实体，包括软件程序。
* SAML断言–一种基于XML的消息，包含有关主题的安全信息。
* SAML Profile–定义如何将SAML消息用于业务用例（如跨域单点登录）的规范。
* 身份提供者–身份提供者是一个服务器，它在跨域单点登录的上下文中发布有关已验证主题的SAML断言。

* 服务提供者–服务提供者将身份验证委托给身份提供者，并依赖于身份提供者在跨域单点登录上下文中发出的SAML断言中有关已验证主题的信息。
* 信任关系–SAML服务提供者和SAML身份提供者之间的协议，其中服务提供者信任身份提供者发布的断言。
* SAML协议绑定–描述如何将SAML消息元素映射到标准通信协议（如HTTP）上，以便在服务提供者和身份提供者之间进行传输。实际上，SAML请求和响应消息通常通过HTTPS发送，使用HTTP重定向或HTTP-POST，分别使用HTTP重定向和HTTP-POST绑定。

## SAML如何工作

最常见的SAML场景是跨域web单点登录。在此场景中，主题是希望使用应用程序的用户。应用程序充当SAML服务提供者。应用程序将用户身份验证委托给可能位于不同安全域中的SAML身份提供者。身份提供者对用户进行身份验证，并将安全令牌（称为SAML断言）返回给应用。SAML断言提供有关身份验证事件和身份验证主题的信息。为了各章的一致性，我们将使用术语应用和服务提供者，但是应该注意，作为身份提供者的实体也可以通过进一步将身份验证委托给另一个身份提供者来充当服务提供者。

为了建立跨域web单点登录的能力，服务提供者（应用程序）和身份提供者之间需要组织交换信息，即元数据。元数据信息包含URL端点和证书等信息，用于验证数字签名的消息。此数据使双方能够交换消息。元数据用于配置和设置服务提供者和身份提供者之间的信任关系，并且必须在身份提供者为服务提供者（应用）验证用户之前完成。

一旦双方的配置就绪，当用户访问服务提供者（应用）时，它会使用SAML身份验证请求消息将用户的浏览器重定向到身份提供者。身份提供者对用户进行身份验证，并使用SAML身份验证响应消息将用户重定向回应用程序。响应包含一个SAML断言，其中包含有关用户和身份验证事件的信息，如果出现错误情况，则响应包含错误信息。身份提供者可以根据需要为每个服务提供者定制断言中的身份声明。

### 应用SP发起的SSO

跨域单点登录的最简单形式如图8-1所示。在此示例中，用户从服务提供商（SP）（应用程序）开始，因此称为“SP启动”流(该图描述了该场景，其中用户在身份提供者处没有现有的身份验证会话，因此必须进行身份验证）。

![8-1 SAML SP-Initiated.jpg](https://i.loli.net/2021/07/19/3n1IJDpqwYVevmM.jpg)

<center>8-1 SP发起的SSO</center>

1. 用户向应用（Servie Provider）发起访问请求
2. 应用向身份提供者(Identity provider)发出一个SAML Request请求，该请求通过浏览器进行重定向。
3. 身份提供者(Identity provider)接下来将向用户发起身份验证。
4. 用户通过输入凭据进行身份验证。
5. 身份提供者(Identity provider)使用包含SAML身份验证断言的SAML响应,通过用户的浏览器重定向回服务提供者（SP）。响应被发送到服务提供者的断言使用者服务（Assertion Consumer Service-ACS）的URL。
6. 服务提供者（SP）使用并验证SAML响应，如果通过验证，响应用户的原始请求。



###  身份服务IdP发起的SSO

图8-1显示了从服务提供者（应用程序）开始与用户的交互序列。这被称为应用发起的单点登录，因为用户在服务提供商（SP）发起交互。SAML还定义了另一个流程，称为“IdP initiated”-身份服务发起的SSO，其中用户从身份提供者（IdP）开始，如图8-2所示。在这种情况下，身份提供者使用SAML响应消息将用户的浏览器重定向到服务提供者，而服务提供者没有发送任何身份验证请求。在某些企业环境中，用户可以通过企业门户访问应用程序，从而可以使用这种流程。

这种场景如下描述：当用户最初访问公司门户时，他们将被重定向到公司身份提供程序以登录。登录后，用户将返回到门户，并在门户上看到应用程序链接的菜单/图标。单击其中一个链接将用户重定向到身份提供程序，并将应用程序URL作为参数。身份提供程序检测到用户已经有一个经过身份验证的会话，并使用SAML响应消息将用户的浏览器重定向到应用程序，与SP initiated的情况相同。
事实上，IdP发起的SSO，并不一定是需要门户，但是选择了使用门户是IdP发起SSO的一种常见方式。

IdP发起的带有门户的流在企业中很有用，因为它提供了单点登录，并确保用户访问每个应用程序的正确URL，从而降低了用户被钓鱼的风险。IdP启动的流程如图7-2所示。（此图假设用户没有现有的身份验证会话，登录门户实际上是个SP发起SSO的流程，后半部分，用户访问SP是IdP发起SSO的流程。）

![8-2 SAML IdP-Initiated.jpg](https://i.loli.net/2021/07/19/1xYnSFjgClaWVKz.jpg)

<center>8-2 IdP发起的SSO</center>

1. 用户向门户发起访问请求
2. 门户向身份提供者(Identity provider)发出一个SAML Request请求，该请求通过浏览器进行重定向。
3. 身份提供者(Identity provider)接下来将向用户发起身份验证。
4. 用户通过输入凭据进行身份验证。
5. 身份提供者(Identity provider)使用包含SAML身份验证断言的SAML响应,通过用户的浏览器重定向回门户。响应被发送到门户的断言使用者服务（Assertion Consumer Service-ACS）的URL。如果该断言通过门户服务的验证，门户展示用户可以访问的应用列表。
6. 用户点击要访问的应用的链接，该链接将通过浏览器重定向到身份服务者，同时将本次用户需要访问的应用的URL附带在参数中。
7. 身份服务者检查用户的Session是否还存在，如果存在，IdP使用包含SAML身份验证断言的SAML响应,通过用户的浏览器重定向回服务提供者（SP）。响应被发送到服务提供者的断言使用者服务（Assertion Consumer Service-ACS）的URL。
8. 服务提供者（SP）使用并验证SAML响应，如果通过验证，向用户展示该应用的页面。



### 身份联邦

通过SAML，身份联合建立了在服务提供者（应用程序）和身份提供者之间使用的一致的标识符，以指向同一（用户）主体。这使得服务提供商能够将用户的认证委托给身份提供商，并接收具有身份声明的认证断言来判别用户。

图8-3举例说明。一个名为Alice Wang的用户在两个应用程序中拥有一个帐户，Application 1托管在app1.com，Application2托管在app2.com。在Application1中，她的帐户标识符是alice@corp.com，在application2中，她的帐户标识符是“alice”。Alice在企业身份服务提供者也有一个账户，她的账户标识是alice@corp.com.

![8-3 Identity Federation.jpg](https://i.loli.net/2021/07/19/fCneOhQEjwpis7k.jpg)

<center> 8-3 身份联邦</center>

身份提供者的管理员向 Application 1和Application2的管理员提供身份服务有关其环境的元数据，二者使用元数据在Application1/2和身份提供者之间设置联合认证的配置。身份提供者将会向其配置的每个服务提供者发送断言，这些断言包含服务提供者（应用程序）的适当标识符和属性。

当Alice访问Application1时，它通过浏览器重定向到身份提供者“corp.com”。身份提供者对Alice进行身份验证，验证完成后，将alice@corp.com作为身份属性标识，并将包含该标识的身份验证断言重定向回application1。Application1对Alice使用相同的标识符，它基于该标识识别alice。

当Alice访问application2时，类似的，它通过浏览器重定向到身份提供者“corp.com”，此时如果alice已经验证过，无需再进行验证。如果身份提供者返回一个身份验证断言，将她标识为alice@corp.com，那么Application2则无法识别她，不会将该名称认为是一个有效用户。身份提供者需要为不同服务提供者返回适当的标识符。在这种情况下，当身份提供者将身份验证断言传递给application2时，它需要使用“alice”来标识断言的主题。

服务提供者的身份和身份提供者之间的关系可以以不同的方式设置。在实践中，用户的电子邮件通常在服务提供商和身份提供商处被用作用户的标识符，但是这可能是有问题的，因为用户可能需要更改其电子邮件地址，并且它可能与隐私要求相冲突。可以在请求中动态请求特定标识符属性的使用，或者可以将身份提供者配置为向服务提供者发送特定标识符。身份提供者和服务提供者还可以使用用户的不透明内部标识符交换信息，该标识符在每侧映射到用户的概要文件。为每个联合体使用唯一标识符有利于隐私保护，并防止用户活动的关联，但在实践中并不常见，该方法需要在双方交换元数据并配置其基础结构以在服务提供者和身份提供者之间建立关系时设置。



## 中间代理

身份验证中间代理的主要目的是为了支持多种身份验证协议和机制。有的客户希望他们的用户使用公司SAML身份提供者的认证服务。

但是，SAML是一个复杂的协议，需要大量的工作来实现。而基于OIDC协议，则相对来说要较为简单些。那么验证的中间代理就可以充当一个中间协议转换的角色。根据不同应用需要认证协议的不同，提供不同的协议支持。

如图8-4所示，该应用程序使用OIDC和OAuth 2.0协议，通过一个身份验证中间代理代理，能够与多个身份提供者通信，每个身份提供者使用不同的协议。身份验证代理的使用允许应用程序团队在其应用程序中实现较新的身份协议，并将重点放在其应用程序的核心功能上，而不是花费时间直接实现和支持客户请求的较旧的身份协议。

![8-4-IDP Broker.jpg](https://i.loli.net/2021/07/19/VoqrRnTc1GymUFC.jpg)



<center>8-4 IdP中间代理</center>

## 配置说明

表8-1和8-2列出了通常需要在服务提供者（应用程序）和身份提供者处配置的参数及其含义。

| 参数               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| SSO URL            | 身份提供者的单点登录URL的入口，应用向该URL请求身份验证。     |
| Certificate        | 来自身份提供者的证书。用于验证来自身份提供程序的SAML响应/断言上的签名，同时也可在应用发送加密请求时使用。某些提供程序允许两种用途使用不同的证书。 |
| Protocol binding   | 指定发送请求时使用何种协议进行绑定。典型的使用HTTP-Redirect，或者 HTTP-POST 用于直接请求。 |
| Request signing    | 用于指示是否对SAML身份验证请求进行数字签名，如果是，通过哪个签名算法。建议签名。 |
| Request encryption | 是否对SAML身份验证请求进行加密。                             |

<center>8-1 SAML认证 常用SP配置</center>

| 参数               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| ACS URL            | 身份提供者的单点登录URL的入口，应用向该URL请求身份验证。     |
| Certificate        | 来自身份提供者的证书。用于验证来自身份提供程序的SAML响应/断言上的签名，同时也可在应用发送加密请求时使用。某些提供程序允许两种用途使用不同的证书。 |
| Protocol binding   | 指定发送响应时使用何种协议进行绑定。典型的使用 HTTP-POST 。  |
| Request signing    | 用于指示是否对SAML身份验证响应进行数字签名，如果是，通过哪个签名算法。签名是强制性的。 |
| Request encryption | 是否对SAML身份验证响应进行加密。                             |

<center>8-2 SAML认证 常用IDP配置</center>



## 本章重点回顾

SAML2.0为web单点登录和身份联合提供了行业标准解决方案。

* SAML是一个基于XML的框架，用于在业务伙伴之间交换安全信息。
* 应用服将用户身份验证委托给一个身份供应商。
* SAML身份提供者对用户进行身份验证，并在身份验证响应的XML消息中返回用户身份验证事件的结果。
* 身份验证响应包含一个身份验证断言，其中包含有关身份验证事件和已验证用户的声明。
* 身份提供者对不同的应用可以提供不同的用户身份的声明。

这些关键特性，使企业能够使用云应用程序，并且仍然保持对身份的集中控制。SAML的使用消除了应用程序对静态密码凭证的暴露，并为用户提供了单点登录的便利。

国外因为SaaS发展的比较早，且当时尚无OIDC的协议，因此，很多国外的SaaS应用支持SAML的协议。

然而，既然SAML是一个较旧的协议，OIDC和OAuth 2.0已经存在，围绕API设计的现代应用程序选择较新的协议更有优势，因为它们分别不仅支持身份验证，还支持API授权，并且面向消费者和企业的场景中都能够适用。

如果您需要支持SAML，最好不是自己在一个新的应用程序中实现它，使用身份验证代理服务会是一个更好的选择。这将大大简化SAML协议开发的复杂工作，使您能够使用较新的协议实现应用程序，但仍然支持需要较旧协议（如SAML和WS-Fed）。

一旦解决了身份验证，就需要授权和策略执行来控制用户可以做什么，这将在下一章中介绍。



## 参考：

i: https://wiki.oasis-open.org/security/FrontPage


