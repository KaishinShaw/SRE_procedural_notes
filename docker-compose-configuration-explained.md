# Docker Compose 配置文件 docker-compose.yml 详解

Docker Compose配置文件是Docker Compose的核心，用于定义服务、网络和数据卷。格式为YAML，默认路径为./docker-compose.yml，可以使用.yml或.yaml扩展名，目前Compose配置文件格式的最新版本为V3。Compose配置文件中涉及的配置项也比较多，但大部分配置项的含义跟docker run命令相关选项是类似的。

本文主要参考官方文档对目前最新的V3版Compose配置文件进行一个总结。都是一些概念性的内容，不涉及具体操作。

------------------------------------------------------------------------

**一、Compose配置文件版本**

这里主要对Compose配置文件的版本的相关要点进行一个简单的总结。至于每个版本具体的变化和升级信息可以参考官方的Compose配置文件版本与升级指南。

------------------------------------------------------------------------

**1.Compose配置文件格式的版本概述**

当前有三种版本的Compose配置文件格式： - **Version 1**

旧版格式，通过省略YAML的根配置项version来指定。

未声明版本的Compose配置文件都被视为V1版，所有的服务都作为根选项在Compose配置文件中声明。

支持V1的Compose最高到1.6.x，再高版本的Compose不推荐使用V1版Compose配置文件。

不支持数据卷、网络和构建参数配置。

V1的Compose不会利用网络优势，每个容器都位于默认的bridge网络上，并且可以从其他容器的IP地址访问，需要使用links来启用容器之间的发现。 - **Version 2.x**

通过YAML的根配置项version来指定，具体配置如version: '2'或version: '2.1'等。

必须在Compose配置文件根选项指定版本号，并且主版本数字为2，且所有服务必须在services配置项下声明。

1.6.0+版本的Compose都支持V2，Docker Engine的版本需要1.10.0+版本。

支持数据卷和网络的配置。

默认情况下，每个容器都加入了应用范围的默认网络，并且可以在与服务名称相同的主机名下发现。很大程度上links不是必要的。

V2中加入了环境变量替换。 - **Version 3.x**

最新版本，也是推荐使用版本，推出该版的目的是为了在Compose和Docker Engine的swarm模式之间形成交叉兼容。

通过YAML的根配置项version来指定，具体配置如version: '3'或version: '3.1'等。

V3删除了多个配置项，但也新增了更多配置项。

关于Compose配置文件版本的常见注意事项：

**在声明V2和V3版本时需注意：**

在指定Compose配置文件要使用的版本时，需同时指定主版本数字和次版本数字。如果未给定次版本数字，则默认使用0而不是最新版本，因此将不支持再更高版本中才加入的新功能。比如version: '3'，使用的是3.0版本而不是目前最新的3.8版本。

在使用多Compose配置文件时需注意：

使用多个Compose配置文件扩展服务时，每个文件必须为相同的版本。

**2. Compose配置文件格式版本与Docker的兼容性关系**

Compose配置文件格式具有多种版本。其中Compose配置文件格式版本与Docker的兼容性关系如下表所示：

Compose配置文件格式版本 Docker Engine 版本

```         
3.8 19.03.0+
3.7 18.06.0+
3.6 18.02.0+
3.5 17.12.0+
3.4 17.09.0+
3.3 17.06.0+
3.2 17.04.0+
3.1 1.13.1+
3.0 1.13.0+
2.4 17.12.0+
2.3 17.06.0+
2.2 1.13.0+
2.1 1.12.0+
2.0 1.10.0+
1.0 1.9.1.+
```

如果使用的较旧版本的Docker，可以参考官方的Compose版本发布列表。其中的每组发行说明都详细说明了支持的Docker Engine版本和兼容的Compose配置文件格式版本。

**3. 兼容模式**

在1.20.0版本，Compose在docker-compose命令中引入了一个新的选项--compatibility，目的在于帮助开发人员更轻松地过渡到V3版。启用该选项后，docker-compose命令会读取每个服务定义的deploy部分，并尝试将其转换为等效的V2配置项。目前，以下deploy下的配置项已被转换：

resources下的limits和reservations下的memory

replicas

restart_policy下的 condition 和max_attempts

所有的其他配置项都将被忽略，如果这些被忽略的配置项存在则会发出一个警告。可以使用带--compatibility的Config命令查看将用于deploy的配置。

注意请勿在生成环境使用兼容模式！

建议不要在生产环境中使用--compatibility选项。由于使用非Swarm模式属性生成的配置仅是近似值，因此可能会产生意外的结果。

**二、Compose配置文件结构**

Docker Compose配置文件是一个用于定义服务、网络和数据卷的YAML文件。其中服务定义了该服务启动的每个容器的配置，就像将命令行参数传递给docker run一样，网络和数据卷的定义类似于docker network create和docker volume create。跟docker run一样，如果在Dockerfile中通过诸如CMD、EXPOSE、VOLUME和ENV这些指令指定了相关选项，那么在默认情况下，不需要在docker-compose.yml中再次指定它们。下面是从官网引过来的一个Compose配置文件的示例，可以先大致了解一下它的结构：

```         
version: "3.8"
services:
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints:
          - "node.role==manager"
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints:
          - "node.role==manager"

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints:
          - "node.role==manager"

networks:
  frontend:
  backend:

volumes:
  db-data:
```

顶层的version、services、networks和volumes将Compose配置文件分为四个部分，其中version指定Compose配置文件的版本，services定义服务，networks定义网络，volumes定义数据卷。

**三、服务配置**

服务定义了该服务启动的每个容器的配置，就像将命令行参数传递给docker run一样。比如以下配置：

```         
services:
  redis:
    image: redis:alpine
```

services下的redis是用户自定义的服务名称，redis下的image只是众多服务配置项中的其中一个，意思是指定镜像名称或id。下面就对服务的相关配置项进行一个总结。

**1. build**

在构建时应用的配置项。一般直接指定Dockerfile所在文件夹路径，可以是绝对路径，或者相对于Compose配置文件的路径。可以指定为包含构建上下文（context）路径的字符串。例如：

```         
version: "3.8"
services:
  webapp:
    build: ./dir
```

也可以使用context指定上下文路径，使用dockerfile基于上下文路径指定Dockerfile文件，使用args指定构建参数。例如：

```         
version: "3.8"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

如果同时指定了build和image。例如：

```         
build: ./dir
image: webapp:tag
```

Compose会在./dir目录下构建一个名为webapp，标签为tag的镜像。

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，build配置项被忽略。因为docker stack命令不会在部署之前构建镜像。

**(1) context**

指定包含Dockerfile的目录路径或git仓库url。该目录是发送给Docker守护进程（Daemon）的构建上下文（context）。当配置的值是相对路径时，它将被解释为相对于Compose配置文件的路径。例如：

```         
build:
  context: ./dir
