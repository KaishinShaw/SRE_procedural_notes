# 私有 Docker 镜像仓库搭建教程

本教程将详细介绍如何建立一个私有 Docker 镜像仓库。通过自托管镜像仓库，您可以更安全、更自由地管理和分发容器镜像。

## 1. 什么是容器镜像？

容器镜像是包含运行容器所需的所有文件、库和配置的一个包。通常通过 Dockerfile 来创建容器镜像。

**构建镜像示例：**

``` bash
docker build -t pliutau/hello-world:v0 .
docker images
```

输出示例：

```         
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
hello-world   latest    9facd12bbcdd   22 seconds ago   11MB
```

构建好的镜像需要上传到容器镜像仓库以便共享。

## 2. 什么是容器镜像仓库？

容器镜像仓库是用于存储、推送和拉取容器镜像的服务。镜像仓库既可以是公开的（如 Docker Hub），也可以是私有的。

**推送镜像到 Docker Hub 示例：**

``` bash
docker push docker.io/pliutau/hello-world:v0
```

镜像 URL 解析：

```         
docker pull docker.io/pliutau/hello-world:v0@sha256:dc11b2...
                |            |            |          |
                ↓            ↓            ↓          ↓
             registry    repository      tag       digest
```

## 3. 为什么要自托管容器镜像仓库

-   **安全性**：控制镜像的访问权限，保障敏感数据不外泄。
-   **自由度**：对镜像仓库的管理和配置拥有完全的控制权。
-   **减少依赖**：降低对外部服务提供商的依赖，增强业务连续性。

## 4. 如何自托管容器镜像仓库

我们将使用 Docker 官方支持的 `registry` 镜像来搭建私有仓库，并配置 Nginx 和 SSL 证书以确保安全。

### 步骤一：安装 Docker 和 Docker Compose

``` bash
sudo snap install docker
docker --version
docker-compose --version
```

### 步骤二：运行仓库容器

**创建 `docker-compose.yml` 配置文件：**

``` yaml
services:
  registry:
    image: registry:latest
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./registry/registry.password:/auth/registry.password
      - ./registry/data:/data
    ports:
      - "5000:5000"
```

------------------------------------------------------------------------

这是一个用于定义Docker容器服务配置的YAML文件片段，通常用于Docker Compose或类似的容器编排工具。

``` yaml
services:
```

-   `services:` 键定义了一个或多个服务（在此上下文中，每个服务通常是一个容器），这些服务将被Docker Compose管理。

``` yaml
  registry:
```

-   `registry:` 是定义的服务名称，这里指的是将要启动的容器的名字。

``` yaml
    image: registry:latest
```

-   `image:` 指定了容器使用的镜像。这里使用的是 `registry` 镜像的最新版本（`latest`标签）。

``` yaml
    environment:
```

-   `environment:` 键下定义了容器运行时的环境变量。

``` yaml
      REGISTRY_AUTH: htpasswd
```

-   `REGISTRY_AUTH:` 环境变量设置了Docker registry的认证方式，这里使用的是`htpasswd`，基于用户名和密码的认证。

``` yaml
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
```

-   `REGISTRY_AUTH_HTPASWD_REALM:` 设置了用于HTTP基本认证时显示给用户的领域（realm）名称。

``` yaml
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
```

-   `REGISTRY_AUTH_HTPASSWD_PATH:` 指定了存储htpasswd文件的路径，该文件包含了可以访问registry的用户名和密码。

``` yaml
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
```

-   `REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY:` 设置了容器内部用于存储数据的根目录路径。

``` yaml
    volumes:
```

-   `volumes:` 键定义了宿主机和容器之间的卷映射。

``` yaml
      # Mount the password file
      - ./registry/registry.password:/auth/registry.password
```

-   这行定义了一个卷映射，将宿主机的`./registry/registry.password`文件映射到容器的`/auth/registry.password`。这使得容器可以访问外部定义的密码文件。

``` yaml
      # Mount the data directory
      - ./registry/data:/data
```

-   这行定义了另一个卷映射，将宿主机的`./registry/data`目录映射到容器的`/data`目录。这用于持久化或共享数据。

``` yaml
    ports:
```

-   `ports:` 键定义了容器和宿主机之间的端口映射。

``` yaml
      - 5000
```

-   这行将容器的5000端口映射到宿主机的同一端口上，使得可以通过宿主机的5000端口访问容器提供的服务。

整个配置定义了一个名为`registry`的Docker服务，使用`registry:latest`镜像，并配置了基本的认证和存储，同时提供了必要的文件映射和端口映射。这样的服务通常用于私有的Docker镜像仓库。

------------------------------------------------------------------------

### 使用 `htpasswd` 命令创建密码文件

1. **创建存储数据的目录**：
   通过 `mkdir` 命令创建一个名为 `registry/data` 的目录，以存储将来需要用到的数据。

   ```bash
   $ mkdir -p ./registry/data
   ```

2. **安装 `apache2-utils`**：
   使用 `sudo apt install apache2-utils` 命令安装 `apache2-utils` 包，该包包含 `htpasswd` 工具，用于生成和管理用户密码文件。

   ```bash
   $ sudo apt install apache2-utils
   ```

3. **生成密码文件**：
   使用 `htpasswd` 命令创建名为 `registry.password` 的密码文件，该文件位于 `registry` 目录中。此命令将为用户 `busy` 设置密码 `bee`，并使用 `-B` 选项来请求 bcrypt 算法加密密码，`-bn` 选项表示生成密码而不提示输入密码。

   ```bash
   $ htpasswd -Bbn busy bee > ./registry/registry.password
   ```

