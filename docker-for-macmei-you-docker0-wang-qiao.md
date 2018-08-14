Docker For Mac没有docker0网桥

在使用Docker时，要注意平台之间实现的差异性，如Docker For Mac的实现和标准Docker规范有区别，Docker For Mac的Docker Daemon是运行于虚拟机\(xhyve\)中的, 而不是像Linux上那样作为进程运行于宿主机，因此Docker For Mac没有docker0网桥，不能实现host网络模式，host模式会使Container复用Daemon的网络栈\(在xhyve虚拟机中\)，而不是与Host主机网络栈，这样虽然其它容器仍然可通过xhyve网络栈进行交互，但却不是用的Host上的端口\(在Host上无法访问\)。bridge网络模式 -p 参数不受此影响，它能正常打开Host上的端口并映射到Container的对应Port。文档在这一点上并没有充分说明，容易踩坑。

