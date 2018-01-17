---
layout:     post
title:      "[Develop] Microservices[你了解微服务吗?]"
subtitle:   "A definition of this new architectural term"
date:       2018-01-17 09:59:59
author:     "Elve Xu"
header-img: "img/container_monitoring.png"
catalog: true
tags:
    - Microservices
    - Netflix
    - Spring Cloud
---

> Code is Not Only Code .

##Microservices[微服务]
> 新的架构术语的定义
![](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/microservices.png)
过去几年来，“微服务体系结构”（microservice architecture）这个术语出现了，它描述了一种将软件应用程序设计成可独立部署的服务套件的特殊方式。 虽然这种架构风格没有确切的定义，但围绕业务能力，自动化部署，端点智能以及对语言和数据的分散控制等方面，组织具有一些共同特征。


-------
“微服务” - 在软件架构拥挤的街道上又一个新名词。 虽然我们自然的倾向是以轻蔑的眼光来传递这样的东西，但这个术语描述了一种我们发现越来越吸引人的软件系统风格。 我们已经看到许多项目在过去几年中都使用这种风格，迄今为止的结果都是积极的，因此对于我们的许多同事来说，这正成为构建企业应用程序的默认风格。 可悲的是，没有太多的信息概括了微服务的风格，以及如何去做。

-------
简而言之，微服务架构风格[1]是一种将单个应用程序作为一套小型服务开发的方法，每种应用程序都在自己的进程中运行，并与轻量级机制（通常是HTTP资源API）进行通信。 这些服务是围绕业务功能构建的，可以通过全自动部署机制独立部署。 这些服务的集中管理最少，可以用不同的编程语言编写，并使用不同的数据存储技术。

-------
要开始解释微服务风格，将其与单一风格进行比较是非常有用的：构建为单一单元的单一应用程序。 企业应用程序通常建立在三个主要部分：一个客户端用户界面（由在用户的机器上的浏览器中运行的HTML页面和JavaScript组成）数据库（由许多表格组成，这些表格被插入一个通用的关系数据库管理 系统）和一个服务器端应用程序。 服务器端应用程序将处理HTTP请求，执行域逻辑，从数据库检索和更新数据，选择并填充要发送到浏览器的HTML视图。 这个服务器端应用程序是一个单一的逻辑可执行文件[2]。 对系统的任何更改都涉及构建和部署新版本的服务器端应用程序。

这样一个庞大的服务器是构建这样一个系统的自然方法。 处理请求的所有逻辑都在一个进程中运行，允许您使用语言的基本功能将应用程序划分为类，函数和名称空间。 仔细一点，您可以在开发人员的笔记本电脑上运行和测试应用程序，并使用部署管道来确保更改已经过正确测试并部署到生产环境中。 您可以通过在负载平衡器后面运行多个实例来水平缩放整体。


-------
单一应用程序可能会成功，但是越来越多的人感到沮丧，特别是随着越来越多的应用程序被部署到云中。 更改周期是连在一起的 - 对应用程序的一小部分进行更改，需要重新整理整个整体。 随着时间的推移，通常很难保持良好的模块化结构，使得难以保持仅应该影响该模块中的一个模块的变化。 缩放需要整个应用程序的缩放，而不是需要更多资源的部分。

![](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/microservices/sketch.png)
>Figure 1: Monoliths and Microservices

这些挫折导致了微服务架构风格：构建应用程序作为服务套件。 除了服务可独立部署和扩展之外，每个服务还提供了一个牢固的模块边界，甚至允许用不同的编程语言编写不同的服务。 他们也可以由不同的团队管理。

我们并不认为微服务的风格是新颖的或创新的，它的根源至少可以追溯到Unix的设计原则。 但是我们确实认为没有足够多的人考虑微服务架构，如果他们使用了许多软件开发将会更好。


##微服务架构的特点(Characteristics)
>我们不能说微服务架构风格有一个正式的定义，但是我们可以试图描述我们认为符合标签的架构的共同特征。 就像任何概括共同特征的定义一样，并不是所有的微服务体系结构都具有所有的特征，但我们确实期望大多数微服务体系结构展现出最多的特征。 虽然我们的作者是这个相当松散的社区的积极成员，但我们的目的是试图描述我们在自己的工作中所看到的以及我们所知道的团队的类似努力。 特别是我们并没有制定一些符合的定义。

####Componentization via Services

