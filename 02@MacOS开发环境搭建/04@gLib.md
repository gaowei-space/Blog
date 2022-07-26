



# GLib



## 背景
😣 **合并请求**的诞生之路：打开浏览器 -> 访问gitlab网站 -> 选择项目 -> 访问提交合并请求的页面 -> 点击创建新的合并请求 -> 选择源分支和目标分支 -> 点击提交 -> 输入标题和内容 -> 点击创建合并请求按钮。



也可以通过创建书签，减少一些步骤，但我觉得还是太冗长了，如果有一款软件可以替我们干这些事，岂不乐哉！👏 幸运的是已经有大佬开发了一款**Gitlab CLI**软件，**主角**登场👇



## 简介



[GLib](https://github.com/profclems/glab) 是 gitlab 终端操作软件，支持包括但不限于使用命令提交合并请求等，大大节省了`gitlib`网站操作的步骤，极大的提升了合并代码等操作的效率



## 安装

### MacOS

使用 `Homebrew`, 或者直接下载EXE文件

```shell
$ brew install glab
```

### Windows

使用 [WinGet](https://github.com/microsoft/winget-cli), [scoop](https://scoop.sh/), 或者直接下载EXE文件

#### WinGet

```shell
$ winget install glab
```

#### Scoop

```shell
$ scoop install glab
```

#### Releases

https://github.com/profclems/glab/releases

### 其他

https://github.com/profclems/glab#installation





## 使用

```shell
$ glab <command> <subcommand> [flags]
```



### 1.登录你的`gitlab`网站创建`token`

访问 https://your_gitlab.com/-/profile/personal_access_tokens 创建一个token，赋予 `api` 和 `write_repository` 权限

### 2.授权

执行 `glab auth login` 授权你的 GitLab 账户

```shell
1. 选择 gitlab 站点，私有还是官方网站
? What GitLab instance do you want to log into?  [Use arrows to move, type to filter]
  gitlab.com
> GitLab Self-hosted Instance # 选择私有部署的 gitlab 

2. 输入提供 `web` 的服务域名
? GitLab hostname: your_gitlab.com # 输入你的gitlab网站host，如: your_gitlab.com

3. 输入 `API` 域名
? API hostname: your_gitlab.com

4. 输入上面刚刚获取的 `Token`
? Paste your authentication token: lab_dajsldjlasjdl

5. 选择默认协议
? Choose default git protocol  [Use arrows to move, type to filter]
  SSH
> HTTPS
  HTTP
  
6. 是否授权
? Authenticate Git with your GitLab credentials? (Y/n) Y

7. 选择 `API` 协议
? Choose host API protocol  [Use arrows to move, type to filter]
> HTTPS
  HTTP

8. 抛出下面内容，表示成功了
- glab config set -h your_gitlab.com git_protocol https
✓ Configured git protocol
- glab config set -h your_gitlab.com api_protocol https
✓ Configured API protocol
✓ Logged in as yourname

```



## 核心命令

- `glab mr [list, create, close, reopen, delete, ...]`
- `glab issue [list, create, close, reopen, delete, ...]`
- `glab pipeline [list, delete, ci status, ci view, ...]`
- `glab release`
- `glab repo`
- `glab label`
- `glab alias`



## 使用示例

```shell
$ glab auth login --stdin < token.txt
$ glab issue list
$ glab mr for 123   # Create merge request for issue 123
$ glab mr checkout 243
$ glab pipeline ci view
$ glab mr view
$ glab mr approve
$ glab mr merge
```



## 常用命令

```shell
# 使用当前分支，创建一个合并请求，可以
$ glab mr create -f test/release/master（默认，也可以不输入）
```



## 了解更多

https://glab.readthedocs.io/en/latest/index.html 