```

指定上下文为Compose配置文件目录下的dir目录。

**(2) dockerfile**

指定Dockerfile文件。Compose会使用指定的Dockerfile文件构建镜像，但必须要指定构建上下文路径。例如：

```         
build:
  context: .
  dockerfile: Dockerfile-alternate
```

Compose会使用Compose配置文件所在目录下名为Dockerfile-alternate的Dockerfile文件构建镜像。

**(3) args**

添加构建参数，这些只能在构建过程中访问的环境变量。首先在Dockerfile文件中指定参数：

```         
ARG buildno
ARG gitcommithash


RUN echo "Build number: $buildno"
RUN echo "Based on commit: $gitcommithash"
```

然后build中指定参数，以下两种写法都可以：

```         
build:
  context: .
  args:
    buildno: 1
    gitcommithash: cdc3b19
```

```         

build:
  context: .
  args:
    - buildno=1
    - gitcommithash=cdc3b19
```

这时构建过程中使用的参数的值为args指定的值。在指定构建参数时也可以不指定值，在这种情况下，构建过程中使用的参数的值为运行Compose的环境中的值。例如：

```         
args:
  - buildno
  - gitcommithash
```

使用布尔值时的注意事项：YMAL中布尔类型的值（"true"、"false"、"yes"、"no"、"on"、"off"）必须用引号引起来，以便解析器将它们解释为字符串。

**(4) cache_from**

在3.2版的配置文件格式中加入

指定缓存解析镜像列表。例如：

```         
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```

**(5) labels**

在3.3版的配置文件格式中加入

将元数据以标签的形式添加到生成的镜像中。可以使用数组或字典两种格式。推荐使用反向DNS写法以避免和其他应用的标签冲突。例如：

```         
build:
  context: .
  labels:
    com.example.description: "Accounting webapp"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""
```

```         
build:
  context: .
  labels:
    - "com.example.description=Accounting webapp"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
```

**(6) network**

在3.4版的配置文件格式中加入

设置容器网络连接以获取构建过程中的RUN指令。例如：

```         
build:
  context: .
  network: custom_network_1
```

设置为none可以在构建期间禁用网络连接。例如：

```         
build:
  context: .
  network: none
```

**(7) shm_size**

在3.5版的配置文件格式中加入

指定容器的/dev/shm分区大小。指定的值为表示字节数的整数值或表示字节值的字符串。例如

```         
build:
  context: .
  shm_size: 10000000
```

```         
build:
  context: .
  shm_size: '2gb'
```

**(8) target**

在3.4版的配置文件格式中加入

指定在Dockerfile中定义的构建阶段，即镜像只构建到指定阶段就停止构建。例如：

```         
build:
  context: .
  target: prod
```

指定构建阶段为prod，即镜像只构建到prod阶段，prod阶段之后的内容不会被构建。

**2. cap_add、cap_drop**

添加或删除容器内核能力（capability）。完整的capability列表可查看man 7 capabilities。例如，让容器拥有所有内核能力：

```         
cap_add:
  - ALL
```

例如，删除NET_ADMIN和SYS_ADMIN能力：

```         
cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，cap_add和cap_drop配置项将被忽略。

**3. cgroup_parent**

为容器指定一个可选的父控制组。例如：

```         
cgroup_parent: m-executor-abcd
```

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，cgroup_parent配置项将被忽略。

**4. command**

覆盖容器启动后默认执行的命令。可以写成字符串形式。例如：

```         
command: bundle exec thin -p 3000
```

也可以写成JSON数组形式。例如：

```         
command: ["bundle", "exec", "thin", "-p", "3000"]
```

**5. configs**

在3.3版的配置文件格式中加入

为每个服务授予对配置（configs）的访问权限。支持short和long两种格式的语法。更多configs信息，参考configs。

注意：该配置（config）必须已存在或者在堆栈文件顶层configs配置项中定义，否则堆栈部署将失败。

short语法仅指定config名称来授予容器访问config的权限并将其挂载到容器的/<config_name>上。source名称和目标挂载点都设置为config名称。例如以下示例，授予了redis服务对configs的my_config和my_other_config的访问权限，其中my_config的值设置到文件./my_config.txt的内容中，my_other_config定义为外部资源，这意味着它已经在Docker中通过运行docker config create命令或其他堆栈部署进行定义，如果外部config不存在，堆栈部署将会失败并显示config not found错误：

```         
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

long语法提供了在服务的任务容器内如何创建config的更多粒度：

source：Docker中存在的config名称。

target：指定要挂载到服务的任务容器的文件的路径加名称。如果未指定，默认为/。

uid和gid：指定服务的任务容器所拥有的该文件的UID或GID。如果在LInux中未指定，两者都默认为0。不支持Windows。

mode：以八进制表示法指定要挂载到服务的任务容器的文件权限。例如，0444代表可读。默认值就为0444。config内容已挂载到临时文件系统中，所以不可写，如果设置了可写位将被忽略。可以设置可执行位。如果不熟悉UNIX文件权限模式，可以使用权限计算器 。

例如以下示例，指定config名称为my_config，授予redis服务对my_config的访问权限，指定要挂载到redis服务的任务容器的路径加文件名称为/redis_config，指定UID和GID均为103，指定要挂载到服务的任务容器的文件权限为0440（group-readable），但该redis服务没有访问my_other_config的权限：

```         
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

可以授予服务访问多个config的权限，并且可以混合long和short语法。定义config并不意味着授予服务对其的访问权限。

**6. container_name**

指定自定义容器的名称，而不是使用默认名称。例如：

```         
container_name: my-web-container
```

因为Docker容器的名称必须唯一，所以为一个服务指定了自定义容器名称后，该服务不能进行扩展。如果尝试为该服务扩容将会导致错误。

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，container_name配置项将被忽略。

**7. credential_spec**

在3.3版的配置文件格式中加入

在3.8或更高版本文件格式中支持将组托管服务帐户（GMSA）配置与Compose配置文件一起使用。

配置托管服务帐户的凭据规格（credential spec）。此选项仅用于使用Windows容器的服务。credential_spec配置必须采用file://或registry://格式。使用file:时，引用的文件必须存在于Docker数据目录的CredentialSpecs子目录中，在Windows上，Docker数据目录默认为C:\ProgramData\Docker。以下示例从名为C:\ProgramData\Docker\CredentialSpecs\my-credential-spec.json的文件加载凭证规格：

```         
credential_spec:
  file: my-credential-spec.json
```

使用registry:时，将从守护进程主机上的Windows注册表中读取凭据规格。其注册表值必须位于HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\CredentialSpecs。以下示例从注册表中名为my-credential-spec的值加载凭证规格：

```         
credential_spec:
  registry: my-credential-spec
```

为服务配置GMSA凭据规格时，只需用config指定凭据规格。例如：

```         
version: "3.8"
services:
  myservice:
    image: myimage:latest
    credential_spec:
      config: my_credential_spec

configs:
  my_credentials_spec:
    file: ./my-credential-spec.json|
```