只要我们涉足软件行业，就一直希望通过将组件整合在一起来构建系统，而这正是我们在现实世界中看到事物的方式。在过去的几十年中，我们已经看到相当大的进步，大部分语言平台的共同图书馆的大纲。

当谈到组件时，我们遇到了构成组件的困难定义。我们的定义是，一个组件是一个可独立替换和升级的软件单元。

微服务架构将使用库，但他们自己的软件组件化的主要方式是分解成服务。我们将库定义为链接到程序中的组件，并使用内存中的函数调用进行调用，而服务是与Web服务请求或远程过程调用等机制进行通信的进程外组件。 （这与许多面向对象程序中服务对象的概念不同[3]）。

使用服务作为组件（而不是库）的一个主要原因是服务是可独立部署的。如果在单个进程中有一个由多个库组成的应用程序[4]，则对任何单个组件的更改都会导致必须重新部署整个应用程序。但是，如果将该应用程序分解为多个服务，则可能会期望许多单个服务更改只需要重新部署该服务。这不是绝对的，有些改变会改变服务接口，导致一些协调，但是一个好的微服务架构的目标是通过服务合约中的内聚服务边界和进化机制来最小化这些服务接口。

使用服务作为组件的另一个结果是更明确的组件接口。大多数语言没有一个好的机制来定义一个明确的发布接口。通常只有文档和规则可以防止客户端破坏组件的封装，导致组件之间过于紧密的耦合。通过使用显式远程调用机制，服务可以更容易地避免这种情况。

使用这样的服务确实有缺点。远程调用比进程内调用更加昂贵，因此远程API需要更粗粒度，这通常更难以使用。如果您需要更改组件之间的责任分配，那么当您跨越流程边界时，这样的行为动作就很难做到。

首先，我们可以观察到服务映射到运行时进程，但这只是第一个近似值。一个服务可能由多个进程组成，这些进程总是被开发和部署在一起，例如一个应用程序进程和一个只被该服务使用的数据库。

####Organized around Business Capabilities
在将大型应用程序拆分为多个部分时，管理层往往将注意力集中在技术层面，导致UI团队，服务器端逻辑团队和数据库团队。 当团队沿着这些路线分开时，即使是简单的改变也会导致跨团队项目需要时间和预算批准。 一个聪明的团队将围绕这一点进行优化，并为两个弊端中的较小者提供丰富的内容 - 只需将逻辑强制到他们可以访问的任何应用程序。 换句话说到处都是逻辑。 这是**Conway's Law**[5]的一个例子。

> Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.
> -- Melvyn Conway, 1967

