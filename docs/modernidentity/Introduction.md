# 引言



云计算、移动互联网、IoT、5G 等一批新技术不断采用，线上的业务与日俱增。另一方面，网络安全的形式也愈加紧迫。

网络安全工作人员缺口巨大且不断扩大，根据网络安全劳动力研究报告，全球网络安全人才缺口已扩大至近 300 万。其中，亚太地区缺口最高，达到了214 万。在需要安全保护的在线服务和设备数量激增的今天，网络安全面临的严峻风险令人震惊。为了填补这一空白，除了通过高校和社会机构培养更多的人才以外。必须鼓励更多的人了解这一领域，包括非安全专业的技术人员，例如开发者，鼓励他们更多的理解安全，并为他们提供足够的学习资源，使他们能够在开发阶段有效地选择安全的架构和安全的开发模式。

其中，身份管理是安全的一个重要组成部分，对于保护正在创建的快速扩展的创新在线服务、智能设备、机器人程序、自动化应用等至关重要。正如这些新技术的采用，通过身份来进行安全的访问控制已经成为一种趋势，如今，很多业内人员将这一新的以身份为中心的安全架构称之为“零信任”。这一新的安全架构的技术革命正在发生，并被越来越多的企业首席安全官（CSO ）们认可。

在过去，往往身份的安全会被忽略，或者随机的选择一种可行的方式，而没有慎重的选择适合身份安全架构。国内的安全人员对于身份的安全也愈加重视，但相对其他安全的书籍，身份安全方向的专业的书籍相对缺乏，没有专业的书籍去系统地去详细介绍身份的方方面面。本书将结合我们近几年对于身份的研究和经验，基于一些标准和最佳实践给予读者更多参考，以便于在后续的工作中决策。

我们将提供一个详细的去介绍三种身份管理协议，即OIDC、oauth2.0和saml2.0，这对于应用程序认证设计或者api的授权的设计提供非常大的帮助。为了让开发者更加深入的去理解这三种协议的用例和优势，每个章节会详细讨论每个协议需要解决的问题、如何设计，以及如何解决问题。

除了三个基本的协议，我们还提供了有关典型身份管理要求的需求和设计章节，以帮助您在设计一个身份系统时，更好的确定项目计划和范围：包括身份设计需要考虑的规划、可能出错的内容、常见错误，以及如何接近合规性。这些章节对开发人员、架构师、技术项目经理以及参与应用程序开发项目的安全团队成员都很有价值。

因此，这本书所有的章节都将聚焦在身份管理的方方面面。为了更好的理解协议，本书还将详细的向介绍身份发展的历史、基础的协议、在不同的场景下如何去落地，如何通过身份驱动系统的安全和系统架构的健康。我们将介绍如何使用三种身份协议解决在创建应用程序时遇到的身份验证和授权的常见用例。即便如此，身份安全覆盖的细节也是极其多，一本书没有办法来涵盖每一个协议的所有案例或协议的每一个细微差别，我们也不能涵盖协议规范中的每一个细节，这些都需要在实践中结合协议去做更多的了解。

为了更好的对一些概念进行说明，更进一步对于一些难以理解的概念进行阐述，我们也会附带了一个示例程序。

通过本书，我们也期望因此书与更多的开发者结缘，了解开发者的需求，并帮助大家在云和企业环境中设计和部署身份和访问管理系统。这里会涉及到各种各样的场景，包括不同类型的应用程序如何选择集成的方式，单点登录、联合的身份认证、不同的访问控制和授权模型，以及身份目录的管理和各种形式的强身份认证。

期望通过本书的对于身份的概述，能够帮助你开始理解身份安全，为你提供足够的背景知识，以为你更全面地了解身份安全做好铺垫。

对于身份安全，涵盖的内容如此之广，也期望读者对我们遗漏的内容、理解偏差的内容以提供更正和反馈。我们将该书发布在docs.oneauth.com，同时可通过https://github.com/OneAuth2/Oneauth 访问，后续发现的任何错误欢迎大家反馈，对于一些错误的更正，也都将在该书的OneAuth GitHub repo中注明。

我们希望这本书和示例代码对您有用，并祝您的应用程序项目好运和安全。