**8. depends_on**

指定服务之间的依赖关系，解决服务启动先后顺序问题。指定服务之间的依赖关系，将会导致以下行为：

docker-compose up以依赖顺序启动服务。

docker-compose up SERVICE会自动包含SERVICE的依赖项。

docker-compose stop以依赖顺序停止服务。

例如以下示例：

```         
version: "3.8"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

启动时会先启动db和redis，最后才启动web。在使用docker-compose up web启动web时，也会启动db和redis，因为在web服务中指定了依赖关系。在停止时也在web之前先停止db和redis。

使用depends_on时的注意事项：

服务不会等待该服务所依赖的服务完全启动之后才启动。例如上例，web不会等到db和redis完全启动之后才启动。

V3版不再支持的condition形式的depends_on。

V3版中，在swarm mode下部署堆栈时，depends_on配置项将被忽略。

**9. deploy**

在3版的配置文件格式中加入

指定部署和运行服务的相关配置。该配置仅在swarm mode下生效，并只能通过docker stack deploy命令部署，docker-compose up和docker-compose run命令将被忽略。例如：

```         
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      placement:
        max_replicas_per_node: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

deploy配置项中包含endpoint_mode、labels、mode、placement、replicas、resources、restart_policy、update_config等子配置项。

**(1) endpoint_mode**

在3.2版的配置文件格式中加入

为外部客户端连接到swarm指定服务发现方式：

endpoint_mode: vip：Docker为服务分配了一个前端的虚拟IP，客户端通过该虚拟IP访问网络上的服务。Docker在客户端和服务的可用工作节点之间进行路由请求，而无须关系有多少节点正在参与该服务或这些节点的IP地址或者端口。这是默认设置。

endpoint_mode: dnsrr：DNS轮询（DNSRR），Docker设置服务的DNS条目，以便对服务名称的DNS查询返回IP地址列表，并且客户端通过轮询的方式直接连接到其中之一。

例如：

```         
version: "3.8"
services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip
```

**(2) labels**

指定服务的标签。这些标签仅在服务上设置，而不在服务的任何容器上设置。例如：

```         
version: "3.8"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```

**(3) mode**

指定服务的容器副本模式。可以为：

global：每个swarm节点只有一个该服务容器。

replicated：整个集群中存在指定份数的服务容器副本，为默认值。

例如，指定容器副本模式为global：

```         
version: "3.8"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

**(4) placement**

指定constraints和preferences。constraints可以指定只有符合要求的节点上才能运行该服务容器，preferences可以指定容器分配策略。例如，指定集群中只有满足node.rolemanager和engine.labels.operatingsystemubuntu 18.04条件的节点上能运行db服务容器，并且在满足node.labels.zone的节点上均匀分配：

```         
version: "3.8"
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - "node.role==manager"
          - "engine.labels.operatingsystem==ubuntu 18.04"
        preferences:
          - spread: node.labels.zone
```

**(5) max_replicas_per_node**

在3.8版的配置文件格式中加入

如果服务的容器副本模式为replicated（默认），可以指定每个节点上运行的最大容器副本数量。当指定的容器副本数量大于最大容器副本数量时，将引发no suitable node (max replicas per node limit exceed)错误。例如：

```         
version: "3.8"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
      placement:
        max_replicas_per_node: 1
```

**(6) replicas**

如果服务的容器副本模式为replicated（默认），指定运行的容器副本数量。例如：

```         
version: "3.8"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
```

**(7) resources**

配置资源限制。例如，指定redis服务使用的cpu份额为25%到50%，内存为20M到50M：

```         
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

在V3版Compose配置文件中的改变：resources取代了V3版之前的Compose配置文件中旧的资源限制的配置项，包括cpu_shares、cpu_quota、cpuset、mem_limit、memswap_limit、mem_swappiness。

在非swarm mode容器上设置资源限制：此处的resources配置项只有用于deploy配置项之下和swarm mode。如果要在非swarm mode部署中设置资源限制，需使用V2版Compose配置文件中CPU、memory和其他资源的配置项。

**(8) restart_policy**

指定容器的重启策略。代替restart。有以下配置选项：

condition：重启策略。值可以为none、on-failure或any，默认为any。

delay：尝试重启的等待时间。指定为持续时间（durations）。默认值为0。

max_attempts：重启最多尝试的次数，超过该次数将放弃。默认为永不放弃。如果在window配置的时间之内未成功重启，则此次尝试不计入max_attempts的值。

window：在决定重启是否成功之前的等待时间。指定为持续时间（durations）。默认值为立即决定。

例如，指定重启策略为失败时重启，等待5s，重启最多尝试3次，决定重启是否成功前的等待时间为120s：

```         
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

**(9) rollback_config**

在3.7版的配置文件格式中加入

配置在更新失败的情况下如何回滚服务。有以下配置选项：

parallelism：一次回滚的容器数量。如果设置为0，则所有容器同时回滚。

delay：每个容器组之间的回滚所等待的时间。默认值为0s。

failure_action：回滚失败后的行为。有continue和pause两种，默认值为pause。

monitor：每次任务更新后监视失败的时间（ns\|us\|ms\|s\|m\|h）。默认值为0s。

max_failure_ratio：在回滚期间能够容忍的最大失败率。默认值为0。

order：设置回滚顺序。stop-first为在开启新任务之前停止旧任务，start-first为首先启动新任务，和正在运行任务短暂重叠，默认值为stop-first。

**(10) update_config**

配置如何更新服务。该配置对滚动更新很有用。有以下配置选项：

parallelism：一次更新的容器数量。

delay：更新一组容器之间的等待时间。

failure_action：更新失败后的行为。有continue、rollback和pause三种，默认值为pause。

monitor：每次任务更新后监视失败的时间（ns\|us\|ms\|s\|m\|h）。默认值为0s。

max_failure_ratio：在更新期间能够容忍的最大失败率。

order：设置更新顺序。stop-first为在开启新任务之前停止旧任务，start-first为首先启动新任务，和正在运行任务短暂重叠，默认值为stop-first。注意该配置项在3.4版的配置文件格式中加入，仅支持3.4或更高版本。

例如，指定每次更新2个容器，更新等待时间10s，更新顺序为先停止旧任务再开启新任务：

```         
version: "3.8"
services:
  vote:
    image: dockersamples/examplevotingapp_vote:before
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
        order: stop-first
```

**(11) 不支持docker stack deploy的配置项**

以下为支持docker-compose up和docker-compose run，不支持docker stack deploy或deploy配置项的配置项：

```         
build
cgroup_parent
container_name
devices
tmpfs
external_links
links
network_mode
restart
security_opt
userns_mode
```

**10. devices**

指定设备映射列表。与Docker客户端create的--device选项类似。例如：

```         
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，devices配置选项将被忽略。

**11. dns**

自定义DNS服务器。可以是一个值或一个列表。例如：

```         
dns: 8.8.8.8
```

