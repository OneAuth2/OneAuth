# OpenID Connect

如前一章所述，OAuth2.0提供了一个框架，用于授权应用程序调用API，但不是为验证应用程序的用户而设计的。
OpenID Connect（OIDC）[i](https://openid.net/connect/) 协议在Oauth2.0之上提供了一个身份服务层，旨在允许授权服务器对应用程序的用户进行身份验证，并以标准的方式返回结果。OAuth2.0的一些实现为此添加了专有的附加内容，但是需要一个标准的解决方案。在本章中，我们将更详细地描述OIDC解决的问题和应用程序如何使用OIDC对用户进行身份验证。

## 解决的问题

OIDC协议旨在解决某些场景下，用户需要通过身份验证才能访问应用程序的身份验证问题。OIDC使应用程序能够将用户身份验证委托给OAuth2.0授权服务器，并以标准格式将其返回给应用程序关于已验证用户和身份验证事件的声明。

图 7-1对这一场景提供了示例的说明。

![7-1 OIDC.jpg](https://i.loli.net/2021/07/16/Z9zF7KAjG8VlOPE.jpg)

<center>7-1 OIDC 概览</center>

当用户访问应用程序时，它会将用户的浏览器重定向到实现OIDC的授权服务器。 OIDC将这样的授权服务器称为OpenID提供者(OpenID Provider)，我们将在本章中使用该术语。OpenID提供者与用户交互以验证他们的身份（假设他们还没有登录）。身份验证后，用户的浏览器将重定向回应用程序。应用程序可以请求在称为ID令牌的安全令牌中返回关于已验证用户的声明。或者，它可以请求Oauth2.0访问令牌，并使用它调用OpenID提供者的UserInfo端点来获取声明。因为OIDC是OAuth2.0之上的一层 ，应用程序可以使用OpenID提供程序进行用户身份验证和授权，以调用OpenID提供程序的API。通过这个示例，我们了解了OIDC的基本概念。下一节将定义一些附加术语，以便我们提供进一步的细节和更准确的描述。

## 术语

我们将对以上提到的术语进行进一步准确的定义和解释：

* 最终用户End User – 最终用户是要验证的主体。 (我们将使用术语“用户”来简化和保持各章节的一致性。）

* OpenID提供者（OP） – OpenID提供程者是Oauth2.0授权服务器实现OIDC，并对用户进行身份验证，将有关已验证用户和身份验证事件的声明返回给依赖方（应用程序）。
* 依赖方（RP） – OAuth 2.0 将用户身份验证委托给OpenID提供者，并从OpenID提供程序请求有关用户的声明的客户端。对于依赖方，我们通常使用“应用”一词 来保持跨章节的一致性。

### 公共与非公开的客户端

OIDC定义了客户端的类型与OAuth2.0定义的两种客户端类型类似：

* 公共客户端 – 一种主要在用户侧设备上运营的程序，如（手机APP应用程序）或客户端浏览器中执行的应用程序，它不能安全地存储秘密。OAuth2.0 本机应用程序定义了最佳的实践。[ii](https://datatracker.ietf.org/doc/html/rfc8252)
* 非公开客户端 – 在受保护的服务器上运行的一种应用程序，它可以安全地存储机密，可以一种安全的机制向授权服务器进行身份验证，并存储授权服务器返回的机密。

### 令牌和授权码

OIDC使用授权码、访问令牌和刷新令牌，如OAuth2.0所述，并定义一个ID令牌。

* 授权码 – 返回给应用程序的一种中间的不透明一次性代码，用于获取访问令牌和可选的刷新令牌。每个授权码使用一次。
* 访问令牌：应用程序用来访问API的令牌。 它表示应用程序调用API的授权，并且有一个过期时间
* 刷新令牌：一种可选令牌，当先前的访问令牌过期时，应用程序可以使用它来请求新的访问令牌。
* ID令牌：一种令牌，用于向依赖方（应用程序）传递有关认证事件和用户的声明。

### 端点

OIDC除了利用OAuth 2.0中描述的的授权和令牌端点以外，增加加UserInfo端点。

UserInfo Endpoint–返回关于经过身份验证的用户的声明。调用端点需要访问令牌，并且返回的声明由访问令牌控制。



### ID Token

ID Token令牌是OpenID提供者用来向应用程序传递关于身份验证事件和身份验证用户的声明的安全令牌。ID令牌以Json Web Token（JWT）[iii](https://tools.ietf.org/html/rfc7519)格式编码。图7-2显示了一个示例ID令牌。



![7-2 OIDC IDToken.jpg](https://i.loli.net/2021/07/16/IraHYnfyRzp8qTl.jpg)

<center>7-2 ID Token</center>

JWT格式旨在在双方之间传递声明。作为JWT，ID令牌由报头（Header）、有效负载（Payload）和签名（Signature）组成。最终，这些组合在一起通过base-64进行编码。

* ID令牌的头部分

  包含关于对象类型（JWT）和用于保护有效负载中声明的完整性的特定签名算法的信息。常见的签名算法有HS256（HMAC带SHA256）或RS256（RSA带SHA256签名）。

* 有效负载（Payload）部分

  包含关于用户和身份验证事件的声明。主要包括：

  * 发行人

  * 主体或经过身份验证的用户（通常是资源所有者）

  * 用户如何认证以及何时认证

  * 令牌的对象（即受众）

    这些标记非常灵活，允许您添加自己的声明

* 签名部分

  包含基于ID令牌的有效负载部分的数字签名和OpenID提供者已知的密钥。

  OpenID提供者根据签名规范（JWS）[iv](https://datatracker.ietf.org/doc/html/rfc7515)对JWT进行签名。RP应用程序可以验证ID令牌上的签名，以检查其中声明的完整性。为了保密，OpenID提供者可以选择在JWT签名后使用Jsonweb加密（JWE）[v](https://datatracker.ietf.org/doc/html/rfc7516)对其进行加密。如果这样做，它将生成一个嵌套的JWT。

OIDC规范中，ID令牌JWT的Payload定义了一组适用于所有类型的OIDC身份验证的用户和身份验证事件的声明[vi](https://openid.net/specs/openid-connect-core-1_0.html#IDToken)。如表7-1所示。

| Claims 声明 | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| iss         | ID令牌的颁发者，以URL格式标识。颁发者通常是OpenID提供者。“iss”声明不应包括URL查询或片段组件。 |
| sub         | 唯一的（在OpenID提供者中），对已验证用户或主题实体区分大小写的字符串标识符，长度不超过255个ASCII字符。决不应将子声明中的标识符重新分配给新用户或实体。 |
| aud         | ID令牌所针对的依赖方（应用程序）的客户端ID。单个区分大小写的字符串，如果有多个访问群体，则可以是区分大小写的字符串的数组。 |
| jti         | Jti声明为JWT提供了唯一的标识符。 实现JTI以唯一地标识JWT可以帮助防止攻击者发送相同JWT以发出请求的重放攻击.服务器将生成jti值,并在每个响应上将其与新的JWT一起发送.当收到新请求时,服务器必须验证jti值(以确保它之前没有使用过). |
| iat         | 颁发ID令牌的时间，指定为自1970年1月1日00:00:00 UTC到ID令牌颁发时间的秒数。 |
| exp         | ID令牌的过期时间，指定为自1970年1月1日00:00:00 UTC到令牌过期时间的秒数。应用程序必须在这个时间之后考虑ID令牌无效，允许对时钟偏移允许几分钟的公差。 |
| idp         | 颁发令牌的IDP的唯一标识                                      |
| auth_time   | 用户通过身份验证的时间，指定为自1970年1月1日00:00:00 UTC到身份验证时间的秒数。 |
| nonce       | 从依赖方传入的身份验证请求中传递的不可使用的、区分大小写的字符串值，由OpenID提供程序添加到ID令牌中，以将ID令牌链接到依赖方应用程序会话，并便于检测重播的ID令牌。 |
| amr         | 【可选】身份验证方法引用的字符串–用于指示用于对ID令牌的主题实体进行身份验证的身份验证方法。认证方法参考值规范[vii](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-amr-values-04)为该声明定义了一组初始标准值。 |
| acr         | 【可选】包含身份验证上下文类引用的字符串–用于指示用于对ID令牌的主题进行身份验证的身份验证机制的身份验证上下文类。值可以由OpenID提供者决定，也可以由依赖方和OpenID提供者商定，并且可以使用诸如OpenID Connect扩展身份验证配置文件ACR值草案之类的标准[viii](https://openid.net/specs/openid-connect-eap-acr-values-1_0.html) |
| azp         | 【可选】Authorized party，结合aud使用。只有在被认证的一方和受众（aud）不一致时才使用此值，一般情况下很少使用。 |

ID令牌可以包含表7-1中所列之外的额外声明。可以添加的附加声明包含：username, given_name, family_name, email, email_verified, locale, and picture等等。附加标准声明的列表可以参考OIDC核心规范的第5.1节[ix](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)。特定类型的OIDC请求（流）可能涉及附加声明。也可以添加自定义的声明，如IDP等。



## OIDC如何工作

与OAuth2.0中定义的授权流程类似，OIDC流程围绕不同应用程序的类型而设计的，OIDC核心规范定义了以下流程：

* 授权码流程
* 隐式流程
* 混合流程

以下各节将介绍每种方法的工作原理:

### OIDC 授权码模式

OIDC授权代码流类似于OAuth2.0授权代码授，整个流程中，主要依赖于两次请求和一个中间授权代码。为了验证用户身份，应用程序将用户的浏览器重定向到OpenID提供者。OpenID提供者对用户进行身份验证，并使用授权码将用户的浏览器重定向回应用程序。应用程序使用授权代码从OpenID提供者的令牌端点获取ID令牌、访问令牌和可选的刷新令牌。图7-3描述了这个流程，假设用户当前没有登录会话，应用程序最终请求了全部的三个安全令牌（IDToken、AccessToken、Refresh Token），此图包含了如第6章所述的PKCE使用方式。

![7-3 OIDC CodeFlow.jpg](https://i.loli.net/2021/07/16/jSY3WfZekLt4GO7.jpg)

<center>OIDC 授权码流程</center>



1. 用户向应用（RP）发起请求
2. 应用向OpenID 提供者发出一个请求，发送一个基本的授权范围，该请求通过浏览器进行重定向。
3. OpenID提供者接下来将向用户发起身份验证，和征求用户的授权同意。
4. 用户通过输入凭据进行身份验证，并且对请求的权限进行授权，该授权将影响令牌中包含影响作用域的信息。
5. OpenID提供者生成本次授权请求的授权码，然后将此授权码包含在重定向请求中，通过浏览器将授权码返回给应用。
6. 应用向OpenID提供者的令牌端点发送获取令牌请求，在此请求中，包含了授权码，以获取访问令牌。
7. OpenID提供者向该应用颁发访问令牌（AccessToken）、ID 令牌和刷新令牌（可选）。
8. 应用使用此访问令牌来调用OpenID提供者的用户信息的API。

> 注意：

步骤6调用令牌端点以获取安全令牌时，如果应用程序是非公开客户端，能够向OpenID提供程序进行自己的身份验证，则不需要PKCE的模式。如果，应用是不能安全地维护此类身份验证的秘密的公共客户端应用程序，可以使用密码交换证明密钥（PKCE）。PKCE的使用旨在降低授权代码被未授权方截获的风险。以下示例请求假定使用PKCE。

#### 认证请求

应用程序通过浏览器将用户的身份验证请求重定向到OpenID提供程序的授权端点，例如

```
GET /authorize? 

response_type=code

& client_id=<client_id>

& state=<state_value>

& nonce=<nonce_value>

& scope=<scope>

& redirect_uri=<callback_url>

& code_challenge=<code_challenge>& code_challenge_method=<code_challenge_method> HTTP/1.1 Host: authorizationserver.com
```

该示例中使用的参数如表7-2所示，但不同的OpenID提供者会有所差异。

| 参数                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| response_type         | 指示OIDC授权类型。“code”用于指示该流程为授权码流程。         |
| client_id             | 应用程序的标识符，在向OpenID服务器注册时分配。               |
| response_mode         | 一个可选参数，请求授权服务器用于向客户端应用程序传递响应参数为非默认方式时，可以指定该参数 |
| state                 | 一个不可猜测的字符串，对于每个调用都是唯一的，对授权服务器来说是不透明的，客户端使用它来跟踪相应请求和响应之间的状态，以降低CSRF攻击的风险。它应该包含一个将请求与用户会话相关联的值。这可以通过将会话cookie或其他会话标识符的散列与附加的每个请求唯一组件连接起来来实现。当收到响应时，客户机应确保响应中的状态参数与从同一浏览器发送的请求的状态参数匹配。 |
| scope                 | 本次请求授予权限的范围。例如：openid，profile，email         |
| nonce                 | 在请求中传递给OpenID提供者的不可猜测的值，如果请求了ID令牌，则在ID令牌中返回该值未修改的声明。用于防止令牌重放。 |
| redirect_uri          | 指示应用程序的回调URL，OpenID提供者将其带有授权代码的响应发送到此URL。 |
| code_challenge        | PKCE挑战码，由PKCE编码验证码派生                             |
| code_challenge_method | PKCE中指示生成挑战码的派生方法。 “S256” or “plain.”          |

<center>7-2 OIDC 认证请求参数</center>

身份验证请求中的response_type参数用于指示所需的OIDC流。可选参数response_mode控制将响应参数返回到应用程序的方法。除非另有说明，否则一般不指定response_mode，默认响应模式是使用查询参数或散列片段方式在重定向uri中将授权码、安全令牌等返回。

OAuth2.0中的Scope参数用于请求通过访问令牌授予API特权。对于OIDC身份验证请求，作用域用于指示OIDC的使用，并请求有关已验证用户信息的特定声明。OIDC身份验证请求必须包含“openid”，“profile”用于包含请求默认的用户配置文件声明的授权，例如name、family name和given name。“email”用于请求用户的电子邮件地址以及该地址是否已被验证。如果是通过访问令牌去请求向OpenID提供者的UserInfo端点，这些作用域将会限制应用请求返回用户信息时的权限，只会返回被授予的作用域范围内的声明。如果未颁发访问令牌，则请求的声明将包含在ID令牌中。有关请求声明的更多详细信息，请参见OpenID Connect Core规范的第5.4节和第5.5节[x](https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims)。

另一个称为“nonce”的可选参数很重要。如果请求ID令牌，则应包括nonce值。当应用程序向OpenID提供者发出身份验证请求时，它应该指定一个唯一的、不可猜测的nonce值，该值与应用程序为用户启动的会话相关联。一种方法是生成一个随机值，将其安全地存储在用户会话中，并使用其哈希作为nonce。当应用程序收到ID令牌时，它必须检查该令牌是否包含在身份验证请求中指定的确切nonce值，以及nonce是否与先前存储在会话中的值的哈希匹配。这将ID令牌与用户的应用程序会话链接起来，并降低了ID令牌被重放的风险。

有几个附加的可选参数可以在身份验证请求中传递，用于控制OpenID提供者如何以及是否提示用户进行身份验证和提供同意、指定首选语言、传递有关用户会话或标识符的提示，或者请求特定的用户声明。有关更多信息，请参阅OpenID核心规范XIII的第3.1.2.1节[xi](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest)。

#### 请求响应

OpenID提供程序返回对身份验证请求中指定的重定向URI的响应，该URI必须在OpenID提供者处注册。对于授权码流，默认响应模式使用查询参数将授权码返回到身份验证请求中指定的重定向URI（回调），同时返回在身份验证请求中传递的状态值。

```
HTTP/1.1 302 Found

Location: https://clientapplication.com/callback?

code=<authorization_code>

& state=<state_value>
```

应用程序应该检查响应是否包含任何错误代码，以及响应返回的状态值是否与其在身份验证请求中发送的状态值匹配。然后，它可以使用授权码发出令牌请求。每个授权码只能使用一次，因为如果已经使用了授权码，服务器响应报错。

#### 请求令牌

应用程序使用授权码向OpenID提供者的令牌端点请求令牌。下面的示例为非公共客户端使用其密钥，通过HTTP基本身份验证，向OpenID提供者的令牌端点请求令牌。

```
POST /token HTTP/1.1

Host: authorizationserver.com

Content-Type: application/x-www-form-urlencoded

Authorization: Basic <encoded client credentials>

grant_type=authorization_code

& code=<authorization_code>

& redirect_uri=<redirect_uri>

& code_verifier=<code_verifier>
```

应用程序使用授权码向OpenID提供者的令牌端点请求令牌时，需要进行客户端的认证，OIDC核心规范第9节[xii](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication)定义了身份验证方法的更多信息，具体方式可以参阅规范。令牌请求示例的参数如表7-3所示。



| 参数          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| grant_type    | 指示授权类型,指定为“code”用于指示该流程为授权码流程。        |
| code          | 授权响应中收到的授权码                                       |
| code_verifier | 为了派生代码挑战码而产生的PKCE的验证码值。它应该是一个长度介于43到128个字符（含）之间的不可使用的加密随机字符串，使用字符A–Z、A–Z、0–9、“-”、“.”、“”和“~” |
| redirect_uri  | 授权服务器响应的回调URI。 应该与授权请求中传递给授权端点的重定向uri的值相匹配 |

<center>7-3 令牌请求参数</center>

OpenID提供者将以JSON格式响应请求的令牌。下面是一个示例响应：

```
HTTP/ 1.1 200 OK

Content-Type: application/json;charset=UTF-8

Cache-Control: no-store

Pragma: no-cache

{

"id_token" : <id_token>,

"access_token" : <access_token value>,

"refresh_token" : <refresh_token value>,

"token_type" : "Bearer",

"expires_in" : <token lifetime>

}
```

响应包含的参数如表7-4所示

| 参数          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| id_token      | 包含用户信息声明的身份令牌                                   |
| access_token  | 用于调用User Endpoint API的访问令牌。                        |
| token_type    | 颁发的令牌类型。                                             |
| expires_in    | 令牌的有效期                                                 |
| refresh_token | 刷新令牌 是可选的。 是否返回刷新令牌由授权服务器自行决定。有关更多信息，请参阅本章后面的刷新令牌部分。 |

<center>7-4 令牌响应参数</center>

在信任ID令牌中的声明之前，应用程序应按照发布OpenID提供者提供的指导，JWT规范[xiii](https://datatracker.ietf.org/doc/html/rfc7519#section-7.2)中的验证步骤验证ID令牌。应用程序可以从ID令牌或通过使用访问令牌调用OpenID提供者的UserInfo端点来获取关于已验证用户的声明。



### OIDC隐式模式

OIDC中的隐式流程与OAuth2.0的隐式授权类型相似。
如第6章所述，不建议使用OAuth2.0隐式授权来获取访问令牌，至少在默认响应模式下是这样。然而，该指导原则的前提是基于暴露URL片段中的访问令牌的风险，即该URL片段可能通过浏览器历史记录或referer头泄漏。如果，应用程序只需要对用户进行身份验证并且可以通过ID令牌获取用户信息，而不需要访问令牌。在这种情况下，隐式流是可以接受的。图7-4显示了应用程序仅接收ID令牌的流程。

![7-4 OIDC implicit.jpg](https://i.loli.net/2021/07/16/ZDIVfbOosuFrktc.jpg)

<center>7-4 OIDC 隐式流程</center>

1. 用户向应用（RP）发起请求

2. 应用向OpenID提供者发出一个请求，发送一个基本的授权范围，该请求通过浏览器进行重定向。

3. OpenID提供者器接下来将向用户发起身份验证，和征求用户的授权同意范围。

4. 用户通过输入凭据进行身份验证，并且对请求的权限进行授权，该授权将影响令牌中包含影响作用域的信息。
5. OpenID提供者生成本次授权请求的身份令牌（IDToken），然后将此令牌包含在重定向请求中，通过浏览器将令牌返回给应用。
6. 应用程序从ID令牌获取关于用户身份的声明，并向用户显示需要展示的内容。

使用隐式流程对用户进行身份验证的（仅请求ID令牌）身份验证请求，如下：

```
GET /authorize? 

response_type=id_token

& client_id=<*client_id*>

& state=<*state_value*>

& nonce=<*nonce_value*>

& scope=<scope_value>

& redirect_uri=<callback_url> HTTP/1.1 

Host: authorizationserver.com 


```

隐式流身份验证请求中使用的参数，除了响应类型值意外，其他参数与表7-2所示相同的定义。对于隐式流，允许的响应类型值为：

* id_token--响应仅包含idtoken
* id_token token--响应包含id token和access token

默认情况下，隐式流使用URL片段通过前端浏览器交互将所有令牌返回到重定向URI。

> 注意

不建议在默认响应模式下使用“id_token token”的response_type，因为它会通过referer头或浏览器的历史记录使访问令牌置于暴露于潜在的风险。如果ID令牌不包含敏感数据，则使用默认响应模式和“id_token”则无此风险。

对于只需要ID令牌的应用程序，应考虑使用非默认de form_post响应模式，因为响应参数编码在post响应的主体中,这避免了通过一个URL片段公开ID令牌和数据，但是这种响应模式对于某些应用可能是不可行的。需要敏感的访问令牌和或ID令牌的公共客户端应该考虑使用PKCE授权码流来代替。

#### 响应

下面显示对隐式流程身份验证请求的示例响应，该请求使用“id_token”响应类型，仅请求id令牌。

```
HTTP/1.1 302 Found

Location: https://clientapplication.com/callback#

id_token=<id_token>

& state = <state>
```



### OIDC 混合流程

OIDC混合流包含授权代码流和隐式流程的元素。它是为既有安全后端，又有在前端执行JavaScript的客户端的应用程序而设计的。混合流支持诸如在对应用程序前端的前通道响应中返回ID令牌和授权码、让应用程序后端使用授权码从令牌端点获取访问令牌（和可选的刷新令牌）之类的模型。该流程如图7-5所示。

![7-5-HybridFlow.jpg](https://i.loli.net/2021/07/16/IqRwzpSukUD3hsT.jpg)

<center>7-5 OIDC 混合流程</center>



1. 用户向应用（RP）发起请求

2. 应用向OpenID提供者发出一个请求，发送一个基本的授权范围，该请求通过浏览器进行重定向。

3. OpenID提供者接下来将向用户发起身份验证，和征求用户的授权同意。

4. 用户通过输入凭据进行身份验证，并且对请求的权限进行授权，该授权将影响令牌中包含影响作用域的信息。
5. OpenID提供者生成本次授权请求的授权码和IDToken，然后将此授权码和IDToken包含在重定向请求中，通过浏览器将授权码和ID Token返回给应用。
6. 客户端验证ID Token，如果合法，应用后端使用授权码向OpenID提供者的令牌端点发送获取令牌请求，以获取访问令牌。
7. 授权服务器向该应用颁发授权令牌（AccessToken），刷新令牌（可选）。
8. 应用使用此令牌来访问用户信息。



身份验证请求的参数如表7-2所示，但混合流的响应类型使用三个不同的值来控制从OpenID提供者的授权端点返回响应中的令牌。可以通过对令牌端点的后续令牌请求来请求其他令牌。表7-5总结了响应类型的可能值。

| response_type       | 响应包含的类型                                               |
| ------------------- | ------------------------------------------------------------ |
| code id_token       | 返回code，id_token                                           |
| code token          | 返回code，访问令牌 access token——不建议使用默认的response_mode |
| code id_token token | 返回code，身份令牌 ID Token，访问令牌 access token——不建议使用默认的response_mode |

> 注意

不建议在默认响应模式下使用通过授权端点的前端通道响应返回访问令牌的响应类型，即“code token”和“code id token token”，因为访问令牌将在浏览器中作为URL片段公开，并且可能通过referer标头或浏览器历史中泄漏。

如果使用默认响应模式，则应使用“code id_token”响应类型，仅使用前通道响应将id令牌和授权码返回到浏览器。然后，通过应用程序后端的安全通道交互，可以从OpenID提供者的令牌端点获得访问令牌和可选刷新令牌。

在实际应用中，混合流的应用并不广泛。使用此流需要一个实现，该实现将同时提供前端和后端信息，例如nonce和state，用这些信息验证它们接收到的任何响应和安全令牌，并防止诸如CSRF或令牌注入之类的攻击。应用程序应该考虑使用具有PKCE的授权代码流，除非它们具有需要混合流的特定用例。



下面的示例中显示了使用混合流和“code id_token”响应类型和默认响应模式的示例身份验证请求，参数定义与前面的示例类似。

```
GET /authorize? 

response_type=code%20id_token

&client_id=<client_identifier>

&redirect_uri=<callback_url>

&scope=<scope_value>

&state=<state_value>

&nonce=<nonce_value> HTTP/1.1 

Host: authorizationserver.com 

```

当应用程序接收到响应时，应用程序后端可以使用授权码去请求更多的令牌，如前一节中对授权码流所述。

### 用户信息端点 Userinfo Endpoint

应用程序可以从OpenID提供者的UserInfo端点检索关于用户的声明。UserInfo端点是OAuth2.0 api端点，要调用它需要OpenID提供者发出的访问令牌。当请求访问令牌时，应用程序使用scope参数来指示所需的关于用户的声明。

对UserInfo端点的示例应用程序请求如下所示：

```
GET /userinfo HTTP/1.1

Host: authorizationserver.com

Authorization: Bearer <access_token>


```

OpenID提供者的UserInfo端点响应返回带有JSON对象的声明（除非使用签名或加密的响应）。下面的示例是请求范围为“openid profile email”响应。

```
HTTP/1.1 200 OK

Content-Type: application/json

{

"sub": "00u7474uxinMfMu3Q695",

"name": "yu yang",

"given_name": "yang",

"family_name": "yu",

"preferred_username": "yang.yu",

"email": "yang.yu@oneauth.cn", 

"email_verified": true, 

"picture":"https://oneauth.com/profile/yang.yu/yang.yu.jpg",

}


```



如果所需的用户配置文件声明对于通过URL片段返回的ID令牌来说太大，则使用UserInfo端点来返回用户的信息。

## 本章重点回顾

OpenID Connect协议在oauth2.0之上提供了一个身份层，它支持对应用程序的用户进行身份验证，并支持单点登录。在OAuth2.0中，OIDC添加了一个ID令牌和一个UserInfo端点，它向应用程序返回关于认证事件和认证用户的声明。

OIDC允许应用程序将用户身份验证委托给OpenID提供者，同时使用OIDC和OAuth2.0解决了身份验证和API授权问题。

* OIDC 提供了SSO单点登录功能。
* OIDC定义了一系列关于身份的声明。

## 参考：

i: https://openid.net/connect/

ii:https://datatracker.ietf.org/doc/html/rfc8252

iii:https://tools.ietf.org/html/rfc7519

iv:https://datatracker.ietf.org/doc/html/rfc7515

v:https://datatracker.ietf.org/doc/html/rfc7516

vi:https://openid.net/specs/openid-connect-core-1_0.html#IDToken

vii:https://datatracker.ietf.org/doc/html/draft-ietf-oauth-amr-values-04

viii:https://openid.net/specs/openid-connect-eap-acr-values-1_0.html

ix：https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims

x：https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims

xi:https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest

xii:https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication

xiii：https://datatracker.ietf.org/doc/html/rfc7519#section-7.2

