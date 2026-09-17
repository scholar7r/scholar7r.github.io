+++
title = "使用 Doco CD 实现 Docker GitOps"
date = "2026-09-17T10:04:31+08:00"
author = "scholar7r"
authorTwitter = "scholar7r"
cover = "https://doco.cd/latest/images/doco-cd_logo.svg"
description = "学习 Kubernetes 也有一段时间了，前好些日子给家里的 Home Lab. 上线了 Flux CD 做持续部署。向 GitHub 提交，等待滚动更新确实更加畅快。想到自己还有很多运行在公网上的服务，如果也能做 GitOps，岂不美哉？"
tags = ["墨技"]
+++

## 背景

学习 Kubernetes 也有一段时间了，前好些日子给家里的 Home Lab. 上线了 Flux CD 做持续部署。向 GitHub 提交，等待滚动更新确实更加畅快。想到自己还有很多运行在公网上的服务，都跑在 Docker 中，如果也能做 GitOps，岂不美哉？

说干就干，在 Google Cloud 免费层有一台免费的 E2 Micro，打着不用白不用的原则，一直运行着 Vaultwarden 密码存储。当然，这只是文章中案例，<b>进行迁移之前，需要结合自己此前的部署环境设计迁移步骤。</b>否则，就会有弄丢的数据和抓耳挠腮鞭策 AI 的人。

<!--more-->

## GitOps

首先得了解，什么是 GitOps。看看 Red Hat 是怎么解释它的：

> GitOps 是一套用于管理基础架构和应用配置的实践，旨在扩展现有流程并优化应用生命周期。它使用 Git 存储库作为单一事实来源，以交付基础架构即代码（IaC）。
>
> IaC 是指通过代码（而非手动流程）来管理和置备基础架构。利用 IaC，可以创建包含基础架构规范的配置文件，确保每次都能置备相同的环境。它是实施 DevOps 实践和持续集成/持续交付（CI/CD）的一个重要组成部分。
>
> GitOps 要求对系统的预期状态进行声明式描述。通过使用声明式工具，所有配置文件和源代码都可以在 Git 中进行版本控制。代码的所有变动都会被追踪记录，这不仅简化了更新流程，同时也提供了版本控制功能，以便在必要时进行回滚。
>
> GitOps 可帮助您实现以下目标：
>
> - 采用标准的应用开发工作流。
> - 增强可见性和可审计性，提升安全级别。
> - 利用 Git 提供的可见性和版本控制能力，实现更高的可靠性。
> - 跨集群、云和本地环境保持一致。

粗糙理解，可以认为 GitOps 是一个事件循环。GitOps 控制器会主动监听存储库中对业务部署状态声明的变更，并将本地实际状态收敛到期望状态。由于引入了 Git 版本管理，任何操作可回滚，可溯源。

## Doco CD 单仓库最佳实践

Doco CD 提供了最佳实践案例，下文遵循单仓库最佳实践。

这个结构遵循三个原则：单仓库多栈、按应用隔离、配置与密钥分离。Doco CD 在根目录寻找 .doco-cd.\<stack\>.yaml 作为入口，每个入口对应一个可独立部署的栈；栈自己的 Compose 文件和辅助配置则放在同名应用目录下。

这里的“栈”（stack）指一个可独立部署的服务单元，对应仓库根目录下的一个 .doco-cd.\<stack\>.yaml 入口。

- `.doco-cd.vaultwarden.yaml`：告诉 Doco CD 如何调谐这个栈；
- `compose.yaml`：描述服务本身应该长什么样；
- `litestream.yaml`：描述数据备份策略；
- `vaultwarden.sops.env`：描述敏感运行时配置，但以加密形式进入 Git；
- `.sops.yaml`：规定哪些文件必须加密。

```
.
├── .doco-cd.vaultwarden.yaml
├── .sops.yaml
└── vaultwarden
    ├── secrets
    │   └── vaultwarden.sops.env
    └── vaultwarden
        ├── compose.yaml
        └── litestream.yaml
```

为了避免依赖单一 Doco CD 节点，每台 Docker 主机都运行一个 Doco CD 实例，但每个实例只调谐本节点负责的栈。各节点各自拉取并收敛本地状态；某个节点故障时，在新节点重新部署 Doco CD 并恢复数据即可。

## Doco CD 节点部署

Doco CD 自身也用 Docker Compose 管理。每台 Docker 主机运行一个 Doco CD 实例，通过 Docker Socket 操作宿主机 Docker，并定期轮询 Git 仓库，使本地实际状态向 Git 中的期望状态收敛。

```yaml
# compose.yaml
services:
  doco-cd:
    image: ghcr.io/kimdre/doco-cd:latest
    restart: unless-stopped
    env_file:
      - secrets.env
    environment:
      TZ: Etc/UTC
      POLL_CONFIG_FILE: /etc/doco-cd/poll.yaml
      SOPS_AGE_KEY_FILE: /run/secrets/sops_age_key
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./poll.yaml:/etc/doco-cd/poll.yaml:ro
      - data:/data
    secrets:
      - sops_age_key
    healthcheck:
      test: ["CMD", "/doco-cd", "healthcheck"]
      start_period: 15s
      interval: 30s
      timeout: 5s
      retries: 3

secrets:
  sops_age_key:
    file: ./age.key # chmod 600

volumes:
  data:
```

其中，secrets.env 用于存放访问私有仓库的凭据等运行时变量，例如 PAT；这类信息不进入 Git。

轮询配置如下：

```yaml
# poll.yaml
- url: https://github.com/scholar7r/doco.git
  reference: refs/heads/main
  interval: 30
  target: <STACK>
```

- `url`：要跟踪的 Git 仓库地址。
- `reference`：要跟踪的分支或 ref。
- `interval`：轮询间隔，单位为秒。
- `target`：对应仓库根目录下的 .doco-cd.\<stack\>.yaml，例如 vaultwarden。

## 公共组件

公共组件的 Compose 只维护一份，各 stack 的 .doco-cd.\<stack\>.yaml 按需指向它，并选用该 stack 对应的 secret 文件。这样镜像升级、端口调整等只需改一处，所有引用它的 stack 都会在下次调谐时同步。

```
.
├── beszel
│   ├── beszel
│   │   └── compose.yaml
│   └── secrets
│       └── beszel.sops.env
├── beszel-agent
│   ├── compose.yaml
│   └── secrets
│       ├── beszel.sops.env
│       ├── miniflux.sops.env
│       └── vaultwarden.sops.env
├── .doco-cd.beszel.yaml
├── .doco-cd.miniflux.yaml
├── .doco-cd.vaultwarden.yaml
├── miniflux
│   ├── miniflux
│   │   └── compose.yaml
│   └── secrets
│       └── miniflux.sops.env
├── .sops.yaml
└── vaultwarden
    ├── secrets
    │   └── vaultwarden.sops.env
    └── vaultwarden
        ├── compose.yaml
        └── litestream.yaml
```

## 展望

Doco CD 把 GitOps 带到了 Docker 环境，单仓库、多栈、公共组件、SOPS 加密、多节点调谐这些基础实践已经能覆盖不少场景。

不过，通过前置一个 Ansible 来批量管理节点运行 Doco CD，或许是一个批量初始化的通用实践。
