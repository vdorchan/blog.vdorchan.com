---
title: 使用 Travis-CI 完成高级的自动化部署
date: 2019-04-18 10:32:01
tags:
---

> 会啰嗦几句简单介绍下然后实战

写代码经常要花很多时间在构建和部署上面，像是我的个人网站、博客系统或者是一些小项目，每次有些改动就得去重新构建生产代码，改完之后还要把它弄上服务器，更新到线上去。

项目一多，改动一多，于是乎懒惰的我，就会把它“堆起来”，等一个比较长的开发周期结束之后，再去更新线上。虽然这样减少了更新的次数，但依然是耗费时间。有一天我觉得身为高贵的程序员不能再总是亲手干这种活了，必须要找个苦力给我搞定它。

通过 Google 发现 Travis CI 是最合适的选择。

## 什么是 CI

CI 即*持续集成服务*，是 Continuous Integration 的简称，而 Travis CI 是提供这种服务里面市场份额最大的那个。

> 之前开发 Electron 应用，需要 windows 环境构建，但因为 Travis CI 不支持 windows 环境，还一起用过 appveyor。appveyor 也是提供 CI 服务的。不过后来看到新闻，说 Travis CI 开始支持 windows 了，不过我还没试过。

持续集成就是在团队开发的时候，成员们持续（频繁）将代码改动集成到主干上去。而每次集成都是通过自动化的构建（包括编译，发布，自动化测试）来验证。

持续集成的好处在于，每次代码的小幅变更，就能看到运行结果，从而也能尽早的发现集成错误。这样子就能不断累积小的变更，而不是在开发周期结束时，一下子合并一大块代码。

Travis CI 和 Github 账号绑定，你可以选择需要持续集成的项目，之后只要这个项目有代码变动，就会自动抓取，然后提供一个运行环境，执行测试，完成构建，然后也能部署到服务器上去。

## .travis.yml

