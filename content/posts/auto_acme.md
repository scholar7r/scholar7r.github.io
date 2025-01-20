+++
title = 'GitHub Actions 自动化部署续签 SSL 证书'
date = 2024-11-04T19:49:21+08:00
draft = false
categories = '自动化'
tags = ['tls', 'acme.sh']
+++

时值 2024 年，大部分云服务器厂商已经停止了 Let's Encrypt 一年的免费 SSL 证书支持，取而代之的是缩短为三个月的免费证书。以静态个人博客用途为例，虽然 SSL 不是硬性需求，但是作为一个已经解析到域名并上线的网站，只有部署了 SSL 证书，才能得到初步的技术认同。

Inkwell 博客针对中国境内境外做了不同的解析，针对境外的流量，直接通过 CNAME 解析到源站；针对境内的流量，则需要通过阿里云的全站加速 DCDN。阿里云 DCDN 需要对网站配置 SSL 证书，否则无法访问。

<!--more-->

```goat
       ┌────────────┐
       │User request│
       └──────┬─────┘
        ______▽_______      ┌─────────────────────────────┐
       ╱              ╲     │Redirects to Alibaba Cloud   │
      ╱ Is the request ╲____│full site acceleration (DCDN)│
      ╲ from China     ╱yes └──────────────┬──────────────┘
       ╲______________╱          __________▽__________     ┌───────────────────┐
              │no               ╱                     ╲    │Provide full site  │
┌─────────────▽────────────┐   ╱ Has a SSL certificate ╲___│acceleration (DCDN)│
│Redirects to GitHub Pages │   ╲                       ╱yes└───────────────────┘
│source site and GitHub    │    ╲_____________________╱
│Pages provided force HTTPS│               │no
└──────────────────────────┘          ┌────▽────┐
                                      │SSL Error│
                                      └─────────┘
```

这种解析条件下，如果我不对阿里云全站加速配置 SSL 证书信息，那么通过国内流量访问网站则会出现 ERR_SSL_VERSION_OR_CIPHER_MISMATCH 错误。要解决这个错误，唯有配置好 SSL 证书。

三个月的免费证书，每三个月续签一次，这反而让维护网站耗费了大量的成本。要如何通过自动化的手段续签就成了目前的要求。

好在开源项目 [acme.sh](https://acme.sh) 实现了这一需求。此开源项目旨在与自动化 SSL 证书申请流程，并提供多平台的 Hook 实现，可以全自动化从 TXT 解析验证到部署 SSL 证书。

自动化平台选用 GitHub Actions，通过 YAML 配置文件定义 Workflow 工作流程即可实现自动化操作，更重要的是该方法无需任何费用。

要使用 GitHub Actions，必须有一个仓库用来存放 Workflows 文件。创建一个新私有仓库并在 Actions 选项下新建一个 Workflow。

```yaml
name: Acme
on:
  schedule:
    - cron: "0 2 1 * *"
  workflow_dispatch:
  push:
    branches:
      - main

env:
  EMAIL: ${{ secrets.EMAIL }}
  DOMAIN: ${{ secrets.EMAIL }}
  ALI_KEY: ${{ secrets.EMAIL }}
  ALI_SECRET: ${{ secrets.EMAIL }}
```

上述内容是单个 Workflow 的开头部分，通过 `cron` 指定了每月运行一次该任务。`cron` 的内容对应的是 Crontab 定时任务表达式，如果对其并不熟悉，可以使用网络上的开源工具或者是 AI 进行生成。随后 `workflow_dispatch` 指定用户可以手动执行这个任务，避免需要手动执行的某些特定场景问题。最后当 `main` 分支接收到推送操作的时候也会触发这个任务。

Env 项基本负责读取和保存 GitHub Secret 中配置的内容，保证敏感数据不被泄露。上述的所有操作中涉及到敏感操作的为阿里云的 Access 访问代码，如果在高权限状态下泄露将会面临严重的云产品安全问题。

```yaml
jobs:
  Acme:
    runs-on: ubuntu-latest
    steps:
      - name: Install acme.sh
        env:
          EMAIL: ${{ secrets.EMAIL }}
        run: |
          curl https://get.acme.sh | sh -s email="${EMAIL}"
      - name: Check acme.sh version
        run: |
          /home/runner/.acme.sh/acme.sh --version
      - name: Set up variables for Aliyun
        env:
          Ali_Key: ${{ secrets.ALI_KEY }}
          Ali_Secret: ${{ secrets.ALI_SECRET }}
        run: |
          echo "Ali_Key=$Ali_Key" >> $GITHUB_ENV
          echo "Ali_Secret=$Ali_Secret" >> $GITHUB_ENV
          echo "DEPLOY_ALI_DCDN_DOMAIN=$DOMAIN www.$DOMAIN" >> $GITHUB_ENV
      - name: Issue certificate
        run: |
          /home/runner/.acme.sh/acme.sh --issue \
            -d ${DOMAIN} \
            -d *.${DOMAIN} \
            --dns dns_ali
      - name: Deploy to Aliyun DCDN
        run: |
          /home/runner/.acme.sh/acme.sh --deploy \
          -d ${DOMAIN} \
          --deploy-hook ali_dcdn
```

Jobs 下定义了一个 Acme 工程，这个工程会在 Ubuntu 最新版本下运行。下面的每一个 `name` 相当于是一条或多条命令。最主要的是 `Issue certificate` 和 `Deploy to Aliyun DCDN` 环节，前者负责申请证书，后者负责部署证书到阿里云全站加速。

需要注意的是 Acme 的 DNS 验证需要向目标域名的 DNS 解析中插入一条 TXT 解析，如果没有该解析的话 Acme 会提示并尝试插入。尝试插入的前提条件是需要存在环境变量 `Ali_Key` 和 `Ali_Secret`。

```goat

    ┌────────┐
    │Acme Job│
    └────┬───┘
  _______▽_______         ________________
 ╱               ╲       ╱                ╲      ┌─────────────────┐
╱ Successfully    ╲_____╱ Set up variables ╲_____│Issue certificate│
╲ install acme.sh ╱yes  ╲ for Aliyun       ╱yes  └────────┬────────┘
 ╲_______________╱       ╲________________╱    ┌──────────▽──────────┐
         │no                     │no           │Deploy to Aliyun DCDN│
      ┌──▽─┐          ┌──────────▽──────────┐  └──────────┬──────────┘
      │Exit│          │No enough information│        _____▽_____     ┌───────────────────┐
      └────┘          │to continue          │       ╱           ╲    │Please check GitHub│
                      └─────────────────────┘      ╱ There some  ╲___│Secret or commands │
                                                   ╲ thing wrong ╱yes└─────────┬─────────┘
                                                    ╲___________╱              │
                                                          │no                  │
                                                          └─────┬──────────────┘
                                                            ┌───▽──┐
                                                            │Finish│
                                                            └──────┘
```
