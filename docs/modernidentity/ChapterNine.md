# 授权与访问策略执行

前面的第五章介绍了API访问的安全，以及第六章和第七章分别介绍了用户授权、OIDC的协议，那么二者如何结合，共同保障应用、API和数据的安全，本章节将介绍如何通过授权来保障API访问策略的执行。

## 授权和访问策略的关系

授权与访问策略执行，共同的保障了正确的（人）实体能够访问到正确的资源方面，而非法的实体或非法的请求则被拒绝。为了更好的实现这一目标，我们从逻辑上进行了授权与访问策略执行的分离。即授权（通过某种形式，如Token/Ticket）授予某一实体一定的权限，访问策略执行来检查实体持有响应的票据（Token/Tiket）的合法性来进行相应的允许或者拒绝访问被保护的资源。

举个例子来说，我们在入住酒店的时候，会在前台通过身份证的验证获取到房卡（类似于token），这个房卡里包含了你可以进入的房间，可以消费的类型（餐厅、健身房）等权利。房间、餐厅、健身房的刷卡机会检查该房卡是否能够进入进行消费。

授权是在消费之间就已经确定的，在前台登记的那时，有前台人员录入相应的权限。访问的策略执行点一般在受保护资源内或最近的执行点进行执行，以免被绕过。

如图8-1所示,对授权和访问策略的的执行进行了逻辑上的分离。

