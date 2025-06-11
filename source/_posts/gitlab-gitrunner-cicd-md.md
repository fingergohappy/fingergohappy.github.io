---
title: gitlab使用gitlab-runner进行CICDD指引
cover: https://source.unsplash.com//1200x628
toc: true
date: 2025-06-11 14:13:27
tags:
    - cicd
    - gitlab
    - pipeline
categories:
    - cicd
    - gitlab
    - pipeline
---

gitlab提供了强大的CI/CD功能，可以通过`gitlab-runner`来实现自动化构建、测试和部署。
和其他CI/CD工具相比，gitlab的CI/CD功能集成度更高，使用起来也更方便。

<!--more-->
## 下载安装gitlab-runner

`gitlab-runner`和`gitlab`是分离的，gitlab-runner可以安装在任何服务器上，只要能访问到gitlab即可。


[安装指南](https://docs.gitlab.com/runner/install)


## 配置gitlab-runner

### 获取注册令牌

项目设置 -> CI/CD -> Runner设置 -> 注册令牌

### 注册gitlab-runner

使用注册令牌注册runner,我这里注册两个,一个用于部署，一个用于编译。
```bash
sudo gitlab-runner register \
  --non-interactive \
  --url "your.gitlab-server.com" \
  --registration-token "token" \
  --executor "shell" \
  --description "shell-deploy-runner" \
  --tag-list "shell,deploy" \
  --run-untagged="false" \
  --locked="false" \
  --tls-ca-file=""



sudo gitlab-runner register \
  --non-interactive \
  --url "your.gitlab-server.com" \
  --registration-token "xxx" \
  --executor "docker" \
  --docker-image "node:20-alpine" \
  --description "docker-node-build-runner" \
  --tag-list "node-20" \
  --run-untagged="true" \
  --locked="false" \
  --tls-ca-file="" 
```


参数说明:

```--non-interactive: 非交互式注册
    --url: gitlab的地址
    --registration-token: 注册令牌
    --executor: 执行器类型，常用的有shell、docker等
    --description: runner的描述
    --tag-list: 标签列表，用逗号分隔,后面可以在流水线文件中指定使用哪个tag
    --docker-image: 如果使用docker执行器，需要指定docker镜像,优先级比较低,如果在流水线文件中指定了镜像,则使用流水线文件中的镜像
```

注册完成后,在刚刚ruuner页面即可看到

##  编写.gitlab-ci.yml

在项目根目录下创建`.gitlab-ci.yml`文件,这是gitlab的流水线配置文件,可以参考[官方文档](https://docs.gitlab.com/ee/ci/yaml/)。

实例
```yaml

# 全局变量
variables:
  PNPM_CACHE_DIR: ".pnpm-store"
  DEPLOY_PATH: "/opt/dockerApps/nginx/html"
  NPM_CONFIG_REGISTRY: "http://192.168.1.11:8081/repository/io-npm-proxy/"
  NPM_AUTH_USER: "username"
  NPM_AUTH_PASS: "password"
  NODE_TLS_REJECT_UNAUTHORIZED: "0"
  HUSKY: "0"
  HUSKY_SKIP_INSTALL: "1"
  DISABLE_HUSKY: "1"
  VITE_CDN: "false"
  NODE_OPTIONS: "--max-old-space-size=8192"
  CI: "true"
  GIT_SSL_NO_VERIFY: 1
  GIT_STRATEGY: clone

stages:
  - build
  - deploy

build:
  stage: build
  # 使用哪个镜像
  image: gitlab-runner/node:20-alpine-build
# 缓存配置,不能使用绝对路径,因为在docker中运行时,绝对路径会被映射到宿主机的路径
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .pnpm-store/
      - node_modules/
# 使用哪个runner,这里使用上面注册的docker-node-build-runner
  tags:
    - node-20
# 运行前的脚本
  before_script:
    - corepack enable
    - corepack prepare pnpm@latest --activate
    - pnpm config set store-dir .pnpm-store
    - |
      echo "registry=${NPM_CONFIG_REGISTRY}
      //192.168.1.11:8081/repository/io-npm-proxy/:_auth=$(echo -n ${NPM_AUTH_USER}:${NPM_AUTH_PASS} | base64)
      always-auth=true
      strict-ssl=false
      shamefully-hoist=true" > .npmrc
# 运行脚本
  script:
    # - pnpm install vite-plugin-cdn-import@1.0.1 --save-dev --ignore-scripts
    - pnpm install --frozen-lockfile --shamefully-hoist --ignore-scripts
    - pnpm run build
# artifacts: 构建产物,这里指定了dist目录,可以在流水线页面下载
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
# 只有在指定的分支上运行
  only:
    - test
    - dev-auto-deploy

deploy:
  stage: deploy
  # 使用shell执行器的runner进行部署
  tags:
    - shell
    - deploy
  script:
    - sudo mkdir -p ${DEPLOY_PATH}
    - ls dist/*
    - sudo cp -r dist/* ${DEPLOY_PATH}/
    - sudo docker restart nasio_nginx
  dependencies:
    - build
  only:
    - main
    - dev-auto-deploy
```

### 说明

#### 结构说明

这份yaml文件还是挺简单的,主要包括stages、variables、jobs等部分。
参考下[官方文档](https://docs.gitlab.com/ee/ci/yaml/)就能写的七七八八.


#### 冒号问题

经过测试,貌似`:`在变量中会导致问题,所以在变量中不要使用`:`，可以使用其他符号代替或者通过单引号包裹起来。

#### 镜像问题

使用docker来构建时,每次都会重新创建一个容器,可以在运行`gitlab-ruuner`的机器上的`/etc/gitlab-runner/config.toml`中修改配置`pull_policy = ["if-not-present"]`，这样就不会每次都拉取镜像了。

```
[[runners]]
  name = "docker-node-build-runner"
  url = "https://dev.iotechina.com:6580/"
  id = 58
  token = "us4ddKTyyP3yGbaVxcRz"
  token_obtained_at = 2025-06-10T15:28:03Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  [runners.cache]
    MaxUploadedArchiveSize = 0
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "node:20-alpine"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    shm_size = 0
    network_mtu = 0
    # 设置缓存目录,将docker容器中的/cache目录映射到宿主机的/opt/gitlab-runner/cache目录
    volumes = ["/opt/gitlab-runner/cache:/cache:rw"]
    # 只有在需要时才拉取镜像
    pull_policy = ["if-not-present"]
```

修改之后重启`gitlab-runner`服务即可:`gitlab-runner restart`。

有时现有的镜像不满足需求，可以通过`docker build`命令构建一个新的镜像，然后在`.gitlab-ci.yml`中使用这个新镜像。
如果国内网络环境下，可以考虑使用[docker镜像](https://github.com/dongyubin/DockerHub)

下载镜像可以使用命如下拉取镜像从而避免修改docker的配置文件。

```bash
docker pull docker.1panel.live/library/node:20-alpine
```

#### 构建产物传递

在gitlab-ci中，构建产物可以通过`artifacts`来传递，指定需要传递的文件或目录。

上面的配置文件中可以看到,`build`阶段的`artifacts`部分指定了`dist/`目录，这样在构建完成后，`dist/`目录中的文件就可以在流水线页面下载。
在`deploy`阶段中，通过`dependencies`指定了依赖于`build`阶段，这样就可以在部署阶段通过`- sudo cp -r dist/* ${DEPLOY_PATH}/
`访问到`build`阶段的产物。

