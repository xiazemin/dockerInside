# docker-compose

$ pip install -U docker-compose

Collecting docker-compose

Downloading [http://pypi.doubanio.com/packages/37/2b/f564105d548d8d92925aac550468b14282504e144d83b0c17139ce350fa3/docker\_compose-1.13.0-py2.py3-none-any.whl](http://pypi.doubanio.com/packages/37/2b/f564105d548d8d92925aac550468b14282504e144d83b0c17139ce350fa3/docker_compose-1.13.0-py2.py3-none-any.whl) \(94kB\)

$ docker-compose -v

docker-compose version 1.13.0, build 1719ceb

参考：[http://www.widuu.com/docker/compose/install.html](http://www.widuu.com/docker/compose/install.html)

使用Compose

使用Compose只需要简单的三个步骤：

首先，使用Dockerfile来定义你的应用环境：





FROM python:2.7

ADD ./code

WORKDIR /code

RUN pip install -r requirements.txt

1

2

3

4

FROM python:2.7

ADD ./code

WORKDIR /code

RUN pip install -r requirements.txt

其中，requirements.txt中的内容包括：





flask

redis

1

2

flask

redis

再用Python写一个简单的app.py





from flask importFlaskfrom redis importRedisimport os

app =Flask\(\_\_name\_\_\)

redis =Redis\(host='redis', port=6379\)@app.route\('/'\)def hello\(\):

    redis.incr\('hits'\)return'Hello World! I have been seen %s times.'% redis.get\('hits'\)if \_\_name\_\_ =="\_\_main\_\_":

    app.run\(host="0.0.0.0", debug=True\)

1

2

3

4

5

from flask importFlaskfrom redis importRedisimport os

app =Flask\(\_\_name\_\_\)

redis =Redis\(host='redis', port=6379\)@app.route\('/'\)def hello\(\):

    redis.incr\('hits'\)return'Hello World! I have been seen %s times.'% redis.get\('hits'\)if \_\_name\_\_ =="\_\_main\_\_":

    app.run\(host="0.0.0.0", debug=True\)

第二步，用一个compose.yaml来定义你的应用服务，他们可以把不同的服务生成不同的容器中组成你的应用。





web:

  build:.

  command: python app.py

  ports:

         - "5000:5000"

  volumes:

         - .:/code

  links:

         - redis

redis:

  image: redis

1

2

3

4

5

6

7

8

9

10

11

web:

  build:.

  command: python app.py

  ports:

         - "5000:5000"

  volumes:

         - .:/code

  links:

         - redis

redis:

  image: redis

第三步，执行docker-compose up来启动你的应用，它会根据compose.yaml的设置来pull/run这俩个容器，然后再启动。





Creating myapp\_redis\_1...

Creating myapp\_web\_1...

Building web...

Step 0 : FROM python:2.7

2.7: Pulling from python

...

Status: Downloaded newer image for python:2.7

 ---&gt; d833e0b23482

Step 1 : ADD . /code

 ---&gt; 1c04b1b15808

Removing intermediate container 9dab91b4410d

Step 2 : WORKDIR /code

 ---&gt; Running in f495a62feac9

 ---&gt; ffea89a7b090

Attaching to myapp\_redis\_1, myapp\_web\_1

......

redis\_1 \| \[1\] 17 May 10:42:38.147 \* The server is now ready to accept connections on port 6379

web\_1   \|  \* Running on http://0.0.0.0:5000/ \(Press CTRL+C to quit\)

web\_1   \|  \* Restarting with stat

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

Creating myapp\_redis\_1...

Creating myapp\_web\_1...

Building web...

Step 0 : FROM python:2.7

2.7: Pulling from python

...

Status: Downloaded newer image for python:2.7

 ---&gt; d833e0b23482

Step 1 : ADD . /code

 ---&gt; 1c04b1b15808

Removing intermediate container 9dab91b4410d

Step 2 : WORKDIR /code

 ---&gt; Running in f495a62feac9

 ---&gt; ffea89a7b090

Attaching to myapp\_redis\_1, myapp\_web\_1

......

redis\_1 \| \[1\] 17 May 10:42:38.147 \* The server is now ready to accept connections on port 6379

web\_1   \|  \* Running on http://0.0.0.0:5000/ \(Press CTRL+C to quit\)

web\_1   \|  \* Restarting with stat

3. Yaml文件参考

在上面的yaml文件中，我们可以看到compose文件的基本结构。首先是定义一个服务名，下面是yaml服务中的一些选项条目：

image:镜像的ID

build:直接从pwd的Dockerfile来build，而非通过image选项来pull

links：连接到那些容器。每个占一行，格式为SERVICE\[:ALIAS\],例如 – db\[:database\]

external\_links：连接到该compose.yaml文件之外的容器中，比如是提供共享或者通用服务的容器服务。格式同links

command：替换默认的command命令

ports: 导出端口。格式可以是：





ports:-"3000"-"8000:8000"-"127.0.0.1:8001:8001"

1

ports:-"3000"-"8000:8000"-"127.0.0.1:8001:8001"

expose：导出端口，但不映射到宿主机的端口上。它仅对links的容器开放。格式直接指定端口号即可。

volumes：加载路径作为卷，可以指定只读模式：





volumes:-/var/lib/mysql

 - cache/:/tmp/cache

 -~/configs:/etc/configs/:ro

1

2

3

volumes:-/var/lib/mysql

 - cache/:/tmp/cache

 -~/configs:/etc/configs/:ro

volumes\_from：加载其他容器或者服务的所有卷





environment:- RACK\_ENV=development

  - SESSION\_SECRET

1

2

environment:- RACK\_ENV=development

  - SESSION\_SECRET

env\_file：从一个文件中导入环境变量，文件的格式为RACK\_ENV=development

extends:扩展另一个服务，可以覆盖其中的一些选项。一个sample如下：





common.yml

webapp:

  build:./webapp

  environment:- DEBUG=false- SEND\_EMAILS=false

development.yml

web:extends:

    file: common.yml

    service: webapp

  ports:-"8000:8000"

  links:- db

  environment:- DEBUG=true

db:

  image: postgres

1

2

3

4

5

6

7

8

9

10

11

12

13

common.yml

webapp:

  build:./webapp

  environment:- DEBUG=false- SEND\_EMAILS=false

development.yml

web:extends:

    file: common.yml

    service: webapp

  ports:-"8000:8000"

  links:- db

  environment:- DEBUG=true

db:

  image: postgres

net：容器的网络模式，可以为”bridge”, “none”, “container:\[name or id\]”, “host”中的一个。

dns：可以设置一个或多个自定义的DNS地址。

dns\_search:可以设置一个或多个DNS的扫描域。

其他的working\_dir, entrypoint, user, hostname, domainname, mem\_limit, privileged, restart, stdin\_open, tty, cpu\_shares，和docker run命令是一样的，这些命令都是单行的命令。例如：





cpu\_shares:73

working\_dir:/code

entrypoint: /code/entrypoint.sh

user: postgresql

hostname: foo

domainname: foo.com

mem\_limit:1000000000

privileged:true

restart: always

stdin\_open:true

tty:true

1

2

3

4

5

6

7

8

9

10

11

cpu\_shares:73

working\_dir:/code

entrypoint: /code/entrypoint.sh

user: postgresql

hostname: foo

domainname: foo.com

mem\_limit:1000000000

privileged:true

restart: always

stdin\_open:true

tty:true

4. docker-compose常用命令

在第二节中的docker-compose up，这两个容器都是在前台运行的。我们可以指定-d命令以daemon的方式启动容器。除此之外，docker-compose还支持下面参数：

--verbose：输出详细信息

-f 制定一个非docker-compose.yml命名的yaml文件

-p 设置一个项目名称（默认是directory名）

docker-compose的动作包括：

build：构建服务

kill -s SIGINT：给服务发送特定的信号。

logs：输出日志

port：输出绑定的端口

ps：输出运行的容器

pull：pull服务的image

rm：删除停止的容器

run: 运行某个服务，例如docker-compose run web python manage.py shell

start：运行某个服务中存在的容器。

stop:停止某个服务中存在的容器。

up：create + run + attach容器到服务。

scale：设置服务运行的容器数量。例如：docker-compose scale web=2 worker=3