![9-1 AuthZ and Police Enforcement.jpg](https://i.loli.net/2021/07/20/XDnzPT7K6ptyY48.jpg)



## 控制级别

授权和访问策略强制可分别指定和应用于不同的级别：

* 级别1—实体是否可以访问应用程序或API
* 级别2–实体可以在应用程序或API中使用哪些功能
* 级别3—实体可以访问或操作的数据



### 级别1-应用或API

在最高级别，授权和访问策略实施可以控制实体是否有权访问应用程序或API。这种用例经常出现在公司环境中。如8-1所示，运营团队中的员工可能没有访问公财务系统的权限。这一级别的策略实施可以在应用程序内处理，也可以由应用程序前面的实体处理，

可以在应用程序外部通过组件（如身份和授权系统或与访问策略执行系统一起工作的反向代理）执行强制的策略。这样的架构可以作为一个高级执行点来转移那些根本没有权限访问应用程序的用户。类似的方法可以用于API，在这两种情况下都有助于减少目标系统上的策略执行工作量。

### 级别2-功能级别

功能级授权和访问策略实施控制实体在应用程序或API中可以做什么。例如，财务部门的初级会计职员可能可以访问公司会计系统并输入个别日记账分录，但不能执行月末结账。这种级别的授权和访问策略实施往往是特定于应用程序的。它可以利用存储在身份和授权系统中的用户信息，例如身份系统中的角色或组，但通常在应用程序或API的逻辑中强制执行。

### 第3级-数据访问
第三级授权和访问政策执行规定了对特定数据子集的访问。如果功能级访问策略强制定义了实体可以执行的功能，则数据级访问策略将进一步限制对特定数据的访问。例如，在运营系统的订单应用中，可以在功能级别授权角色或组为“区域销售经理”的用户查看销售订单，但数据级访问策略将其限制在用户身份属性中的“销售区域”属性中所示的特定区域。数据级访问可以在应用程序或API内或底层存储层中强制执行。

## 用户实体和应用实体的授权

接下来，我们将介绍两种需要授权和访问策略实施的场景。
第一个场景是关于如何控制用户（或实体）在应用程序中做什么，第二个是如何控制应用程序可以请求什么API。

用户需要授权才能在应用程序中执行各种功能，应用程序可以根据用户的授权权限呈现应用程序的用户界面，这样就不会显示用户不能使用的功能。此外，当用户发出请求时，应用程序后端或API必须在执行请求之前检查用户是否具有请求所需的授权。

应用程序需要授权才能调用受保护的API。如果API上的内容归应用程序的用户所有，则访问需要用户的授权。这种情况经常出现在面向消费者的应用程序中。如果应用程序以自己的名义访问API中的内容，则授权服务器将根据授权服务器管理员先前配置的权限向应用程序授予授权。

无论被授权的实体是什么，控制访问通常涉及三个步骤：
•授权和访问策略的规范定义。
•向执行点提供授权信息（如果需要）
•执行点执行访问策略

### 用户授权

授权的规范一直依赖是一个复杂的问题。通常，访问策略与一组与用户或应用程序相关联的属性表示。下面将会介绍如何使用OIDC和SAML2.0将用户的授权信息传递给应用程序，如何使用OAuth2.0去进行授权访问APIs，以及这如何访问策略的实施。用于向用户传递授权的属性可能会有所不同，但最常见的可分为两类。

#### 用户身份属性

用户的身份可以根据在基于角色的访问控制（RBAC）模型中用户分配的角色、访问控制列表（ACL）中的组或成员，或基于属性的访问控制（ABAC）模型的用户的属性来授予授权。这些属性是相对静态的因素，它们保持不变，而不管用户的位置、设备等动态的信息。

授予用户权限的授权步骤通常在用户在应用程序中发出请求之前完成。例如，如果新员工加入公司的财务团队，企业可以通过在新员工第一天在企业标识系统中分配用户角色来授权该员工访问其财务应用程序。对于面向消费者的应用程序，当用户购买服务的特定订阅级别时，可以向其分配与访问相关的用户配置文件属性。

#### 用户事务属性

授权还可以基于在认证或访问受保护资源过程中的上下文的事务。这些因素可以包括用户的地理位置、用户是否在公司防火墙内部或外部、或者用户的设备是否被认证为遵守某些安全配置标准，访问的时间，验证使用的身份验证机制的强度因素等等。这些因素是在身份验证时动态获取的，而不是用户身份配置文件的一部分。如果这些因素被身份提供者获取，就可以安全令牌中的声明的形式提供给应用程序。

#### 传递

对于使用OIDC的应用程序，用户授权信息可以作为ID令牌中的声明或来自OIDC提供者的UserInfo端点的响应传递给应用程序。使用SAML2.0的应用程序可以通过SAML2.0断言中的属性语句接收信息。用户配置文件信息（例如用户的角色、组或购买的订阅级别）以及因素（例如用户的IP地址或身份验证方法的强度）可以通过这种方式传递到应用程序。然后，应用程序可以使用这些信息执行访问策略强制。

图8-2显示了通过ID令牌传递用户配置文件信息以支持访问强制的示例。在该示例中，该应用程序是一个视频应用程序，用户可以购买不同的订阅级别（如普通会员、高级会员）以观看不同的视频。ID令牌将用户购买的订阅级别传递给应用程序。

![9-2 example for Authz Delvery .jpg](https://i.loli.net/2021/07/20/W91QBsgYuM42dRS.jpg)

<center>9-2授权传递</center>

1.用户重定向到在OIDC OpenID提供程序处登录。
2.ID令牌包括用户购买的订阅信息。
3.ID Token中的订阅数据用于确定显示的电影列表。
4.用户选择要观看的电影。
5.应用程序后端检查用户所选电影的订阅级别。



在该示例中，ID令牌中关于用户的订阅级别的信息用于向用户显示他们有权观看的电影。它还用于强制执行访问策略。尽管前端将电影列表限制在用户可以观看的范围内，但恶意用户可能会找到绕过此限制的方法，因此访问策略强制检查是在应用程序后端进行的，无法绕过此检查。本例使用OIDC，但是使用SAML2.0的应用程序可以遵循类似的模型，从SAML2.0断言中获取授权数据。



#### 策略执行

在信任令牌声明的信息之前，需要验证令牌的有效性，声明的是未经篡改的真实性，一般流程如下：

1. 验证ID令牌是格式正确的JWT（JSON Web令牌）
2. 验证ID令牌上的签名
3. 检查令牌是否未过期
4. 检查颁发者是否是正确的OpenID提供者
5. 检查应用程序中令牌的目标受众是否相符

请务必查看OpenID提供程序的文档，以了解实现它们所需的确切验证步骤。一旦ID令牌被验证，应用程序就可以使用令牌中的声明来执行访问策略。



### 应用授权

授权和策略实施的第二种情况是应用程序调用API。

#### 应用属性

代表用户调用API的应用程序请求由用户授权，代表自己调用API的请求由授权服务器基于配置的策略授权。此策略通常是在应用程序调用API之前配置的。除非应用程序或API的数量很大，否则策略规范通常通过指示授权调用特定API的特定应用程序以及可以访问哪些端点和操作来表示。如果使用OAuth2.0，策略可以按作用域来指定，例如“get:documents”

##### 授权

OAuth2.0授权将其请求的作用域指定为授权请求的参数。例如，应用程序请求OpenID提供者的UserInfo端点的访问令牌来检索用户属性，可能会使用“scope=OpenID profile email”。

作用域扮演着非常重要的角色。如果没有声明，作用域只是一个字符串列表，其中包含作用域标记或作用域名称。示例：scope=“doc_read doc_write openid email”。

将作用域scope参数看作访问范围是种不错的方式，也就是说，它列出了客户需要访问的内容，可以用于粗粒度授权，以决定是否允许客户机查询API。但是，即使令牌中存在doc_read的作用域，我们也不知道应该允许它读取哪个文档（doc）。为了进一步说明，以OpenID Connect为例，它将作用域定义为一组声明的集合。因此，“email” Scope令牌定义为“email”和“email is verified”声明。声明具有与其关联的值，而作用域标记只是一个名称，这意味着作用域只是一组声明的打包，例如OIDC中的Profile，具有如下的结构：

![9-3 OIDC Profile Token.png](https://i.loli.net/2021/07/20/q7OI4NyCDo18seE.png)

<center>9-3 OIDC Profile</center>

事实上每个OpenID声明都与一个作用域相关联，如下图所示。

![9-4 OIDC Scope.png](https://i.loli.net/2021/07/20/2irTDLGOSZXUMo4.png)

<center>9-3 OIDC Scopes</center>

这种定义的方式可以推广到任意作用域和声明，当涉及API访问时，这将变得非常有用。

使用授权代码和隐式授权类型，请求的API资源归用户所有，因此OAuth2.0授权服务器将提示用户，显示请求的作用域，并在发出访问令牌之前获得用户对应用程序请求的同意。通过授予客户机凭据，请求的API资源归应用程序所有，授予每个应用程序的权限在授权服务器（或从中访问的策略服务器）中指定，并且在发出访问令牌之前，它将使用该信息来决定是否允许请求。

#### 传递

不管受保护的资源是由用户还是由应用程序拥有，如果请求已被授权，授权服务器将为请求的API向应用程序颁发访问令牌。授权服务器可以允许向访问令牌添加额外的自定义声明，例如关于用户的声明，包括关于特权（例如角色）的数据。附加声明可能对强制访问的API有用。通过用户的自定义声明和属性对扩展性的支持可能因授权服务器实现而异。

#### 策略执行

API必须验证访问令牌并执行访问策略，以确保在响应之前允许应用程序的请求。验证和获取有关访问令牌的信息的步骤将因授权服务器对访问令牌的实现而异。您应该查看授权服务方文档，以了解有关如何验证它发出的任何令牌的详细信息。一旦验证了访问令牌，API就可以使用令牌中的信息（如果允许的话，包括任何自定义声明）在响应请求之前执行其访问策略强制。



## 本章重点回顾

授权涉及特权的授予，而访问策略强制是在请求资源时进行的检查，以验证请求者是否已被授予请求所需的特权。授权和访问策略实施可用于控制用户在应用程序中可以做什么，以及应用程序是否可以向API发出请求。用户授权可基于用户配置文件中的静态因素和/或身份验证时评估的动态因素，两者都可以通过安全令牌传递给应用程序。应用程序授权可以基于用户或授权服务器批准的作用域，并在传递给应用程序并用于调用API的访问令牌中显示。在下一章中，我们将讨论一个示例应用程序，以及它如何使用OIDC和OAuth 2.0来授权对API的访问。



