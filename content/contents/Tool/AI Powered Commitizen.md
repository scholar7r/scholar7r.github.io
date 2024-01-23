---
title: "AI Powered Commitizen"
date: 2024-01-23T23:12:38+08:00
categories: "工具"
draft: false
---
For someone like me, coding is maybe not always the hardest thing. Writting comments or explain something can always do a large harm to me. So I used to use some commitizen package to generate words automatically. Today we have AI to help us to read changes and produce comments, and a commitizen named `cz-git` that help us to generate regular comments. The first thing to enjoy lazy time is make everything done, so let's prepare the enviroment.

<!--more-->

## Preparations

- Cz-git
- OpenAI token (Or third part)
- Lazygit (Optional)
- Action power and a clever mind

## Install cz-git

> More engineered, lightweight, customizable, standard output format Commitizen adapter and Git commit CLI.

*The steps below maybe outdated oneday, if you wish to install  `cz-git` more correctly, please read [official document](https://cz-git.qbb.sh/).*

**Install global dependencies**

```shell
npm install -g czg
```

**Global configuration adapter type**

```shell
echo '{ "path": "cz-git", "$schema": "https://cdn.jsdelivr.net/gh/Zhengqbbb/cz-git@1.8.0/docs/public/schema/cz-git.json" }' > ~/.czrc
```

## Setup OpenAI token

Login to [OpenAI](https://platform.openai.com/account/api-keys) and create your API secret key, which starts with `sk-`, and then run command `npx czg --api-key=<API secret key>` and input your key to setup your token save to local.

```shell
npx czg --api-key=sk-xxxxx
```

Then, you can use `czg ai` command to generate comment to commit.

## Lazygit (Optional)

When writting this blog, I found that `Lazygit` provided [commitizen integration]( https://github.com/jesseduffield/lazygit/issues/41 ) for so long time, it's worth to have a try. Read more via [Custom Commands Compendium · jesseduffield/lazygit Wiki (github.com)](https://github.com/jesseduffield/lazygit/wiki/Custom-Commands-Compendium#committing-via-commitizen-cz-c).

Just change the commit command, and get cool AI powerd commitizen.

```yaml
customCommands:
  - key: "C"
    command: "git cz ai"
    description: "commit with commitizen"
    context: "files"
    loadingText: "opening commitizen commit tool"
    subprocess: true
```
