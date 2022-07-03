



# iTerm2



## 简介

iTerm2 是 Terminal 的替代品，也是 iTerm 的继承者。它适用于装有 macOS 10.14 或更新版本的 Mac。 iTerm2 将终端带入**现代**，具有您从未想过的功能。



## 安装

- https://iterm2.com/downloads.html



## 配置

### 主题

#### 1.1 下载

- 使用git

​	git clone https://github.com/dracula/iterm.git

- 手动

​	从`Github`下载压缩包[download](https://github.com/dracula/iterm/archive/master.zip) ，并解压



#### 1.2 配置

	1. 打开 *iTerm2 > Preferences > Profiles > Colors Tab*
	1. 右下角打开 *Color Presets...*
	1. 选择 *Import...* 
	1. 选择 `Dracula.itermcolors` 文件
	1. 然后*Color Presets...*选择 *Dracula* 主题



### OH-MY-ZSH

> 终端增强器，官网: https://ohmyz.sh/#install

#### 2.1 安装

```shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```



#### 2.2 主题 [https://draculatheme.com/zsh](https://draculatheme.com/zsh)

1. 下载

   ```shell
   $ mkdir /Users/wei/Documents/dracula
   $ cd /Users/wei/Documents/dracula
   $ git clone https://github.com/dracula/zsh.git
   ```

   

2. 创建环境变量

   ```shell
   # dracula 主题的绝对路径或者相对路径
   $ echo "export DRACULA_THEME=/Users/wei/Documents/dracula" >> ~/.zshrc
   $ source ~/.zshrc
   ```

3. 创建软连接

   ```shell
   $ ln -s $DRACULA_THEME/zsh/dracula.zsh-theme $ZSH/themes/dracula.zsh-theme
   ```

4. 修改 `vim ~/.zshrc`，切换主题为 `dracula`



#### 2.3 git alias

- 常用

  ```shell
  gd - git diff
  gl - git pull
  ggpush - git push origin $(current_branch)
  gm - git merge
  ga - git add
  gcm - git checkout master
  gco - git checkout
  gb - git branch
  glog - git log --oneline --decorate --color --graph
  gss - git status -s
  gsta - git stash
  gstp - git stash pop
  ```

  

- 全部

  ```shell
  g - git
  gst - git status
  gl - git pull
  gup - git pull --rebase
  gp - git push
  gd - git diff
  gdc - git diff --cached
  gdv - git diff -w "$@" | view
  gc - git commit -v
  gc! - git commit -v --amend
  gca - git commit -v -a
  gca! - git commit -v -a --amend
  gcmsg - git commit -m
  gco - git checkout
  gcm - git checkout master
  gr - git remote
  grv - git remote -v
  grmv - git remote rename
  grrm - git remote remove
  gsetr - git remote set-url
  grup - git remote update
  grbi - git rebase -i
  grbc - git rebase --continue
  grba - git rebase --abort
  gb - git branch
  gba - git branch -a
  gcount - git shortlog -sn
  gcl - git config --list
  gcp - git cherry-pick
  glg - git log --stat --max-count=10
  glgg - git log --graph --max-count=10
  glgga - git log --graph --decorate --all
  glo - git log --oneline --decorate --color
  glog - git log --oneline --decorate --color --graph
  gss - git status -s
  ga - git add
  gm - git merge
  grh - git reset HEAD
  grhh - git reset HEAD --hard
  gclean - git reset --hard && git clean -dfx
  gwc - git whatchanged -p --abbrev-commit --pretty=medium
  gsts - git stash show --text
  gsta - git stash
  gstp - git stash pop
  gstd - git stash drop
  ggpull - git pull origin $(current_branch)
  ggpur - git pull --rebase origin $(current_branch)
  ggpush - git push origin $(current_branch)
  ggpnp - git pull origin $(current_branch) && git push origin $(current_branch)
  ```

  



#### 2.4 zsh-autosuggestions

> 下载地址：[https://github.com/zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

1. 克隆项目，并安装至 `$ZSH_CUSTOM/plugins` (一般都在 `~/.oh-my-zsh/custom/plugins`)

   ```shell
   git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
   ```

   

2. 修改 `vim ~/.zshrc`，增加插件 `zsh-autosuggestions`

   ```shell
   plugins=( 
       # other plugins...
       zsh-autosuggestions
   )
   ```

   

3. 加载配置文件

   ```shell
   source ~/.zshrc
   ```

   

