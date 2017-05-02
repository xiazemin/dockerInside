# Docker工具

软件项目的成功常常根据其催生的生态系统来衡量。围绕或基于核心技术构建的项目增添了功能和易用性，它们常常日臻完善。Docker就是一个典例，这个软件容器化系统让IT部门可以专注于应用程序、而不是虚拟机，作为生产的标准单位。

Docker一向备受第一方和第三方开发人员的关注，而众多项目扩展、补充或改进Docker，却没有成为Docker的一部分。下面是如今正在开发中的10个最知名的项目，从长远来看，有些项目有机会成为Docker的一部分。

Kubernetes



谈论第三方Docker项目自然少不了提到Kubernetes，这是谷歌开发的一款开源Docker管理工具，用于跨计算机集群部署容器。除了通过让集群上部署的容器保持均衡，从而有助于管理Docker节点的工作负载外，Kubernetes还提供了让容器可以彼此联系的方法，不需要开启网络端口或执行其他操作。这些功能，加上Kubernetes用Go编写的事实（Docker也用这种语言编写），强烈表明它在未来某个时间会并入到Docker。

项目：Kubernetes

GitHub：https://github.com/GoogleCloudPlatform/kubernetes

Dockersh



如果你想让用户可以访问外壳（shell），可是对由此带来的安全后果有顾虑，Dockersh提供了一种Docker化的方式，为外壳会话提供高于平均水平的安全性。

Dockersh让多个用户可以连接到某个主机，每个用户都运行自行选择的单独的Docker容器所生成的外壳。用户可以查看其主目录，并对主目录进行永久性更改，但他们只能看到自己的进程，而且只能使用自己的专用网络堆栈。开发者担心Dockersh里面的潜在安全漏洞，不建议它用于不受限制的公众访问，至少在Docker以这种方式加以改进之前不建议这么做。而光这个概念就让这个项目值得关注。

项目：Dockersh

GitHub：https://github.com/Yelp/dockersh

DockerUI



虽然大多数开发人员和管理人员通过命令行来创建及运行Docker容器，但Docker的Remote API让他们可以通过充分利用REST（代表性状态传输协议）的API，运行相同的命令。这时，DockerUI有了用武之地。这个Web前端程序让你可以处理通常通过Web浏览器的命令行来管理的许多任务。某一个主机上的所有容器都可以通过仅仅一条连接来处理，该项目几乎没有任何依赖关系。不过，它仍在大力开发之中，但是它采用麻省理工学院（MIT）许可证，所以可以免费地重复使用。此外，它不包含任何内置的身份验证或安全机制，所以务必将任何公之于众的DockerUI连接放在用密码来保护的系统后面。

项目：DockerUI

GitHub：https://github.com/crosbymichael/dockerui

Shipyard



Shipyard使用Citadel集群管理工具包，简化对横跨多个主机的Docker容器集群进行管理。通过Web用户界面，你可以大致浏览相关信息，比如你的容器在使用多少处理器和内存资源、在运行哪些容器，还可以检查所有集群上的事件日志。包含完整的API和命令行接口（CLI），而专门构建的Docker镜像（又叫扩展镜像）可用来扩展Shipyard的功能。这后一个想法仍在开发之路，不过可以通过Interlock项目，获得负载均衡/路由镜像。

项目：Shipyard

GitHub：https://github.com/shipyard/shipyard

Kitematic



许多项目旨在让Docker成为基于OS X的编程员们手里一款实用的桌面环境开发工具，而Kitematic正是其中之一。它简化了下载Docker镜像、启动这些镜像以及管理它们的过程，让这项任务变得如同在VMware Workstation等应用程序中使用虚拟机一样简单。同一类别的其他项目包括：DVM、Docker OS X和OS X Installer，不过Kitematic很可能是这批项目中最完善的。唯一的重大缺点是，卸载过程有点错综复杂。

项目：Kitematic

GitHub：https://github.com/kitematic/kitematic

Logspout



Docker还没有提供一种方法来管理在Docker容器里面运行的程序所生成的日志。Logspout是一个Docker容器，大小仅14MB，使用BusyBox作为其核心，它可以将来自容器应用程序的日志发送到某一个中央位置，比如单一JSON对象或者通过HTTP API可获得的流式端点。就挖掘的信息方面而言，Logspout目前功能有限，因为它只能实现容器的标准输出（stdout）和标准错误输出（stderr），不过已计划一旦Docker提供相关钩子（hook），就允许更全面的日志功能。将来应密切关注这个项目。

项目：Logspout

GitHub：https://github.com/progrium/logspout

Autodock



Docker自动化工具可以说是个大众化产品。毕竟，更容易自动化不是Docker的全部意义吗？但Autodock却凭借几个不同之处脱颖而出。它被设计成可在使用Salt和SaltStack作为主要自动化技术的环境中运行，它还经过了专门的设计，通过确定某一个Docker集群中哪些服务器拥有的负载最小，以便尽快启用新容器。一个可能存在的缺点是，让它发挥功效需要好多基本组件（SaltStack、Golang、Etcd和Python）。

项目：Autodock

GitHub：https://github.com/cholcombe973/autodock

DIND（Docker-in-Docker）



Docker-in-Docker正如其名：这是让你可以在Docker容器里面运行Docker的一种方式，在Docker 6.0中实现的方式是，为容器添加特权模式。

抛开噱头和笑话不说，如果你想把Docker本身作为一项服务提供给Docker容器，这个工具很有用――比如说，如果你想试用某种自动化工具或方法。请注意，Docker的“内部”实例是最新的Docker二进制代码，构建时可以从docker.io来获取。另外牢记一点：以这种方式运行的实例是在特权模式下运行的；正因为如此，你将它们暴露在非Docker化的外界面前时，需要采取更多的防范措施。

项目：Docker-in-Docker

GitHub：https://github.com/jpetazzo/dind

Heroku-Docker



Heroku曾是一种支持多种语言的出色的平台即服务（PaaS），如今在一定程度上仍然是这样，但Docker让我们几乎可以在任何地方从事类似PaaS的工作。为此，对那些想方设法将现有的Heroku项目迁移到Docker，又无须从头开始重新构建的人来说，这是个不二的选择。这个简单的小项目拿来现有的Heroku应用程序后，可以从命令行将其转换成Docker镜像，执行整个操作只需要几个命令就行。

项目：Heroku-Docker

GitHub：https://github.com/ddollar/heroku-docker

Docker Node Tester



当你使用某一项最热门的新IT技术作为另一项热门的新IT技术的测试机制时，会使用什么？显然是Docker Node Tester。DNT提供了一个测试平台，Node.js项目针对Docker容器中多个版本的Node.js运行，然后以表格方式输出结果。你还可以针对最前沿版本的Node进行测试，无论是什么版本。请注意，不同版本的Node都是从源代码构建的，这意味着你最后会得到Node整个源代码树的本地副本；确保你有足够的空间来存储它。

项目：Docker Node Tester

GitHub：https://github.com/rvagg/dnt