```         
dns:
  - 8.8.8.8
  - 9.9.9.9
```

**12. dns_search**

自定义DNS搜索域。可以是一个值或一个列表。例如：

```         
dns_search: example.com
```

```         
dns_search:
  - dc1.example.com
  - dc2.example.com
```

**13. entrypoint**

覆盖默认的入口命令。注意设置entrypoint会覆盖所有在服务镜像上使用Dockerfile的ENTRYPOINT指令设置的默认入口命令，并清除掉服务镜像上任何使用Dockerfile的CMD指令设置的启动容器时默认执行的命令。可以写成字符串形式，例如：

```         
entrypoint: /code/entrypoint.sh
```

也可以写成JSON数组形式，例如：

```         
entrypoint: ["php", "-d", "memory_limit=-1", "vendor/bin/phpunit"]
```

**14. env_file**

从文件中获取环境变量。可以是一个值或一个列表。例如：

```         
env_file: .env
```

```         
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/runtime_opts.env
```

如果指定了Compose配置文件，env_file路径为相对于该文件所在目录的路径。如果环境文件中设置有与environment选项同名的变量，将以后者为准，无论这些变量的值是空还是未定义。其中环境文件每行都以VAR=VAL格式声明环境变量，以#开头的行被解析为注释，和空行一样将被忽略。环境文件示例如下：

```         
# Set Rails/Rack environment
RACK_ENV=development
```

如果变量的值被引号引起来（通常是shell变量），则引号也包含在传递给Compose的值中。如果以列表的形式同时指定了多个环境文件，列表中文件的顺序对于给多次出现的环境变量确定值十分重要，且列表中的文件是从上到下处理的。如果指定了多个环境文件且有至少两个文件声明了相同名称但不同值的环境变量，那么指定列表中顺序靠下的文件将覆盖顺序靠上的文件中的相同名称的环境变量的值。例如：

```         
services:
  some-service:
    env_file:
      - a.env
      - b.env
```

如果a.env中有VAR=1，b.env中有VAR=2，则最终\$VAR=2。

注意：这里所说的环境变量是针对宿主机的Compose而言的，如果在服务中指定了build配置项，那么这些变量并不会进入构建过程中，如果要定义构建时用的环境变量首选build的arg子选项。

**15. environment**

设置环境变量。可以使用数组或字典两种格式。任何布尔类型的值都必须用引号引起来，以便解析器将它们解释为字符串。值设置了键没设置值的环境变量可以在运行Compose的主机环境中解析它们的值，这对于使用密钥和特定于主机的值用处很大。例如：

```         
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
```

```         
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

注意：这里所说的环境变量是针对宿主机的Compose而言的，如果在服务中指定了build配置项，那么这些变量并不会进入构建过程中，如果要定义构建时用的环境变量首选build的arg子选项。

**16. expose**

暴露指定端口，但不映射到宿主机，只被连接的服务访问。只能指定内部端口。例如：

```         
expose:
  - "3000"
  - "8000"
```

**17. external_links**

链接到docker-compose.yml外部的容器，甚至并非Compose管理的外部容器，特别是对于提供共享或公共服务的容器。在同时指定容器名称和链接别名（CONTAINER:ALIAS）时，external_links与旧版本中的配置项links有类似的语义。例如：

```         
external_links:
  - redis_1
  - project_db_1:mysql
  - project_db_1:postgresql
```

注意：Compose项目里面的容器连接到外部容器的前提条件是外部容器中必须至少有一个容器连接到与项目内的服务的同一个网络里面。建议使用networks代替旧版本中的配置项Links。

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，external_links配置项将被忽略。

**18. extra_hosts**

添加主机名到IP的映射。使用和Docker客户端中的--add-host的参数一样的值。例如：

```         
extra_hosts:
  - "somehost:162.242.195.82"
  - "otherhost:50.31.209.229"
```

会在启动后的服务容器中的/etc/hosts文件中添加如下两条具有主机名和IP地址的条目：

```         
162.242.195.82  somehost
50.31.209.229   otherhost
```

**19. healthcheck**

配置运行检查以确定服务容器是否健康。支持以下配置选项：

test：指定健康检测的方法。

interval：启动容器到进行健康检查的间隔时间以及两次健康检查的间隔时间。

timeout：单次健康检查的超时时间，超过该时间该次健康检查失败。

retries：健康检查失败后的最大重试次数，重试了最大次数依然失败，容器将被视为unhealthy。

start_period：为需要时间引导的容器提供的初始化时间，在此期间检查失败将不计入最大重试次数，但是如果在启动期间健康检查成功，则会将容器视为已启动，并且所有连续失败将计入最大重试次数。

其中interval、timeout和start_period都被指定为持续时间（durations）。start_period是在3.4版的配置文件格式中加入。test必须是字符串或JSON数组格式。如果是JSON数组格式，第一项必须是NONE、CMD或CMD-SHELL其中之一。如果是字符串格式，则等效于指定CMD-SHELL后跟该字符串的JSON数组格式。

例如以下示例，指定检测方法为访问 http://localhost ，健康检查间隔时间为1m30s，健康检查超时时间为10s，重试次数为3，启动容器后等待健康检查的初始化时间为40s：

```         
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

例如以下示例，两种形式等效：

```         
test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
```

```         
test: curl -f https://localhost || exit 1
```

使用disable: true可以设置镜像禁用所有健康检查，相对于test: ["NONE"]。例如：

```         
healthcheck:
  disable: true
```

**20. image**

指定要从中启动容器的镜像。可以写仓库/标签（repository/tag）或镜像ID。示例如下：

```         
image: redis
```

```         
image: ubuntu:18.04
```

```         
image: tutum/influxdb
```

```         
image: example-registry.com:4000/postgresql
```

```         
image: a4bc65fd
```

如果镜像不存在，Compose会自动拉取镜像，除非指定了build，这种情况下会使用指定选项构建镜像并给镜像打上指定标签。

**21. init**

在3.7版的配置文件格式中加入

在容器内运行一个初始化程序以转发信号并获取进程。设置为true即可为服务启动此功能。例如：

```         
version: "3.8"
services:
  web:
    image: alpine:latest
    init: true
```

注意：默认使用的初始化二进制文件是Tini，并安装在主机守护进程的/usr/libexec/docker-init。可以通过Daemon configuration file中的init-path将守护进程配置为使用自定义的初始化二进制文件 。

**22. isolation**

指定容器的隔离技术。Linux上只支持default值。Windows上支持default、process和hyperv这三个值。

**23. labels**

将元数据以标签的形式添加到容器中。可以使用数组或字典两种格式。例如：

```         
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
```

