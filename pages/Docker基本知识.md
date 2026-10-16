# 1.  安装Docker
- 使用官方安装脚本自动安装
  
  ```
  curl -fsSL https://get.docker.com -o get-docker.sh
  sudo sh get-docker.sh
  ```
- ## 1.1.  在debian下安装Docker
- 步骤 1：更新系统
- 首先，我们需要确保系统是最新的，以获取最新的软件包和安全更新。
  
  ```
  sudo apt update
  sudo apt upgrade
  ```
- 步骤 2：安装必要的软件包
- 我们需要安装一些软件包，以便apt可以通过HTTPS使用存储库。
  
  ```
  sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common
  ```
- 步骤 3：添加Docker的官方GPG密钥
- 为了验证从Docker下载的软件包的真实性，我们需要添加Docker的官方GPG密钥。
  
  ```
  curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  ```
- 步骤 4：添加Docker的APT存储库
- 现在，我们将添加Docker的APT存储库，以便apt可以从中获取Docker软件包。
  
  ```
  echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
- 步骤 5：更新apt存储库
- 我们需要更新apt存储库，以便它包含Docker存储库中的软件包。
  
  ```
  sudo apt update
  ```
- 步骤 6：安装Docker Engine
- 现在，我们可以使用apt安装Docker Engine。
  
  ```
  sudo apt install docker-ce docker-ce-cli containerd.io
  ```
- 步骤 7：验证安装
- 让我们验证一下Docker是否成功安装。
  
  ```
  sudo docker run hello-world
  ```
	- 这将下载一个简单的Docker映像并在容器中运行它。如果一切正常，您将看到一条消息，确认Docker已正确安装。
- 步骤 8：（可选）将用户添加到docker组
- 为了避免每次运行Docker命令时都需要使用sudo，您可以将当前用户添加到docker组中。
  
  ```
  sudo usermod -aG docker $USER
  ```
- 请注意，添加用户到docker组后，需要重新登录才能使更改生效。
- # 2.  使用Docker
- Docker 允许你在容器内运行应用程序， 使用 **docker run** 命令来在容器内运行一个应用程序。输出Hello world
  
  ```
  runoob@runoob:~$ docker run ubuntu:15.10 /bin/echo "Hello world"
  Hello world
  ```
- 各个参数解析：
	- **docker:** Docker 的二进制执行文件。
	- **run:** 与前面的 docker 组合来运行一个容器。
	- **ubuntu:15.10** 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
	- **/bin/echo "Hello world":**** **在启动的容器里执行的命令
- 以上命令完整的意思可以解释为：Docker 以 ubuntu15.10 镜像创建一个新容器，然后在容器里执行 bin/echo "Hello world"，然后输出结果。
- ## 2.1.  Docker容器的使用
- 通过docker stats --help命令可以看到docker的所有命令
- 获取镜像
  
  ```
  docker pull ubuntu
  ```
- 启动容器
  
  ```
  docker run -it ubuntu /bin/bash
  ```
	- 参数说明：
		- **-i**: 交互式操作。
		- **-t**: 终端。
		- **ubuntu**: ubuntu 镜像。
		- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
- 要退出终端，直接输入 **exit**:
- 停止容器
	- 首先查看所有容器
	  
	  ```
	  docker ps -a
	  ```
	- 使用 docker start 启动一个已停止的容器：
	  
	  ```
	  docker start b750bbbcfd88
	  ```
- 后台运行容器
  
  ```
  docker run -itd --name ubuntu-test ubuntu /bin/bash
  ```
- 停止一个容器
  
  ```
  docker stop <容器 ID>
  ```
- 停止的容器可以通过 docker restart 重启：
  
  ```
  docker restart <容器 ID>
  ```
- 进入容器
- 在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：
	- **docker attach**
	- **docker exec**：推荐大家使用 docker exec 命令，因为此命令会退出容器终端，但不会导致容器的停止。
	- docker attach命令
	  
	  ```
	  docker attach 1e560fca3906
	  ```
	- docker exec命令
	  
	  ```
	  docker exec -it 243c32535da7 /bin/bash
	  ```
- 导入和导出容器
	- 如果要导出本地某个容器，可以使用 **docker export** 命令。
	  
	  ```
	  docker export 1e560fca3906 > ubuntu.tar
	  ```
	- 可以使用 docker import 从容器快照文件中再导入为镜像，以下实例将快照文件 ubuntu.tar 导入到镜像 test/ubuntu:v1:
	  
	  ```
	  cat docker/ubuntu.tar | docker import - test/ubuntu:v1
	  ```
	- 此外，也可以通过指定 URL 或者某个目录来导入，例如：
	  
	  ```
	  docker import http://example.com/exampleimage.tgz example/imagerepo
	  ```
- 删除容器
	- 删除容器使用 **docker rm** 命令：
	  
	  ```
	  docker rm -f 1e560fca3906
	  ```
- ## 2.2.  Docker镜像的使用
- 当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。
- 可以使用 **docker images** 来列出本地主机上的镜像。
- 各个选项说明:
	- **REPOSITORY：**表示镜像的仓库源
	- **TAG：**镜像的标签
	- **IMAGE ID：**镜像ID
	- **CREATED：**镜像创建时间
	- **SIZE：**镜像大小
- 同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如 ubuntu 仓库源里，有 15.10、14.04 等多个不同的版本，那么可以使用 REPOSITORY:TAG 来定义不同的镜像。
- 可以使用如下命令来运行容器：
  
  ```
  runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash 
  root@d77ccb2e5cca:/#
  ```
- 参数说明：
	- **-i**: 交互式操作。
	- **-t**: 终端。
	- **ubuntu:15.10**: 这是指用 ubuntu 15.10 版本镜像为基础来启动容器。
	- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
- 获取镜像
  
  ```
  docker pull
  ```
- 查找镜像
	- 使用 docker search 命令来搜索镜像。比如我们需要一个 httpd 的镜像来作为我们的 web 服务。我们可以通过 docker search 命令搜索 httpd 来寻找适合我们的镜像。
	  
	  ```
	  docker search
	  ```
	- 具体参数
		- **NAME:**** **镜像仓库源的名称
		- **DESCRIPTION: **镜像的描述
		- **OFFICIAL: **是否 docker 官方发布
		- **stars: **类似 Github 里面的 star，表示点赞、喜欢的意思。
		- **AUTOMATED: **自动构建。
- 拖取镜像
	- 使用命令 docker pull 来下载镜像
- 删除镜像
	- 镜像删除使用 **docker rmi** 命令
- 创建镜像
	- 以通过以下两种方式对镜像进行更改。
		- 1、从已经创建的容器中更新镜像，并且提交这个镜像
		- 2、使用 Dockerfile 指令来创建一个新的镜像
	- 构建镜像
		- 使用命令 **docker build** ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。
		  
		  ```
		  runoob@runoob:~$ cat Dockerfile 
		  FROM    centos:6.7
		  MAINTAINER      Fisher "fisher@sudops.com"
		  
		  RUN     /bin/echo 'root:123456' |chpasswd
		  RUN     useradd runoob
		  RUN     /bin/echo 'runoob:123456' |chpasswd
		  RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
		  EXPOSE  22
		  EXPOSE  80
		  CMD     /usr/sbin/sshd -D
		  ```
		- 每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。第一条FROM，指定使用哪个镜像源。RUN 指令告诉docker 在镜像内执行命令。然后使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。
	- 设置镜像标签
		- 可以使用 docker tag 命令，为镜像添加一个新的标签。
- ## 2.3.  Docker容器的连接
- 容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 **-P** 或 **-p** 参数来指定端口映射。
- 网络端口映射
	- 两种方式的区别：
		- **-P :**是容器内部端口**随机**映射到主机的端口。
		- **-p : **是容器内部端口绑定到**指定**的主机端口。
	- **docker port** 命令可以让我们快捷地查看端口的绑定情况
- 容器互连
	- docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。
- 容器命名
	- 创建一个容器的时候，docker 会自动对它进行命名。另外，我们也可以使用 **--name** 标识来命名容器，例如：
	  
	  ```
	  runoob@runoob:~$  docker run -d -P --name runoob training/webapp python app.py
	  43780a6eabaaf14e590b6e849235c75f3012995403f97749775e38436db9a441
	  ```
	- 可以使用 **docker ps** 命令来查看容器名称。
- 新建网络
	- 创建一个新的 Docker 网络。
	  
	  ```
	  $ docker network create -d bridge test-net
	  ```
	- 参数说明：
		- **-d**：参数指定 Docker 网络类型，有 bridge、overlay。
- 连接容器
	- 运行一个容器并连接到新建的 test-net 网络:
	  
	  ```
	  $ docker run -itd --name test1 --network test-net ubuntu /bin/bash
	  ```
	- 打开新的终端，再运行一个容器并加入到 test-net 网络:
	  
	  ```
	  $ docker run -itd --name test2 --network test-net ubuntu /bin/bash
	  ```
- 配置DNS
	- 可以在宿主机的 /etc/docker/daemon.json 文件中增加以下内容来设置全部容器的 DNS：
	  
	  ```
	  {
	  "dns" : [
	   "114.114.114.114",
	   "8.8.8.8"
	  ]
	  }
	  ```
	- 设置后，启动容器的 DNS 会自动配置为 114.114.114.114 和 8.8.8.8。
	- 配置完，需要重启 docker 才能生效。
	- 查看容器的 DNS 是否生效可以使用以下命令，它会输出容器的 DNS 信息：
	  
	  ```
	  $ docker run -it --rm  ubuntu  cat etc/resolv.conf
	  ```
	- **手动指定容器的配置**
	- 只想在指定的容器设置 DNS，则可以使用以下命令：
	  
	  ```
	  $ docker run -it --rm -h host_ubuntu  --dns=114.114.114.114 --dns-search=test.com ubuntu
	  ```
	- 参数说明：
		- **--rm**：容器退出时自动清理容器内部的文件系统。
		- **-h HOSTNAME 或者 --hostname=HOSTNAME**： 设定容器的主机名，它会被写到容器内的 /etc/hostname 和 /etc/hosts。
		- **--dns=IP_ADDRESS**： 添加 DNS 服务器到容器的 /etc/resolv.conf 中，让容器用这个服务器来解析所有不在 /etc/hosts 中的主机名。
		- **--dns-search=DOMAIN**： 设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 host.example.com。
- ## 2.4.  Docker仓库的管理
- 仓库（Repository）是集中存放镜像的地方。当然不止 docker hub，只是远程的服务商不一样，操作都是一样的。
- 想要建立自己的docker仓的话，只需要从docker hub中拉一个对应的register镜像，然后将数据映射到外部宿主机就可以了。
- docker hub是Docker官方维护的一个公共仓库大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。
- 第一步，注册账号
- 第二步，登录和退出
	- 登录需要输入用户名和密码，登录成功后，就可以从 docker hub 上拉取自己账号下的全部镜像。
	  
	  ```
	  $ docker login
	  ```
	- 退出 docker hub 可以使用以下命令：
	  
	  ```
	  $ docker logout
	  ```
- 第三步，可以通过docker pull的命令来将其下载到本地
- 第四步，推送镜像
	- 用户登录后，可以通过 docker push 命令将自己的镜像推送到 Docker Hub。
	- 以下命令中的 username 请替换为你的 Docker 账号用户名。
	  
	  ```
	  $ docker tag ubuntu:18.04 username/ubuntu:18.04
	  $ docker image ls
	  
	  REPOSITORY      TAG        IMAGE ID            CREATED           ...  
	  ubuntu          18.04      275d79972a86        6 days ago        ...  
	  username/ubuntu 18.04      275d79972a86        6 days ago        ...  
	  $ docker push username/ubuntu:18.04
	  $ docker search username/ubuntu
	  
	  NAME             DESCRIPTION       STARS         OFFICIAL    AUTOMATED
	  username/ubuntu
	  ```
- ## 2.5.  Docker Dockerfile
- Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。
- 以定制一个nginx镜像为例讲解Dockerfile
	- 1.  在一个空目录下，新建一个名为 Dockerfile 文件，并在文件内添加以下内容：
	  
	  ```
	  FROM nginx
	  RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
	  ```
	- 2. FROM和RUN指令的作用
		- FROM：定制的镜像都是基于FROM的镜像，这里的nginx就是定制需要的基础镜像。
		- RUN：用于执行后面跟着的命令行命令。有两种方式：
			- shell格式
			  
			  ```
			  RUN <命令行命令>
			  # <命令行命令> 等同于，在终端操作的 shell 命令。
			  ```
			- exec格式
			  
			  ```
			  RUN ["可执行文件", "参数1", "参数2"]
			  # 例如：
			  # RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
			  ```
	- 注意：每执行一次RUN，就相当于会在镜像上执行一层，所以过多无意义的层，会造成镜像膨胀过大。
	- 3.开始构建镜像
		- 在 Dockerfile 文件的存放目录下，执行构建动作。
		- 以下示例，通过目录下的 Dockerfile 构建一个 nginx:v3（镜像名称:镜像标签）。
		  
		  ```
		  $ docker build -t nginx:v3 .
		  ```
	- 4. 上下文路径
		- 上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。
		- **解析**：由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。
		- 如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。
		- **注意**：上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢。
- ## 2.6.  Docker Compose
- ### 2.6.1.  Compose简介
- Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。
- Compose 使用的三个步骤：
	- 使用 Dockerfile 定义应用程序的环境。
	- 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
	- 最后，执行 docker-compose up 命令来启动并运行整个应用程序。
- docker-compose.yml的配置案例如下：
  
  ```
  # yaml 配置实例
  version: '3'
  services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
  volumes:
  logvolume01: {}
  ```
- ### 2.6.2.  Compose安装
- 运行以下命令以下载 Docker Compose 的当前稳定版本：
  
  ```
  $ sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  ```
- 将可执行权限应用于二进制文件：
  
  ```
  $ sudo chmod +x /usr/local/bin/docker-compose
  ```
- 创建软链：
  
  ```
  $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
  ```
- 测试是否安装成功
  
  ```
  $ docker-compose version
  cker-compose version 1.24.1, build 4667896b
  ```
- ### 2.6.3.  使用
- 准备：
  
  ```
  $ mkdir composetest
  $ cd composetest
  ```
- 创建Dockerfile文件：
	- 在composetest目录中，创建一个Dockerfile的文件，内容如下：
	  
	  ```
	  FROM python:3.7-alpine
	  WORKDIR /code
	  ENV FLASK_APP app.py
	  ENV FLASK_RUN_HOST 0.0.0.0
	  RUN apk add --no-cache gcc musl-dev linux-headers
	  COPY requirements.txt requirements.txt
	  RUN pip install -r requirements.txt
	  COPY . .
	  CMD ["flask", "run"]
	  ```
- **Dockerfile 内容解释：**
	- **FROM python:3.7-alpine**: 从 Python 3.7 映像开始构建镜像。
	- **WORKDIR /code**: 将工作目录设置为 /code。
	- 设置flask命令使用的环境变量
	  
	  ```
	  ENV FLASK_APP app.py
	  ENV FLASK_RUN_HOST 0.0.0.0
	  ```
	- **RUN apk add --no-cache gcc musl-dev linux-headers**: 安装 gcc，以便诸如 MarkupSafe 和 SQLAlchemy 之类的 Python 包可以编译加速。
	- 复制 requirements.txt 并安装 Python 依赖项。
	  
	  ```
	  COPY requirements.txt requirements.txt
	  RUN pip install -r requirements.txt
	  ```
	- **COPY . .**: 将 . 项目中的当前目录复制到 . 镜像中的工作目录。
	- **CMD ["flask", "run"]**: 容器提供默认的执行命令为：flask run。
- 创建docker-compose.yml
	- 在测试目录中创建一个名为 docker-compose.yml 的文件，然后粘贴以下内容：
	  
	  ```
	  # yaml 配置
	  version: '3'
	  services:
	  web:
	   build: .
	   ports:
	     - "5000:5000"
	  redis:
	   image: "redis:alpine"
	  ```
	- 该 Compose 文件定义了两个服务：web 和 redis。
		- **web**：该 web 服务使用从 Dockerfile 当前目录中构建的镜像。然后，它将容器和主机绑定到暴露的端口 5000。此示例服务使用 Flask Web 服务器的默认端口 5000 。
		- **redis**：该 redis 服务使用 Docker Hub 的公共 Redis 映像
- 使用Compose命令构建和运行应用
	- 在测试目录中，执行以下命令来启动应用程序：
	  
	  ```
	  docker-compose up
	  ```
	- 如果你想在后台执行该服务可以加上 **-d** 参数：
	  
	  ```
	  docker-compose up -d
	  ```
- ## 2.7.  Docker Machine
- Docker Machine 是一种可以让您在虚拟主机上安装 Docker 的工具，并可以使用 docker-machine 命令来管理主机。
- Docker Machine 也可以集中管理所有的 docker 主机
- 使用 docker-machine 命令，可以启动，检查，停止和重新启动托管主机，也可以升级 Docker 客户端和守护程序，以及配置 Docker 客户端与您的主机进行通信。
- ### 2.7.1.  安装
- 安装 Docker Machine 之前你需要先安装 Docker。
- 以下为在Linux上的安装命令
  
  ```
  $ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
  chmod +x /usr/local/bin/docker-machine
  ```
- ### 2.7.2.  使用
- 通过 virtualbox 来介绍 docker-machine 的使用方法。其他云服务商操作与此基本一致。
- 列出可用机器
	- 可以看到目前只有这里默认的 default 虚拟机。
	  
	  ```
	  $ docker-machine ls
	  ```
- 创建机器
	- 创建一台名为 test 的机器
	  
	  ```
	  $ docker-machine create --driver virtualbox test
	  ```
		- **--driver**：指定用来创建机器的驱动类型，这里是 virtualbox。
- 查看机器的ip
  
  ```
  $ docker-machine ip test
  ```
- 停止机器
  
  ```
  $ docker-machine stop test
  ```
- 启动机器
  
  ```
  $ docker-machine start test
  ```
- 进入机器
  
  ```
  $ docker-machine ssh test
  ```
- docker-machine 命令参数说明
	- **docker-machine active**：查看当前激活状态的 Docker 主机。
	- **config**：查看当前激活状态 Docker 主机的连接信息。
	- **create**：创建 Docker 主机
	- **env**：显示连接到某个主机需要的环境变量
	- **inspect**： 以 json 格式输出指定Docker的详细信息
	- **ip**： 获取指定 Docker 主机的地址
	- **kill**： 直接杀死指定的 Docker 主机
	- **ls**： 列出所有的管理主机
	- **provision**： 重新配置指定主机
	- **regenerate-certs**： 为某个主机重新生成 TLS 信息
	- **restart**： 重启指定的主机
	- **rm**： 删除某台 Docker 主机，对应的虚拟机也会被删除
	- **ssh**： 通过 SSH 连接到主机上，执行命令
	- **scp**： 在 Docker 主机之间以及 Docker 主机和本地主机之间通过 scp 远程复制数据
	- **mount**： 使用 SSHFS 从计算机装载或卸载目录
	- **start**： 启动一个指定的 Docker 主机，如果对象是个虚拟机，该虚拟机将被启动
	- **status**： 获取指定 Docker 主机的状态(包括：Running、Paused、Saved、Stopped、Stopping、Starting、Error)等
	- **stop**： 停止一个指定的 Docker 主机
	- **upgrade**： 将一个指定主机的 Docker 版本更新为最新
	- **url**： 获取指定 Docker 主机的监听 URL
	- **version**： 显示 Docker Machine 的版本或者主机 Docker 版本
	- **help**： 显示帮助信息
- ## 2.8.  Swarm集群管理
- Docker Swarm 是 Docker 的集群管理工具。它将 Docker 主机池转变为单个虚拟 Docker 主机。 Docker Swarm 提供了标准的 Docker API，所有任何已经与 Docker 守护程序通信的工具都可以使用 Swarm 轻松地扩展到多个主机。
- ### 2.8.1.  原理
- 如下图所示，swarm 集群由管理节点（manager）和工作节点（work node）构成。
	- **swarm mananger**：负责整个集群的管理工作包括集群配置、服务管理等所有跟集群有关的工作。
	- **work node**：即图中的 available node，主要负责运行相应的服务来执行任务（task）。
	  
	  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1697843374352-3987db1c-ebee-4e3d-bcb3-e44bd9c57a03.png)
- ### 2.8.2.  使用
- 创建swarm集群管理节点
	- 创建 docker 机器：
	  
	  ```
	  $ docker-machine create -d virtualbox swarm-manager
	  ```
	- 初始化 swarm 集群，进行初始化的这台机器，就是集群的管理节点。
	  
	  ```
	  $ docker-machine ssh swarm-manager
	  $ docker swarm init --advertise-addr 192.168.99.107 #这里的 IP 为创建机器时分配的 ip。
	  ```
	- 以上输出，证明已经初始化成功。需要把以下这行复制出来，在增加工作节点时会用到：
	  
	  ```
	  docker swarm join --token SWMTKN-1-4oogo9qziq768dma0uh3j0z0m5twlm10iynvz7ixza96k6jh9p-ajkb6w7qd06y1e33yrgko64sk 192.168.99.107:2377
	  ```
- 创建swarm集群工作节点
	- 这里直接创建好俩台机器，swarm-worker1 和 swarm-worker2
	- 分别进入两个机器里，指定添加至上一步中创建的集群，这里会用到上一步复制的内容。
- 查看集群信息
	- 进入管理节点，执行：docker info 可以查看当前集群的信息。
- 部署服务到集群中
	- **注意**：跟集群管理有关的任何操作，都是在管理节点上操作的。
	- 以下例子，在一个工作节点上创建一个名为 helloworld 的服务，这里是随机指派给一个工作节点：
	  
	  ```
	  docker@swarm-manager:~$ docker service create --replicas 1 --name helloworld alpine ping docker.com
	  ```
- 查看服务部署情况
	- 查看 helloworld 服务运行在哪个节点上，可以看到目前是在 swarm-worker1 节点：
	  
	  ```
	  docker@swarm-manager:~$ docker service ps helloworld
	  ```
	- 查看 helloworld 部署的具体信息：
	  
	  ```
	  docker@swarm-manager:~$ docker service inspect --pretty helloworld
	  ```
- 扩展集群服务
	- 将上述的 helloworld 服务扩展到俩个节点。
	  
	  ```
	  docker@swarm-manager:~$ docker service scale helloworld=2
	  ```
- 删除服务
  
  ```
  docker@swarm-manager:~$ docker service rm helloworld
  ```
- 滚动升级服务
  
  ```
  docker@swarm-manager:~$ docker service create --replicas 1 --name redis --update-delay 10s redis:3.0.6
  ```
	- 滚动升级 redis
	  
	  ```
	  docker@swarm-manager:~$ docker service update --image redis:3.0.7 redis
	  ```
- 停止某个节点接收新的任务
	- 查看所有的节点
	  
	  ```
	  docker@swarm-manager:~$ docker node ls
	  ```
	- 可以看到目前所有的节点都是 Active, 可以接收新的任务分配。