#  Docker基本命令使用详解

1. 运行容器

$docker run -i -t ubuntu /bin/bash

-i 标志保证容器中的STDIN是开启的

-t 标志告诉Docker为要创建的容器分配一个伪tty终端

ubuntu 表示我们创建容器使用的镜像

/bin/bash 表示当容器创建完成之后，Docker就会执行容器中的/bin/bash命令