```         
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

**24. links**

警告：--link是Docker的遗留功能。它最终可能会被删除。建议您使用用户自定义网络代替--link来进行两个容器间的通信。用户自定义的网络不支持--link在容器之间共享的环境变量的功能。但是可以使用例如数据卷之类的其他机制以更可控的方式在容器之间共享环境变量。

链接到其他服务中的容器。可以使用"[SERVICE:ALIAS"或"SERVICE"的格式，其中SERVICE为服务名称，ALIAS为链接别名。例如](SERVICE:ALIAS%22或%22SERVICE%22的格式，其中SERVICE为服务名称，ALIAS为链接别名。例如){.uri}：

```         
web:
  links:
    - "db"
    - "db:database"
    - "redis"
```

链接服务的容器可以通过与别名相同的主机名访问，如果未指定别名，则可以使用服务名。默认情况下，不需要链接即可使服务进行通信，任何服务都可以使用该服务的名称访问任何其他服务。链接也可以和depends_on一样表示服务之间的依赖关系 ，因此可以确定服务启动的顺序。

注意：如果同时定义链接和网络，则它们之间具有链接的服务必须共享至少一个公共网络才能进行通信。

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，links配置项将被忽略。

**25. logging**

服务的日志配置。例如：

```         
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

driver配置项指定服务容器的日志驱动，与docker run中的--log-driver选项一样。默认值为json-file，这里列举三种日志驱动类型：

```         
driver: "json-file"
```

```         
driver: "syslog"
```

```         
driver: "none"
```

使用options配置项为日志驱动指定日志记录选项，与docker run中的--log-opt选项一样。例如：

```         
driver: "syslog"
options:
  syslog-address: "tcp://192.168.0.42:123"
```

默认日志驱动json-file具有限制日志存储量的选项。例如以下示例，max-size设置最大存储大小为200k，max-file设置存储的最大文件数为10，随着日志超过最大限制，将删除较旧的日志文件以允许存储新日志：

```         
options:
  max-size: "200k"
  max-file: "10"
```

**26. network_mode**

设置网络模式。使用和Docker客户端中的--network的参数一样的值，格式为service:[service name]。可以指定使用服务或者容器的网络。示例如下：

```         
network_mode: "bridge"
```

```         
network_mode: "host"
```

```         
network_mode: "none"
```

```         
network_mode: "service:[service name]"
```

```         
network_mode: "container:[container name/id]"
```

注意事项：

在swarm mode下部署堆栈时，该选项将被忽略。

network_mode: "host"不能与links配置项混用。

**27. networks**

指定所加入的网络。需要在顶层networks配置项中引入具体的网络信息。例如：

```         
services:
  some-service:
    networks:
     - some-network
     - other-network


networks:
  some-network:
  other-network:
```

**(1) aliases**

指定服务在此网络上的别名（备用主机名）。同一网络上的其他容器可以使用服务名称或此别名来连接到服务的任何一个容器。由于aliases属于网络范围，因此同一服务在不同的网络上可以具有不同的别名。例如：

```         
services:
  some-service:
    networks:
      some-network:
        aliases:
          - alias1
          - alias3
      other-network:
        aliases:
          - alias2
```

在以下示例中，提供了web、worker和db三个服务，以及new和legacy两个网络：

```         
version: "3.8"
services:
  web:
    image: "nginx:alpine"
    networks:
      - new
  worker:
    image: "my-worker-image:latest"
    networks:
      - legacy
  db:
    image: mysql
    networks:
      new:
        aliases:
          - database
      legacy:
        aliases:
          - mysql

networks:
  new:
  legacy:
```

在该例中，可以通过主机名db或database在new网络上访问db服务，通过db或mysql在legacy网络上访问db服务。

注意：网络范围内的别名可以被多个容器甚至多个服务共享。如果是这样，则不能保证名称恰好解析到哪一个容器。

**(2) ipv4_address、ipv6_address**

加入网络后，为此服务的容器指定一个静态IP地址。在顶层networks配置项中的相应网络配置必须有子网配置覆盖每个静态地址的ipam配置。例如：

```         
version: "3.8"
services:
  app:
    image: nginx:alpine
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"
        - subnet: "2001:3984:3989::/64"
```

注意：如果需要IPv6寻址，则必须使用V2.x版本的Compose配置文件并设置顶层networks配置项下的enable_ipv6选项。在当前swarm mode下IPv6选项不会起作用。

**28. pid**

跟主机系统共享进程命名空间。

```         
pid: "host"
```

将PID模式设置为主机PID模式，打开该选项的容器之间，以及容器和宿主机操作系统之间可以通过进程ID来相互访问和操作。

**29. ports**

暴露端口。注意端口映射与network_mode: host不兼容。支持short和long两种格式的语法。short语法可以使用HOST:CONTAINER的格式指定端口映射，也可以指定容器端口，宿主机会随机选择临时端口进行映射。例如：

```         
ports:
  - "3000"
  - "3000-3005"
  - "8000:8000"
  - "9090-9091:8080-8081"
  - "49100:22"
  - "127.0.0.1:8001:8001"
  - "127.0.0.1:5000-5010:5000-5010"
  - "6060:6060/udp"
  - "12400-12500:1240"
```

注意：当使用HOST:CONTAINE格式来映射端口时，如果使用的容器端口小于60可能会得到错误得结果，因为YAML将会解析xx:yy这种数字格式为60进制，因此建议始终采用字符串格式来指定端口映射。

long语法支持配置short语法中不支持的附加字段。这些附加字段如下：

target：指定容器内的端口。

published：指定公开的端口。

protocol：指定端口协议（tcp或udp）。

mode：使用host在每个节点公开一个主机端口，或使用ingress对swarm mode端口进行负载均衡。

示例如下：

```         
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

long语法在3.2版的配置文件格式中加入

**30. restart**

指定重启策略。例如想要在容器退出时总是会重启容器，指定以下重启策略：

```         
restart: always
```

一共支持以下重启策略：

no：在任何情况下都不会重启容器。默认的重启策略。

always：在容器退出时总是重启容器。

on-failure：在容器以非0状态码退出时才会重启。

unless-stopped：在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器。

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，restart配置项将被忽略。

**31. secrets**

为每个服务授予对保密数据（secrets）的访问权限。支持short和long两种格式的语法。更多secrets信息，参考secrets。

使用docker stack deploy时的注意事项：该保密数据（secret）必须已存在或者在Compose配置文件顶层secrets配置项中定义，否则堆栈部署将失败。

short语法仅指定secret名称来授予容器访问secret数据的权限并将其挂载到容器的/run/secrets/<secret_name>上。source名称和目标挂载点都设置为secret名称。例如以下示例，授予了redis服务对secrets的my_secret和my_other_secret的访问权限，其中my_secret的值设置到文件./my_secret.txt的内容中，my_other_secret定义为外部资源，这意味着它已经在Docker中通过运行docker secret create命令或其他堆栈部署进行定义，如果外部secret不存在，堆栈部署将会失败并显示secret not found错误：

```         
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

long语法提供了在服务的任务容器内如何创建secret的更多粒度：

source：Docker中存在的secret名称。