### 启动仓库容器

1. **使用 Docker Compose 启动容器**：
   执行 `docker-compose up` 命令来启动定义在 `docker-compose.yml` 文件中的服务。这通常会启动一个或多个容器，具体取决于配置文件的内容。

   ```bash
   $ docker-compose up
   ```

2. **成功启动的输出示例**：
   如果容器成功启动，你会看到类似以下的输出信息，表明仓库服务已在 `5000` 端口监听连接：

   ```
   # registry | level=info msg="listening on [::]:5000"
   ```

以上步骤完成后，你便成功创建了一个带有基本认证的 Docker 仓库，并且仓库服务已经在本地的 5000 端口上启动和监听。

### 步骤三：搭建 Nginx

**继续配置 `docker-compose.yml`：**

``` yaml
services:
  nginx:
    image: nginx:latest
    depends_on:
      - registry
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/certs:/etc/nginx/certs
    ports:
      - "443:443"
```
___

这部分配置定义了两个服务：`registry` 和 `nginx`。下面是对其中 `nginx` 部分的逐行解释：

```yaml
services:
```
- `services:` 键开始定义了一个或多个服务配置，每个服务通常是一个容器。

```yaml
  registry:
```
- 这一行表示定义了一个名为 `registry` 的服务，但具体配置在这里被省略了（表示为 `# ...`）。

```yaml
  nginx:
```
- `nginx:` 这个键定义了另一个服务，名称为 `nginx`。

```yaml
    image: nginx:latest
```
- `image:` 指定了 `nginx` 服务使用的Docker镜像，这里使用的是名为 `nginx` 的官方镜像，版本标签为 `latest`。

```yaml
    depends_on:
```
- `depends_on:` 键用于定义服务间的依赖关系。

```yaml
      - registry
```
- 这行表明 `nginx` 服务启动依赖于 `registry` 服务的启动。即 `nginx` 服务会在 `registry` 服务成功启动后才开始启动。

```yaml
    volumes:
```
- `volumes:` 定义了容器与宿主机之间的存储卷映射。

```yaml
      # mount the nginx configuration
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
```
- 这行映射了宿主机上的 `./nginx/nginx.conf` 文件到容器内的 `/etc/nginx/nginx.conf`。这样做是为了让 `nginx` 使用自定义的配置文件。

```yaml
      # mount the certificates obtained from Let's Encrypt
      - ./nginx/certs:/etc/nginx/certs
```
- 这行映射了宿主机上的 `./nginx/certs` 目录到容器内的 `/etc/nginx/certs`。这个目录包含了从 Let's Encrypt 获取的SSL/TLS证书，用于安全的HTTPS连接。

```yaml
    ports:
```
- `ports:` 定义了容器与宿主机之间的网络端口映射。

```yaml
      - "443:443"
```
- 这行表示将容器的443端口映射到宿主机的443端口上。这是HTTPS协议通常使用的端口，使得可以通过宿主机的443端口访问 `nginx` 服务提供的安全网页。
___

### 步骤四：测试

**登录私有仓库并推送镜像：**

``` bash
docker login registry.pliutau.com
docker build -t registry.pliutau.com/pliutau/hello-world:v0 .
docker push registry.pliutau.com/pliutau/hello-world:v0
```

通过以上步骤，您就能成功搭建并使用自己的私有 Docker 镜像仓库了。

------------------------------------------------------------------------
在你已经运行了 `docker-compose up` 之后，向 `docker-compose.yml` 文件中添加新的服务是完全可行的，但是需要执行一些额外的步骤来确保新添加的服务也被正确地启动和管理。
下面是你应该遵循的步骤：

1. **编辑 `docker-compose.yml` 文件**：
   打开你的 `docker-compose.yml` 文件并添加你需要的服务配置。根据你的描述，这意味着在文件中加入 `nginx` 服务的定义。

2. **保存并重新加载配置**：
   保存你对 `docker-compose.yml` 文件的修改之后，你需要重新加载 Docker Compose 的配置。这可以通过两种方式完成：

   - **完全重新启动**：使用 `docker-compose down` 命令停止并删除所有当前运行的服务（这会导致短暂的服务中断），然后使用 `docker-compose up` 或 `docker-compose up -d` 重新启动所有服务，包括你新添加的 `nginx` 服务。
   - **部分更新**：如果你不想停止其他已经运行的服务，你可以使用 `docker-compose up -d` 命令。Docker Compose 会识别出新添加或者修改过的服务，并只对这些服务进行创建和启动。

3. **确认服务运行**：
   在进行了上述步骤之后，你可以通过 `docker-compose ps` 来检查所有服务的状态，确认 `nginx` 服务是否已经正常运行。

4. **检查网络和依赖**：
   由于你的 `nginx` 服务使用了 `depends_on` 来指定依赖于 `registry` 服务，确保 `registry` 服务在 `nginx` 启动前已经在运行状态。`depends_on` 仅确保顺序启动，不会等待 `registry` 服务完全就绪。

5. **配置和端口映射**：
   确保你的 nginx 配置文件 `./nginx/nginx.conf` 和证书目录 `./nginx/certs` 都已经准备就绪，并且目录路径正确无误。端口映射也应当没有与宿主机上的其他服务冲突。

添加服务到 `docker-compose.yml` 并不会自动应用更新，需要手动触发更新。建议在生产环境中进行此类操作前，先在开发或测试环境验证无误。