![](https://github.com/misselvexu/misselvexu.github.io/blob/master/img/in-post/microservices/conways-law.png)
>Figure 2: Conway's Law in action

微服务的方式是不同的，分解成围绕业务能力组织的服务。 这种服务需要对该业务领域的软件进行广泛的实施，包括用户界面，持久性存储以及任何外部协作。 因此，团队是跨职能的，包括开发所需的全部技能：用户体验，数据库和项目管理。
![](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/microservices/PreferFunctionalStaffOrganization.png)
>Figure 3: Service boundaries reinforced by team boundaries

团队负责构建和操作每个产品，每个产品被拆分成通过消息总线进行通信的多个单独服务。

大型单一应用程序总是可以围绕业务功能模块化，尽管这不是常见的情况。 当然，我们会敦促一个庞大的团队建立一个庞大的应用程序，沿着业务线分工。 我们在这里看到的主要问题是，它们倾向于围绕太多的背景进行组织。 如果巨石跨越了许多这样的模块化边界，那么一个团队的个别成员就很难把他们融入他们的短期记忆中。 另外我们看到模块化的生产线需要执行很多纪律。 服务组件所必需的更明确的分离使得清晰地保持团队边界更容易。

>**How big is a microservice?**
>
>Although “microservice” has become a popular name for this architectural style, its name does lead to an unfortunate focus on the size of service, and arguments about what constitutes “micro”. In our conversations with microservice practitioners, we see a range of sizes of services. The largest sizes reported follow Amazon's notion of the Two Pizza Team (i.e. the whole team can be fed by two pizzas), meaning no more than a dozen people. On the smaller size scale we've seen setups where a team of half-a-dozen would support half-a-dozen services.
This leads to the question of whether there are sufficiently large differences within this size range that the service-per-dozen-people and service-per-person sizes shouldn't be lumped under one microservices label. At the moment we think it's better to group them together, but it's certainly possible that we'll change our mind as we explore this style further.


####Products not Projects

我们所看到的大多数应用程序开发工作都使用了一个项目模型：目标是提供一些被认为完成的软件。完成后，软件被移交给维护组织，建立它的项目组被解散。

微服务支持者倾向于避免这种模式，而更倾向于团队应该在整个生命周期内拥有一个产品的概念。对此，一个共同的启发就是亚马逊的“你自己构建，运行”的概念，即开发团队负责软件的生产。这使开发人员日常接触到他们的软件在生产中的行为，并增加与用户的联系，因为他们必须承担至少一些支持负担。

产品的心态，与商业能力的联系。软件作为一套完整的功能而不是将软件视为一个持续的关系，问题是软件如何帮助用户提高业务能力。

没有理由为什么这种单一的应用程序不能采用同样的方法，但是较小的服务粒度可以更容易地在服务开发人员和用户之间建立个人关系。

####Smart endpoints and dumb pipes

>当在不同流程之间建立沟通结构时，我们看到许多产品和方法强调将重要的智慧带入沟通机制本身。 企业服务总线（ESB）就是一个很好的例子，ESB产品通常包括用于消息路由，编排，转换和应用业务规则的复杂设施。

微服务社区支持另一种方法：smart endpoints and dumb pipes。 从微服务构建的应用程序的目标是尽可能脱离和内聚 - 它们拥有自己的领域逻辑，在传统的Unix意义上更像过滤器 - 接收请求，适当地应用逻辑并产生响应。 这些是使用简单的REST协议而不是复杂的协议（例如WS-Choreography或BPEL）或者通过中央工具编排来编排的。

最常用的两种协议是使用资源API和轻量级消息传递的HTTP请求响应[8]。 第一个最好的表达是
>Be of the web, not behind the web
-- Ian Robinson

微服务团队使用万维网建立的原则和协议。开发人员或操作人员可以通过很少的努力来缓存经常使用的资源。

常用的第二种方法是通过轻量级消息总线进行消息传递。所选择的基础架构通常是愚蠢的（仅仅是作为一个消息路由器而言是愚蠢的） - 像RabbitMQ或ZeroMQ这样的简单实现并不仅仅是提供可靠的异步结构 - 智能仍然存在于正在生成的端点消费信息;在服务。

在整体中，组件正在执行中，它们之间的通信是通过方法调用或函数调用进行的。将巨人变成微服务的最大问题在于改变沟通模式。从内存中方法调用到RPC的天真转换会导致无法正常工作的健谈通信。相反，你需要用更粗糙的方法来取代细粒度的通信。    

>**Microservices and SOA**
>When we've talked about microservices a common question is whether this is just Service Oriented Architecture (SOA) that we saw a decade ago. There is merit to this point, because the microservice style is very similar to what some advocates of SOA have been in favor of. The problem, however, is that SOA means too many different things, and that most of the time that we come across something called "SOA" it's significantly different to the style we're describing here, usually due to a focus on ESBs used to integrate monolithic applications.
>
In particular we have seen so many botched implementations of service orientation - from the tendency to hide complexity away in ESB's [6], to failed multi-year initiatives that cost millions and deliver no value, to centralised governance models that actively inhibit change, that it is sometimes difficult to see past these problems.
>
Certainly, many of the techniques in use in the microservice community have grown from the experiences of developers integrating services in large organisations. The Tolerant Reader pattern is an example of this. Efforts to use the web have contributed, using simple protocols is another approach derived from these experiences - a reaction away from central standards that have reached a complexity that is, frankly, breathtaking. (Any time you need an ontology to manage your ontologies you know you are in deep trouble.)
>
This common manifestation of SOA has led some microservice advocates to reject the SOA label entirely, although others consider microservices to be one form of SOA [7], perhaps service orientation done right. Either way, the fact that SOA means such different things means it's valuable to have a term that more crisply defines this architectural style.
>


####Decentralized Governance

集中治理的后果之一是在单一技术平台上实现标准化的趋势。经验表明，这种方法是狭隘的 - 不是每个问题都是钉子，而不是每一个解决方案。我们更喜欢使用正确的工具来工作，而单一的应用程序可以在一定程度上利用不同的语言，这是不常见的。

将整体零件拆分成服务时，我们在构建每个服务时都有选择。你想使用Node.js站立一个简单的报告页面？去吧。 C ++的一个特别粗糙的接近实时的组件？精细。你想交换一个不同的数据库风格，更适合一个组件的读取行为？我们有技术来重建他。

当然，仅仅因为你可以做点什么，并不意味着你应该这样做，但是用这种方式划分你的系统意味着你可以选择。

构建微服务的团队也更喜欢不同的标准方法。他们并不是使用在纸上写下来的一套定义的标准，而是倾向于生产有用的工具，其他开发人员可以用它来解决他们所面临的类似问题。这些工具通常是从实施中收获，并与更广泛的群体共享，有时，但不是专门使用内部开源模型。既然git和github已经成为了事实上的版本控制系统，开源实践正变得越来越普遍。

Netflix是遵循这一理念的组织的一个很好的例子。作为图书馆，鼓励其他开发者以类似的方式共享有用的，尤其是经过战斗测试的代码，并鼓励其他开发者解决类似的问题，但是如果需要，还可以选择不同的方法。共享库往往侧重于数据存储，进程间通信以及我们下面进一步讨论的基础设施自动化的常见问题。

对于微服务社区来说，管理费用特别缺乏吸引力。这并不是说社区不重视服务合同。恰恰相反，因为往往有更多的人。只是他们正在考虑管理这些合同的不同方式。容忍阅读器和消费者驱动合同等模式通常应用于微服务。这些援助服务合同独立发展。执行消费者驱动的合同作为您的构建的一部分增加了信心，并提供有关您的服务是否运作的快速反馈。事实上，我们知道澳大利亚的一个团队正在推动以消费者为导向的合同来建立新的服务。他们使用简单的工具，允许他们定义服务的合同。在成为新服务的代码之前，这成为自动构建的一部分。然后，这个服务就被建立起来，直到它满足合同为止，这是一个在构建新软件时避免“YAGNI”难题的优雅方法。这些技术和工具在其周围成长，通过减少服务之间的时间耦合来限制中央合同管理的需要。

也许分散治理的最高点是由亚马逊推广的构建它/运行它的精神。 团队负责构建软件的各个方面，包括全天候运行软件。 这种责任水平的转变绝对不是常态，但我们看到越来越多的公司将责任推给开发团队。 Netflix是另一个采用这种风格的组织[11]。 每天早上3点被你的传呼机唤醒，当你写代码的时候，你肯定会把注意力集中在质量上。 这些想法与传统的集中治理模式相距甚远。

####Decentralized Data Management
数据管理的分权化以多种不同的方式呈现出来。 在最抽象的层面上，这意味着世界的概念模型将在不同系统之间有所不同。 当整合大型企业时，这是一个常见问题，客户的销售观点将与支持观点不同。 在销售视图中称为客户的某些内容在支持视图中可能完全不显示。 那些做的可能有不同的属性和（更差的）共同的属性与微妙的不同的语义。

这个问题在应用程序之间是常见的，但是也可以在应用程序中发生，特别是当应用程序被分成不同的组件时。一个有用的思考方式是有界上下文的Domain-Driven Design概念。 DDD将一个复杂的领域划分为多个有界的上下文，并绘制出他们之间的关系。这个过程对整体和微服务体系结构都是有用的，但是服务和上下文边界之间有一个自然的相关性，这有助于澄清，正如我们在业务能力部分所描述的，加强了分离。

除了分散关于概念模型的决定之外，微服务还分散了数据存储决策。虽然整体式应用程序更喜欢使用单一逻辑数据库来存储持久性数据，但企业通常更倾向于在一系列应用程序中使用单一数据库 - 其中许多决策都是通过供应商的商业模式来实现的。微服务更喜欢让每个服务管理自己的数据库，不管是同一个数据库技术的不同实例，还是完全不同的数据库系统 - 一种叫做Polyglot Persistence的方法。您可以在整体中使用多边形持久性，但是对于微服务来说，它会更频繁地出现。

![](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/microservices/decentralised-data.png)

在微服务中分散数据的责任对管理更新有影响。处理更新的通用方法是在更新多个资源时使用事务来保证一致性。这种方法经常在整体中使用。

使用像这样的事务有助于保持一致性，但是会施加显着的时间耦合，这在多个服务中是有问题的。分布式事务是非常难以实现的，因此微服务架构强调服务之间的无交易协调，明确认识到一致性可能只是最终的一致性，而问题是通过补偿操作来处理的。

以这种方式选择管理不一致对于许多开发团队来说是一个新的挑战，但却是经常与商业实践相匹配的挑战。企业往往处理一定程度的不一致性，以便快速响应需求，同时采取某种逆转过程来处理错误。只要纠正错误的成本低于失败的成本，这种折衷是值得的。

####Infrastructure Automation
基础设施自动化技术在过去几年中发生了巨大的变化 - 特别是云和AWS的发展降低了构建，部署和运行微服务的操作复杂性。

许多正在用微服务构建的产品或系统正在由具有持续交付丰富经验的团队构建，这是前驱，持续集成。 以这种方式构建软件的团队广泛使用基础设施自动化技术。 这在下面显示的构建流水线中进行了说明。
![](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/microservices/basic-pipeline.png)
>Figure 5: basic build pipeline

由于这不是一篇关于持续交付的文章，我们将在这里提请注意几个关键特性。 我们希望尽可能多的信心，我们的软件工作，所以我们运行大量的自动化测试。 推动工作软件“向上”流水线意味着我们将自动化部署到每个新的环境。

一个单一的应用程序将被建立，测试，并推动通过这些环境相当愉快。 事实证明，一旦你投入了自动化的整体生产路径，那么部署更多的应用程序似乎不再那么可怕。 请记住，CD的目标之一就是使部署变得无聊，所以无论是它的一个还是三个应用程序，只要它仍然无聊就无所谓了[12]。

我们看到团队使用广泛的基础设施自动化的另一个领域是管理生产中的微服务。 与我们上面的说法相反，只要部署是无聊的，巨石和微服务之间没有太大的区别，每一个的操作环境可能会有惊人的不同。

![](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/microservices/micro-deployment.png)
>Figure 6: Module deployment often differs

####Design for failure
将服务用作组件的后果是，应用程序需要设计成能够容忍服务的失败。 任何服务电话都可能由于供应商不可用而失败，客户必须尽可能优雅地回应。 与单片设计相比，这是一个缺点，因为它引入了额外的复杂性来处理它。 其结果是微服务团队不断反思服务失败如何影响用户体验。 Netflix's Simian Army在工作日期间引发服务甚至数据中心的故障，以测试应用程序的弹性和监控。

这种生产中的自动化测试足以让大多数运营团队通常在一个星期之前的工作中感到不适。这并不是说单一的架构风格不能够完善的监控设置 - 这在我们的经验中不太常见。

由于服务可能随时出现故障，因此能够快速检测到故障并在可能的情况下自动恢复服务很重要。微服务应用程序非常重视对应用程序的实时监控，同时检查架构元素（数据库每秒获得多少请求）和业务相关指标（例如每分钟接收多少订单）。语义监控可以提供一个预警系统，引发开发团队跟踪调查。

这对微服务体系结构来说尤其重要，因为微服务对编排和事件协作的偏好导致了紧急行为。尽管许多专家赞扬偶然出现的价值，事实上，紧急行为有时可能是一件坏事。监控对迅速发现不良应急行为至关重要，因此可以进行修复。

微服务团队希望看到每个单独服务的复杂的监控和日志记录设置，例如仪表板显示上/下状态以及各种运营和业务相关指标。 关于断路器状态，当前吞吐量和延迟的细节是我们经常遇到的其他例子。

####Evolutionary Design
微服务从业者通常来自进化设计背景，并将服务分解视为进一步的工具，以使应用程序开发人员能够控制其应用程序中的更改，而不会减慢更改速度。变更控制并不一定意味着降低变更 - 使用正确的态度和工具，您可以对软件进行频繁，快速和控制良好的变更。

每当你试图将一个软件系统分解成组件时，你都面临着如何分割这些组件的决定 - 我们决定分割我们的应用程序的原则是什么？组件的关键属性是独立替换和可升级性的概念[13] - 这意味着我们寻找可以想象重写组件而不影响其协作者的点。事实上，许多微服务集团通过明确预计许多服务将被废弃而不是长远发展，从而进一步推动了这一进程。

“卫报”网站就是一个很好的例子，这个应用程序是作为一个整体设计和构建的，但是在微服务方面一直在发展。庞然大物仍然是网站的核心，但他们更喜欢通过构建使用庞然大物API的微服务来添加新功能。这种方法对于固有的临时性功能特别有用，例如专门处理体育赛事的页面。这样一个网站的一部分，可以迅速使用快速的开发语言放在一起，一旦事件结束，删除。我们在一家金融机构看到了类似的方法，在这个机构中，为了获得市场机会而增加新的服务，并在几个月甚至几个星期后被抛弃。

这种对可替换性的强调是模块化设计的一个更普遍的原则的特例，即通过变化模式驱动模块化[14]。你想在同一个模块中同时改变事物。很少变化的系统的部分应当与正在经历大量流失的服务不同。如果你发现自己一再改变两项服务，这是一个迹象，他们应该合并。

将组件投入到服务中为更细化的发布计划增添了机会。对于整体来说，任何更改都需要完整构建和部署整个应用程序。但是，使用微服务，您只需重新部署您修改的服务。这可以简化并加速发布过程。缺点是你不得不担心改变一个服务打破了消费者。传统的集成方法是尝试使用版本控制来解决这个问题，但微服务世界中的偏好是仅使用版本控制作为最后的手段。我们可以通过设计服务尽可能地容忍供应商的变化来避免大量的版本控制。


-------
##微服务是未来吗？
写这篇文章的主要目的是解释微服务的主要思想和原理。通过花时间做到这一点，我们清楚地认为，微服务架构风格是一个重要的思想 - 值得认真考虑的企业应用。我们最近建立了一些使用风格的系统，并且知道其他使用过这种方法的人。

那些我们知道谁在某种程度上开拓了架构风格的人包括Amazon, Netflix, The Guardian, the UK Government Digital Service, realestate.com.au, 2013年的会议电路充满了一些公司正在转向微型服务的例子 - 包括Travis CI。另外还有很多组织一直在做我们想做微服务的类，但没有使用过这个名字。 （通常这被称为SOA--尽管如我们所说，SOA有许多相互矛盾的形式[15]）。

尽管有这些积极的经验，但是我们并不认为我们确信微服务是软件架构的未来方向。虽然我们迄今为止的经验与整体应用相比是积极的，但我们意识到，我们没有足够的时间来做出充分的判断。

通常你的架构决策的真正后果只有在你做出它们几年之后才会显现出来。我们已经看到一个项目，一个拥有强烈的模块性愿望的好团队已经建立了一个多年来一直衰败的单一架构。很多人认为，微服务的这种衰减是不太可能的，因为服务边界是明确的，很难修补。然而，直到我们看到足够的年龄足够的系统，我们不能真正评估微服务架构如何成熟。

为什么人们可能会期望微服务不成熟呢？在组件化的任何努力中，成功取决于软件如何适应组件。很难弄清楚组件边界应该在哪里。进化设计意识到获取边界的困难，因此重构它们很容易。但是，当你的组件是远程通信服务的时候，重构要比在进程内库困难得多。在服务边界上移动代码是很困难的，任何接口变化都需要在参与者之间进行协调，需要增加向后兼容层，测试变得更加复杂。

另一个问题是，如果组件不干净地组成，那么你所做的就是将组件内部的复杂性转移到组件之间的连接上。这不仅仅是把复杂性转移到一个地方，而是把它转移到一个不那么明确，难以控制的地方。当你在看一个小而简单的组件时，很容易想到事情会变得更好，同时又错过了服务之间的混乱连接。

最后，还有团队技巧的因素。新技术往往被更多的技巧团队采用。但是对于一支技能更强的球队来说更有效的技术并不一定适合技能较差的球队。我们已经看到很多不熟练的团队构建混乱的单片架构，但是当微服务发生这种混乱时，需要一些时间来看看会发生什么。一个糟糕的团队总是会造成一个糟糕的系统 - 很难判断微服务是否能减少这种情况，或者使情况恶化。

我们听到的一个合理的论点是，你不应该从微服务架构开始。取而代之的是一块巨石，保持它的模块化，一旦巨石成为一个问题，就把它分成微型服务。 （尽管这个建议并不理想，因为一个好的进程内接口通常不是一个好的服务接口。）

所以我们谨慎乐观地写这篇文章。到目前为止，我们已经足够了解微服务的风格，觉得这是一个有价值的道路。我们不能确定我们最终会在哪里，但软件开发的挑战之一是，您只能根据您目前需要处理的不完善信息做出决定。

>Power By : [martinfowler.com](https://martinfowler.com/articles/microservices.html)