target：指定要挂载到服务的任务容器的/run/secrets/中的文件名称。如果未指定，默认为source的值。

uid和gid：指定服务的任务容器的/run/secrets/中所拥有的该文件的UID或GID。如果未指定，两者都默认为0。

mode：以八进制表示法指定要挂载到服务的任务容器的/run/secrets/中的文件权限。例如，0444代表可读。默认值在Docker 1.13.1为0000，在较新版本为0444。secret内容已挂载到临时文件系统中，所以不可写，如果设置了可写位将被忽略。可以设置可执行位。如果不熟悉UNIX文件权限模式，可以使用权限计算器 。

例如以下示例，指定secret名称为my_secret，授予redis服务对my_secret的访问权限，指定要挂载到redis服务的任务容器的/run/secrets/中的文件名称为redis_secret，指定UID和GID均为103，指定要挂载到服务的任务容器的/run/secrets/中的文件权限为0440（group-readable），但该redis服务没有访问my_other_secret的权限：

```         
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

可以授予服务访问多个secret的权限，并且可以混合long和short语法。定义secret并不意味着授予服务对其的访问权限。

**32. security_opt**

为每个容器覆盖默认的标签（label）。例如配置标签中的用户名和角色名：

```         
security_opt:
  - label:user:USER
  - label:role:ROLE
```

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，security_opt配置项将被忽略。

**33. stop_grace_period**

指定在发送SIGKILL之前，如果容器无法处理SIGTERM或使用stop_signal指定的任何停止信号时，试图停止该容器所需要的等待时间。默认情况下，stop在发送SIGKILL之前等待10秒以退出容器。时间指定为duration。具体示例如下：

```         
stop_grace_period: 1s
```

```         
stop_grace_period: 1m30s
```

**34. stop_signal**

设置停止容器的信号。默认使用SIGTERM停止容器。例如指定SIGUSR1信号：

```         
stop_signal: SIGUSR1
```

**35. sysctls**

在容器中设置的内核参数。可以使用数组或字典两种格式。例如指定连接数为1024和开启TCP的syncookies：

```         
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0
```

```         
sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，sysctls配置项要求Docker Engine 19.03或以上。

**36. tmpfs**

在3.6版的配置文件格式中加入

挂载一个临时文件系统到容器内部。可以是一个值或一个列表。例如：

```         
tmpfs: /run
1.
tmpfs:
  - /run
  - /tmp
```

在容器内挂载一个临时文件系统。Size可以指定挂载的tmpfs大小，以字节为单位。默认情况下不受限制。例如：

```         
 - type: tmpfs
     target: /app
     tmpfs:
       size: 1000
```

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时使用3至3.5版本的Compose配置文件，tmpfs配置项将被忽略。

**37. ulimits**

覆盖容器的默认ulimit值。可以单一地将限制值设为一个整数，也可以将soft/hard限制指定为映射。例如，指定最大进程数为65535，指定文件句柄数，其中软限制为20000，系统硬限制为40000，软限制应用可以随时修改，不能超过硬限制，硬限制只能root用户提高：

```         
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```

**38. userns_mode**

指定用户命名空间模式。如果Docker守护进程配置了用户名称空间，则禁用此服务的用户名称空间。以下是使用主机上的用户命名空间的配置示例：

```         
userns_mode: "host"
```

使用docker stack deploy时的注意事项：在swarm mode下部署堆栈时，userns_mode配置项将被忽略。

**39. volumes**

指定所挂载的主机路径或数据卷名称。支持short和long两种格式的语法。可以将主机路径作为单个服务的一部分进行挂载，而无需在顶层volumes配置项中定义。但是如果想要在多个服务之间重用数据卷，需要在顶层volumes配置项中定义一个数据卷名称。

在3版的配置文件格式中的变化：在顶层volumes配置项中定义了数据卷名称并从每个服务的volumes列表中引用了该数据卷。这将替代早期版本的Compose配置文件格式中的volumes_from配置项。

short语法使用通用的[SOURCE:]TARGET[:MODE]格式，SOURCE可以是主机路径或数据卷名称，TARGET为挂载数据卷的容器路径，MODE可以为ro只读模式或rw读写模式（默认）。可以在主机上挂载相对路径，该路径相对于正在使用的Compose配置文件的目录进行扩展，相对路径应始终以.或..开头。例如：

```         
volumes:
  #只指定一个路径，Docker会自动在创建一个数据卷（这个路径是容器内部的）
  - /var/lib/mysql
  #使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql
  #使用基于Compose配置文件的相对路径作为数据卷挂载到容器
  - ./cache:/tmp/cache
  #使用基于root用户的相对路径作为数据卷挂载到容器
  - ~/configs:/etc/configs/:ro
  #使用已经存在命名的数据卷挂载到容器
  - datavolume:/var/lib/mysql
```

long语法支持配置以下short语法中不支持的附加字段：

type：挂载类型，可以为volume、bind、tmpfs或npipe。

source：挂载源，在主机上用于绑定挂载的路径或定义在顶层volumes配置项中的数据卷名称。不适用于tmpfs挂载类型。

target：数据卷挂载在容器中的路径。

read_only：设置数据卷为只读。

bind：配置额外的bind选项。

propagation：用于绑定的传播模式。

volume：配置额外的volume选项。

nocopy：创建数据卷时禁止从容器复制数据。

tmpfs：配置额外的tmpfs选项。

size：tmpfs挂载的大小，以字节为单位。

consistency：挂载的一致性要求，可以为consistent、cached或delegated。其中consistent表示主机和容器具有相同视图。cached表示读取缓存，主机视图是权威的。delegated表示读写缓存，容器视图是权威的。

例如：

```         
version: "3.8"
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

networks:
  webnet:

volumes:
  mydata:
```

**40. 其他配置项**

此外，还有domainname、hostname、ipc、mac_address、privileged、read_only、shm_size、stdin_open、tty、user和working_dir这些配置项。它们都是单值配置，和docker run中的对应选项类似。注意mac_address是旧版本配置项。例如：

```         
#指定容器中运行应用的用户名
user: postgresql
#指定容器的工作目录
working_dir: /code
#指定容器中的搜索域名
domainname: foo.com
#指定容器中的主机名
hostname: foo
ipc: host
#指定容器中的mac地址
mac_address: 02:42:ac:11:65:43
#指定容器为特权容器
privileged: true
#指定以只读模式挂载容器的root文件系统，不能对容器内容进行修改
read_only: true
#设置/dev/shm分区的大小
shm_size: 64M
#打开标准输入，可以接受外部输入
stdin_open: true
#分配tty设备用来支持终端登录
tty: true
```

**四、数据卷配置**

虽然可以声明即时生效的数据卷作为服务声明的一部分，但这部分可以通过顶层volumes配置项定义一个数据卷以实现在多个服务之间重用，并且可以使用docker命令行或API轻松进行检索和检查。例如以下示例，数据库的数据目录作为数据卷与另一个服务共享以便定期备份：