在 [官网](https://travis-ci.com/)，点击右上角的个人头像，可以使用 Github 账户登入 Travis CI。然后便可以选择需要同步的项目。

这个同步的项目必须要有一个 `.travis.yml` 文件，是 Travis.yml 的配置文件，文件格式是 `YAML` 格式。它指定了 Travis 的行为。该文件必须 push 到 Github 仓库里面，一旦代码仓库有新的 Commit，Travis 就会去找这个文件，执行里面的命令。一般它是下面这样的

```YAML
language: python
script: true
```

上面代码设置了两个字段，`language`字段指定了默认运行环境，这里设定使用 `Python` 环境。`script`字段指定要运行的脚本，`script: true` 表示不执行任何脚本，状态直接设为成功。

Travis CI 的完整生命周期为（参考[官方文档](https://docs.travis-ci.com/user/job-lifecycle/#the-job-lifecycle)）：

1. 可选的 Install `apt addons`
2. 可选的 Install c`ache components`
3. `before_install`
4. `install`
5. `before_script`
6. `script`
7. 可选的 `before_cache` (for cleaning up cache)
8. `after_success` 或 `after_failure`
9. 可选的 `before_deploy`
10. 可选的 `deploy`
11. 可选的 `after_deploy`
12. after_script

> 注意当使用 `install` 指定脚本的时候，要确保使用 `chmod +x` 给予可执行的权限。

其中 `install` 指定安装脚本， `script` 指定构建或测试脚本。

```YAML
language: python
install:
  - command1
  - command2
script:
  - command1
  - command2
```

`install` 指定的脚本如果有多个，如果前面有脚本执行失败，构建就会停下来并标记为错误，不再往下进行。

而 `script` 则是继续执行后面的脚本，如果你想要在第一个命令失败时不要执行第二个命令，你可以这么写

```YAML
script:
  - command1
  - command2
```

## 构建失败

* 如果 `before_install`, `install` 或者 `before_script` 返回一个非 0 的退出代码，构建会报错并且立刻中断。

* 如果 `script` 返回一个非 0 的退出代码，会在整个 `script` 流程结束后标记会构建失败，在这之前，即使前面命令报错，后面命令也会继续执行。

* `after_success`, `after_failure`, `after_script`, `after_deploy` 和后面的流程不会影响构建结果，但是如果其中又一个流程超时了，构建会被标记会失败。

## 部署 Vue Spa

现在我正在开发一个名为 vue-movie 的 vue 单页面应用，并需要将它部署到服务器。

在使用 CI 工具之前，工作流程大致是

1. 本地通过 `yarn serve` 开发代码
2. 代码开发调试没问题后，执行 `yarn build` 构建生产代码并 `git push` 到 Github 上
3. 只用同步命令 rsync 部署到线上（这里省略 ssh 验证步骤）

现在我要让 CI 服务帮我做 2 和 3

```bash
cd vue-movie
vi .travis.yml
```

.travis.yml 内容如下，包括指定运行环境的语言、指定只构建 master 分支，指定安装命令和构建命令

```YAML
language: node_js
node_js:
- 10
branches:
  only:
  - master
install: npm i -g yarn && yarn && yarn build
script:
- rsync -avu --progress --delete dist/ root@vdorchan.com:/srv/www/proj.vdorchan.com/vue-movie/
```

> rsync [OPTION]... SRC DEST
> -a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性
> -v, --verbose 详细模式输出
> -u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件，不覆盖更新的文件。
> --progress 显示备份过程
> --delete 删除那些DST中SRC没有的文件

上面的 rsync 命令将项目的 dist 目录和远程服务器上的 /srv/www/proj.vdorchan.com/vue-movie/ 目录进行同步，远程服务器地址为 vdorchan.com

### ssh

当进行远程同步的时候，是通过 ssh 网络协议进行传输的，ssh 以非对称加密实现身份验证。常见的做法是人工生成一对公钥和私钥，通过生成的密钥进行认证，这样就可以在不输入密码的情况下登录。

除了默认添加的 github.com，gist.github.com 或 ssh.github.com，和其它服务器进行 ssh 传输时，都需要手动生成密钥。

如果你使用的电脑有向你的 github 仓库 push 过代码，那么你应该使用过下面的命令生成过密钥了

```bash
ssh-keygen -t rsa -C "vdorchan@gmail.com"
```

-t 指定密钥类型，-C 为注释，换成你自己的邮箱即可

期间会问一些东西，使用默认值，一路回车就可以了，然后进入用户主目录下的 .ssh 文件夹，你就能看到 `id_rsa` 和 `id_rsa.pub` 两个文件，id_rsa 是私钥，绝对不能泄漏。id_rsa.pub 则是配套的公钥。

```bash
$ cd ~/.ssh
$ ls
id_rsa id_rsa.pub
```

然后要将你的公钥 `id_rsa.pub` 放到服务器上去

客户端执行，然后复制

```bash
cat id_rsa.pub
```

服务器执行，创建文件，并编辑粘贴公钥内容

```bash
vi ~/.ssh/authorized_keys
```

### travis 加密文件

在 CI 环境里要想和服务器通信，那么就要把私钥放到 CI 环境里，但前面说过，私钥不能泄露，所以也就不能直接放在 github仓库里，travis CI 官方提供了加密文件的工具。

首先需要安装 travis 命令行工具，Mac os 用户可以直接使用 brew 安装

```bash
brew install travis

# 或
gem install travis
```

可参考[官方仓库](https://github.com/travis-ci/travis.rb)

在项目目录下，使用 travis 的加密文件功能，将私钥进行加密。

```bash
cd vue-movie
travis encrypt-file ~/.ssh/id_rsa --add
```

这时候目录下就会出现 `id_rsa.enc` 这么一个文件，并且在 .travis.yml 文件中会加入下面这些代码

```YAML
before_install:
- openssl aes-256-cbc -K $encrypted_0ad490829f5d_key -iv $encrypted_0ad490829f5d_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
```

上面代码是利用生成的文件和环境变量进行解密得到密钥。

如果有些默认生成的代码是这样子的话，会报错，因为多了 `\`，去掉就好

```YAML
 -out ~\/.ssh/id_rsa -d

 # 去掉 “ \ ” 即可
 -out ~/.ssh/id_rsa -d
```

## 阻止询问公钥

当第一次使用 ssh 传输文件的时候，系统会进行询问

```bash
The authenticity of host ’35.241.118.163 (35.241.118.163)’ can’t be established.
ECDSA key fingerprint is 60:9c:a8:53:07:52:b9:b4:9f:fb:a2:57:53:be:5e:00.
Are you sure you want to continue connecting
```

因为 CI 是不运行交互式提示的，所以要阻止它询问

```YAML
addons:
  ssh_known_hosts: vdorchan.com
```

上面代码将你的服务器域名添加上去

### 你可能会遇到这样的权限问题

```bash
Permissions 0664 for ‘/home/travis/.ssh/id_rsa’ are too open.
```

补充如下代码即可解决

```YAML
before_install:
- openssl aes-256-cbc -K $encrypted_0ad490829f5d_key -iv $encrypted_0ad490829f5d_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
```

完成上述的操作之后，你的代码应该就能自动化的部署了。不是很难，确保每个步骤无误，就肯定没问题了。如果还有问题，可以 Google、看官方文档，也可以问我。

代码可见 [vue-movie](https://github.com/vdorchan/vue-movie)