```         
version: "3.8"
services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data

volumes:
  data-volume:
```

顶层volumes配置项下的条目可以为空，这种情况下配置的默认驱动，默认驱动在大多数情况下为local驱动。下面就对数据卷的相关配置项进行一个总结。

**1. driver**

指定该数据卷所要使用的数据卷驱动。默认为Docker Engine中配置使用的无论哪种驱动，大多数情况下为local驱动。如果驱动不可用，则引擎会在docker-compose up尝试创建数据卷时返回一个错误。以下为指定数据卷驱动的示例：

```         
driver: foobar
```

**2. driver_opts**

以键值对的形式指定用来传递给该数据卷所使用的数据卷驱动的列表选项。这些选项取决于数据卷驱动，可以参考数据卷驱动程序文档。例如：

```         
volumes:
  example:
    driver_opts:
      type: "nfs"
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/example"
```

**3. external**

指定该数据卷是否是外部数据卷。如果设置为true，则指定该数据卷是在Compose外部创建的。由于docker-compose up不会尝试创建该数据卷，如果该数据卷不存在则会引发错误。在3.3及以下版本的Compose配置文件格式中，external配置项不能与包括driver、driver_opts和labels在内的其他数据卷配置项同时使用，在3.4及以上版本中则没有此限制。例如以下示例，Compose不会尝试创建一个名为[projectname]\_data的数据卷，而是查找一个现存的被简单称为data的数据卷并将其挂载到db服务的容器中：

```         
version: "3.8"
services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data

volumes:
  data:
    external: true
```

通过external配置项下的name配置项可以指定数据卷的名称，该名称与Compose配置文件中用于引用数据卷的名称区分开来。例如：

```         
volumes:
  data:
    external:
      name: actual-name-of-volume
```

不过external配置项下的name配置项在3.4版的配置文件格式中就已经被弃用了，可以用name配置项替代。

使用docker stack deploy时的注意事项：如果使用docker stack deploy代替docker-compose up以swarm mode启动应用，则会创建不存在的外部数据卷。在swarm mode下，服务定义数据卷后将自动创建该卷。由于服务任务已在新节点上安排，因此SwarmKit将在本地节点上创建数据卷。

**4. labels**

将元数据以标签的形式添加到容器中。可以使用数组或字典两种格式。例如：

```         
labels:
  com.example.description: "Database volume"
  com.example.department: "IT/Ops"
  com.example.label-with-empty-value: ""
```

```         
labels:
  - "com.example.description=Database volume"
  - "com.example.department=IT/Ops"
  - "com.example.label-with-empty-value"
```

**5. name**

在3.4版的配置文件格式中加入

为该数据卷设置一个自定义名称。例如：

```         
version: "3.8"
volumes:
  data:
    name: my-app-data
```

可以和external配置项一起使用。例如：

```         
version: "3.8"
volumes:
  data:
    external: true
    name: my-app-data
```

**五、网络配置**

通过顶层networks配置项可以指定要创建的网络。下面就对网络的相关配置项进行一个总结。

**1. driver**

指定该网络所要使用的驱动。默认驱动取决于所使用的Docker Engine的配置方式，但是在大多数情况下，单个主机上用的bridge，Swarm上用的overlay。

bridge：表示桥接，Docker在单个主机上默认使用bridge网络。

overlay：overlay驱动在一个swarm中的多个节点之间创建一个命名网络。

如果驱动不可用，则Docker Engine会返回一个错误。以下为指定卷驱动的示例：

```         
driver: overlay
```

如果想要使用主机的网络堆栈，或者不使用网络。这相当于docker run --net = host或docker run --net = none。仅在使用docker stack命令时使用。如果使用docker-compose命令，需使用服务配置中的network_mode配置项。如果要在相同构建版本的容器上使用特定网络，需要在服务配置的build下的network配置项中设置。例如：

```         
services:
  web:
    build:
      network: host
      context: .
```

使用诸如host和none的内置网络的语法有所不同。定义一个名为host或none的外部网络以及Compose可以使用的别名，然后使用该别名向该网络授予服务访问权限，而且该外部网络在Docker中已经自动创建。例如以下示例，定义了名为host的外部网络，Compose中可以使用的别名为hostnet：

```         
version: "3.8"
services:
  web:
    networks:
      hostnet: {}


networks:
  hostnet:
    external: true
    name: host
```

例如以下示例，定义了名为none的外部网络，Compose中可以使用的别名为nonet：

```         
services:
  web:
    networks:
      nonet: {}

networks:
  nonet:
    external: true
    name: none
```

**2. driver_opts**

以键值对的形式指定用来传递给该网络所使用的驱动的列表选项。这些选项取决于驱动，可以参考驱动程序文档。例如：

```         
driver_opts:
  foo: "bar"
  baz: 1
```

**3. attachable**

在3.2版的配置文件格式中加入

指定是否允许除服务之外的独立容器连接到该网络。仅在driver设置为overlay时使用。如果设置为true，则除了服务之外的独立容器也可以连接到该网络。如果独立容器连接到了overlay网络，那它可以与那些也从其他Docker守护进程连接到overlay网络的服务和独立容器进行通信。例如：

```         
networks:
  mynet1:
    driver: overlay
    attachable: true
```

**4. enable_ipv6**

在该网络上启用IPv6网络。

不支持V3版Compose配置文件：如果要使用enable_ipv6配置项，则需要使用V2版Compose配置文件，因为swarm mode尚不支持该配置项。

**5. ipam**

指定自定义IPAM配置。有以下配置选项：

driver：自定义IPAM驱动以代替默认驱动。

config：包含零个或多个配置块的列表，每个配置块可以有以下配置选项。

subnet：设置表示网段的CIDR格式的子网。

以下为指定自定义IPAM配置的示例：

```         
ipam:
  driver: default
  config:
    - subnet: 172.28.0.0/16
```

注意：目前，例如gateway等其他IPAM配置仅适用于V2版Compose配置文件。

**6. internal**

指定是否创建一个与外部隔离的overlay网络。默认情况下，Docker也会将桥接网络连接到它以提供外部连接。如果要创建一个与外部隔离的overlay网络，需将此配置项设置为true。

**7. labels**

将元数据以标签的形式添加到容器中。可以使用数组或字典两种格式。例如：

```         
labels:
  com.example.description: "Financial transaction network"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
```

```         
labels:
  - "com.example.description=Financial transaction network"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

**8. external**

指定该网络是否是外部网络。如果设置为true，则指定该网络是在Compose外部创建的。由于docker-compose up不会尝试创建该网络，如果该网络不存在则会引发错误。在3.3及以下版本的Compose配置文件格式中，external配置项不能与包括driver、driver_opts、ipam和internal等在内的其他网络配置项同时使用，在3.4及以上版本中则没有此限制。例如以下示例，proxy是通往外界的网关，Compose不会尝试创建一个名为[projectname] \_outside的网络，而是查找一个现存的被简单称为的outside网络并将proxy服务的容器连接到该网络：

```         
version: "3.8"
services:
  proxy:
    build: ./proxy
    networks:
      - outside
      - default
  app:
    build: ./app
    networks:
      - default

networks:
  outside:
    external: true
```

通过external配置项下的name配置项可以指定网络的名称，该名称与Compose配置文件中用于引用网络的名称区分开来。例如：

```         
networks:
  outside:
    external:
      name: actual-name-of-network
```

不过external配置项下的name配置项在3.5版的配置文件格式中就已经被弃用了，可以用name配置项替代。

**9. name**

在3.5版的配置文件格式中加入

为该网络设置一个自定义名称。例如：

```         
version: "3.8"
networks:
  network1:
    name: my-app-net
```

可以和external配置项一起使用。例如：

```         
version: "3.8"
networks:
  network1:
    external: true
    name: my-app-net
```

**六、configs配置**

顶层configs配置项定义或引用了授予此堆栈中的服务的configs，config来源于file或external。下面就对configs的相关配置项进行一个简单总结。

file：通过指定路径中的文件内容创建config。

external：如果设置为true，则指定该config已经创建。Docker不会尝试创建它，如果它不存在，会发生config not found错误。

name：指定Docker中config的名称。在3.5版的配置文件格式中加入。

driver和driver_opts：自定义secret驱动的名称，和以键值对的形式指定用来传递给特定驱动的选项。这两个配置项均在3.8版的配置文件格式中加入，仅在使用docker stack时受支持。

template_driver：指定要使用的模板驱动的名称，它控制是否以及如何将secret有效负载作为模板进行评估。如果未设置驱动，则不使用任何模板。当前仅支持使用Go语言的golang驱动。该配置项在3.8版的配置文件格式中加入，仅在使用docker stack时受支持。

**七、secrets配置**

顶层secrets配置项定义或引用了授予此堆栈中的服务的secrets，secret来源于file或external。下面就对secrets的相关配置项进行一个简单总结。

file：通过指定路径中的文件内容创建secret。

external：如果设置为true，则指定该secret已经创建。Docker不会尝试创建它，如果它不存在，会发生secretg not found错误。

name：指定Docker中secret的名称。在3.5版的配置文件格式中加入。

template_driver：指定要使用的模板驱动的名称，它控制是否以及如何将secret有效负载作为模板进行评估。如果未设置驱动，则不使用任何模板。当前仅支持使用Go语言的golang驱动。该配置项在3.8版的配置文件格式中加入，仅在使用docker stack时受支持。

**八、扩展字段**

在3.4版的配置文件格式中加入

可以使用扩展字段来定义可重用的配置片段。这些特殊字段只要位于Compose配置文件的最顶（root）层，可以为任何格式，它们的名称以x-开头。

注意：从3.7版（对于3.x系列）和2.4版（对于2.x系列）的配置文件格式开始，扩展字段可以位于服务、数据卷、网络、config和 secret定义部分的root层。

例如以下示例，通过扩展字段x-custom定义一个可重用的配置片段：

```         
version: '3.4'
x-custom:
  items:
    - a
    - b
  options:
    max-size: '12m'
  name: "custom"
```

这些扩展字段内容将被Compose忽略，但是可以通过YAML锚点来引用扩展字段内容。例如以下日志记录配置，希望被多个服务引用：

```         
logging:
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file
```

可以通过&开头的字符串在扩展字段x-logging定义的日志记录配置片段中设置锚点，然后在需要引用该日志记录配置片段的服务中以\*加上锚点名进行引用：

```         
version: '3.4'
x-logging:
  &default-logging
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file


services:
  web:
    image: myapp/web:latest
    logging: *default-logging
  db:
    image: mysql:latest
    logging: *default-logging
```

也可以通过YAML合并来部分覆盖扩展字段中的值。例如：

```         
version: '3.4'
x-volumes:
  &default-volume
  driver: foobar-storage

services:
  web:
    image: myapp/web:latest
    volumes: ["vol1", "vol2", "vol3"]
volumes:
  vol1: *default-volume
  vol2:
    << : *default-volume
    name: volume02
  vol3:
    << : *default-volume
    driver: default
    name: volume-local
```

**九、其他内容**

**1. 指定持续时间（durations）**

某些配置项如healthcheck配置项的子配置项interval和timeout接受以类似如下格式的字符串表示的持续时间（durations）：

```         
2.5s
10s
1m30s
2h32m
5h34m56s
```

支持的单位有us、ms、s、m和h。

**2. 指定字节值**

某些配置项如build配置项的子配置项shm_size接受以类似如下格式的字符串表示的字节值：

```         
2b
1024kb
2048k
300m
1gb
```

支持的单位有b、k或kb、m或mb、g或gb。目前不支持十进制。

**3. 变量替换**

配置项中的值可以包含环境变量，Compose会使用运行docker-compose时所在的shell中的环境变量值来替换Compose配置文件中的环境变量，${VARIABLE}和$VARIABLE这两种语法都支持。例如以下示例，假设shell中有POSTGRES_VERSION=9.3这个环境变量，则配置中的${POSTGRES_VERSION}将被替换为9.3：
```
db:
  image: "postgres:${POSTGRES_VERSION}"

```         

如果未设置环境变量，则Compose会将配置文件中的环境变量替换为空字符串。如果上例中的POSTGRES_VERSION未设置，则image的值为postgres:。可以使用.env环境文件为环境变量设置默认值，不过shell中的环境变量值会覆盖.env环境文件中设置的值。

使用docker stack deploy时的注意事项：.env环境文件仅在使用docker-compose up命令时有效，不适用于docker stack deploy。

此外，可以使用典型的shell语法提供内联默认值，支持以下两种语法：

${VARIABLE:-default}：如果VARIABLE未设置或为空，则会应用default的值。

${VARIABLE-default}：仅当VARIABLE未设置时才会应用default的值。

同样，也可以使用以下两种语法来强制要求变量必须赋值：

${VARIABLE:?err}：如果VARIABLE未设置或为空，退出并输出一条包含err的错误信息。

${VARIABLE?err}：如果VARIABLE未设置，退出并输出一条包含err的错误信息。

当配置中需要\(符号时，可以使用\)\(符号。也可以防止Compose插值，因此\)$ 可以引用不想由Compose处理的环境变量。例如以下示例，就算设置了环境变量VAR_NOT_INTERPOLATED_BY_COMPOSE，Compose配置文件中的VAR_NOT_INTERPOLATED_BY_COMPOSE也不会被替换：
```

web: build: . command: "\$\$VAR_NOT_INTERPOLATED_BY_COMPOSE"

\`\`\`

如果少指定了一个$，也就是只使用单个$符号，则Compose会将值解释为环境变量，并出现一个类似The VAR_NOT_INTERPOLATED_BY_COMPOSE is not set. Substituting an empty string.这样的警告。
