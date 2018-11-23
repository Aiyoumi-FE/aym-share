## Git环境配置

### 一、 全局配置

#### 1. 配置文件

git全局配置文件.gitconfig默认在当前系统用户文件夹下，window可运行%USERPROFILE%查找，Mac系统在cd ~查找。

具体配置可参考如下，其中：
【user】:  用户提交时显示在log里的信息
【alias】: 常用git命令简写
【core】:  window系统和类linux系统回车键转换
【push】:  默认对应远端（当本地分支名与远程分支名不一致有用）

```
[user]
    name = hoby
    email = hoby@github.com
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    pl = pull --rebase
    ps = push
    mg = merge --no-ff
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
[core]
    autocrlf = input
    ignorecase = false
[push]
    default = upstream
```

若要使用git mergetool功能，再增加配置（cmd处可修改成本地文件比较工具如beyond compare）：

```
[merge]
    tool = sourcetree
[diff]
    tool = sourcetree
[difftool "sourcetree"]
    cmd = 'C:/Program Files/TortoiseGit/bin/TortoiseGitMerge.exe' \"$LOCAL\" \"$REMOTE\"
[mergetool "sourcetree"]
    cmd = 'C:/Program Files/TortoiseGit/bin/TortoiseGitMerge.exe'  -base:\"$BASE\" -mine:\"$LOCAL\" -theirs:\"$REMOTE\" -merged:\"$MERGED\"
    trustExitCode = true
```

#### 2. 命令修改
如果不加--global 修改的是当前项目git配置。
```bash
$ git config --global --list // 查看全局配置
$ git config --global user.name hoby // 修改提交名
$ git config --global alias.br branch // 修改简写
$ git config --unset alias.co // 删除配置项
$ git config --global core.ignorecase false // 关闭忽略大小写
```


### 二、 Git密钥配置

#### 1. 单项目自动化clone
如果想快速简单的clone一个git项目，在拉取时直接带上用户名密码，此方法在服务器自动化构建时经常用到，具体实现如下：
```
$ git clone http(s)://hoby:123456@githbub.com/hoby/xxx.git
```
> 需要注意的是：上面的hoby是用户名而非邮箱，比如hoby@qq.com的用户，只需要用hoby加密码即可

#### 2. 以SSH方式全局配置密钥

**1.）检查SSH Key存在**
如果存在id_rsa.pub 或 id_dsa.pub 文件，跳过此步。
```
$ cd ~/.ssh    // 其实就在用户目录下有个.ssh的文件夹
$ ls
```

**2.）创建SSH Key**
创建ssh key时会提示自定名称和push时的密码（不是git登录密码），一般推荐略过，直接三个回车，如果创建成功会出来一个有图案的小框框。
```
$ ssh-keygen -t rsa -C "your_email@example.com"  // 此处email可任意，不一定要gitLab登录邮箱
```

**3.）查看SSH Key**
copy公钥内容到gitLab里，添加进去。
```
$ cat ~/.ssh/id_rsa.pub
```

**4.）测试SSH Key**
```
$ ssh -T "git@xxx.xxx.com"
```

**5.）配置多个网站ssh密钥**
在生成每个网站ssh-key时，自定义名称不要一样，然后在~/.ssh目录下新建一个config文件，然后配置多个网站的ssh信息，内容如下：
```
# gitLab
Host dev.gitLab.com
HostName dev.gitLab.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa

# phabricator
Host 192.168.1.5
HostName 192.168.1.5
Port 22
PreferredAuthentications publickey
IdentityFile ~/.ssh/phabricator
```


#### 3. 以HTTP(S)方式全局配置密码

http(s)创建的项目，操作时总是提示用户名和密码，不胜其烦，以下方案可以让Git永远记住用户名与密码：

**1.）添加HOME变量**
在Windows用户变量中添加一个HOME环境变量，值为%USERPROFILE%，如下图：
![img](http://pic002.cnblogs.com/images/2011/1/2011070615112192.jpg)

**2.）创建_netrc文件**
在"开始>运行"中打开%Home%，新建一个名为_netrc的文件，输入Git服务器名、用户名、密码保存，可放多个不同登录信息的Git项目，中间空一行即可，具体内容如下：
```
machine dev.github.com
login hoby
password 123456
```


## Git常用命令

### 一、仓库管理

#### 1. 初始化

初始化后，目录下会生成.git隐藏文件夹，图标显示git可控，.git目录下有一个hooks，里面可以对预提交信息做一些规范和限制。
```
$ git init
```


#### 2. 添加/修改仓库地址

a.) 添加关联地址：
```
$ git remote add origin http://dev.github.com:9800/root/front.git
```

b.) 修改关联地址：
```
$ git remote set-url origin http://dev.github.com:9800/root/front.git
```

c.) 切换http(s)/ssh方式：
```
$ git remote rm origin
$ git remote add origin "git远程地址"
$ git push origin
```

#### 3. 查看当前仓库地址

```
$ git remote -v
```

#### 4. 创建本地仓库

```
$ git clone http://dev.github.com:9800/root/front.git
```
会在当前目录下自动生成front的仓库目录，如果自定义文件夹名，后面空格后加上名称


### 二、分支管理

#### 1. 查看分支
```
$ git br    // 查看本地

$ git br -r // 查看远端

$ git br -a // 查看所有
```

#### 2. 查看分支状态（推荐）

可全面查看本地分支状态，远端对应关系，落后/优先多少，最后一次提交信息及hash值
```
$ git br -vv
```

#### 3. 创建分支

a.) 创建本地分支：

本地分支名可不与远程一致，简单一点，方便Tab键补全，只要.gitconfig里的[push]项设置default = upstream即可，不然提交时，会提示本地与远程不一致
```
$ git br <branch> <origin-branch> // 创建分支，并关联远端
或：
$ git co -b <branch> <origin-branch> // 创建分支并立即切换过去
```

b.) 创建远程分支：（本地分支推送到远端）
```
$ git push origin -u <branch>:<origin-branch> // -u：创建远程分支并关联，origin-branch不用带origin/路径
```

#### 4. 切换分支

```
$ git co <branch>

# 切换到游离分支（不在任何分支）
$ git fetch
$ git co FETCH_HEAD
```

#### 5. 合并分支

a.) 合并本地分支：
```
$ git merge <branch> --no-ff  // --no-ff：关闭Fast-Foward合并，这样可以生成merge提交，保留分支合并信息
```
会出来vim编辑窗口(可修改提交信息)，不修改则按shift + : 再输入wq（保存退出）后回车即可。

b.) 合并远端本地(不用先checkout到本地)：
```
$ git fetch // 先更新远程仓库所有分支上最新commit-id到本地
$ git merge origin/<branch> --no-ff  // 再从远端merge，无需先拉取到本地
```

c.) 取消合并：
```
$ git merge --abort
```

d.) 合并冲突处理：
```
$ git mergetool  // 如果在~/.gitconfig下正确配置了比较工具，会依次弹出比较窗口，处理后保存
$ git clean -nfd  // 查看不在版本库中合并文件，以.org结尾
$ git clean -fd   // 确认无误后，移除这个.org临时文件
$ git add -A
$ git merge --continue  // 继续完成merge流程
```

#### 6. 修改分支

a.) 修改分支名称：
```
$ git br -m <branchName>
```

b.) 修改当前分支远端关联：
```
$ git br --set-upstream-to <origin-branch>
```

c.) 修改指定分支远端关联：
```
$ git br --set-upstream <branch> <origin-branch>
```

#### 7. 删除分支

a.) 删除本地分支：
```
$ git br -d <branch>
```

b.) 强制删除本地分支：
```
$ git br -D <branch>  // 未被合并的分支被删除的时候需要强制
```

c.) 删除远端分支：
```
$ git push origin --delete <branch>
或：
$ git push origin :<branch>  // 推送一个空分支到远端，即删除
```

#### 8. 更新分支

从远端拉取最新的分支信息，更新本地缓存
```
$ git fetch -p   // 更新被删除的分支
```


### 三、查看管理

#### 1. 查看当前状态

```
$ git st  // 查看当前工作状态
```

#### 2. 查看提交记录

a.) 查看全部提交记录：
```
$ git lg
```

b.) 查看单个文件所有记录：
```
$ git lg <fileName>
```

c.) 查看单个文件修改详情：
```
$ git lg -p <fileName>  // 可查看比较文件历史修改记录
```

d.) 查看单条记录详情：
```
$ git show <commit-id>  // 日志的hash值
```

e.) 查找所有包括指定字符串文件：
```
$ git grep -n <string>  // -n：显示字符串所在的行数
```

#### 3. 查看比较文件

a.) 查看比较文件：（与暂存区比较）

如果当前文件没被修改，需要查看历史修改情况，用上面的git lg -p fileName
```
$ git diff <fileName>  // 比较当前文件和暂存区文件差异，应用在提交前确认的场景
```

b.) 比较两次提交记录：
```
$ git diff <commit-id1> <commit-id2> // 比较两次提交之间的差异
```

#### 4. 查看历史操作记录
每一次当前HEAD发生改变（包括切换branch, pull, 添加新commit）一个新的纪录就会被添加到reflog
```
$ git reflog
```


### 四、操作管理

#### 1. 更新

a.) 非快进更新：
```
$ git pull --no-ff
```

b.) 基于rebase：

这个命令会迫使git将远程分支上的变更同步到本地，然后将尚未推送的提交重新应用到这个最新版本，就好象它们刚刚发生一样，这样就可以避免合并以及随之而来的丑陋的合并信息
```
$ git pull --rebase   // 避免产生无用的合并信息
```
例：当前状态为 * master  c7bcf10 [origin/master: ahead 1, behind 1] * [cps-bk] add a.js
此时用--rebase只产生1 commit，而普通的git pull会产生2 commits，其中一条是没必要的合并信息。

若--rebase冲突了，解决方案：
```
$ git add -A   // 强制添加到
$ git rebase --continue
$ git push
```

c.) 临时更新：（暂存）
```
$ git stash      // 先放入暂存区
$ git pull
$ git stash pop  // 恢复显示工作内容，或用git stash apply stash@{n}挑选恢复哪个
```


d.) 管理暂存：
```
$ git stash list    // 显示暂存列表
$ git stash clear   // 清除暂存列表
```

#### 2. 提交

a.) 多个修改，只提交部分：
```
$ git add <file1> <file2>   // 只添加要提交的到本地暂存区
$ git ci -m 'commit info'   // -m处不加a
$ git push
```

b.) 提交全部：
```
$ git add .               // 有新文件时，一定要
$ git ci -am 'commit info'
$ git push
```

c.) 追加提交：（未push到远端） 
将最近一次的变更追加到最新的提交，同时也可以编辑提交信息，不产生新的提交记录。
```
$ git ci --amend   // 如果已push到远端，不建议追加，容易跟自己冲突
$ git push
```

d.) 提交时绕过pre-commit验证：
```
git ci -am 'commit info' --no-verify
```

#### 3. 提取
从别的分支同步一个commit到本分支
```
$ git cherry-pick <commit-id>

// 提取一个merge提交
$ git cherry-pick -m <parent-number> <commit-id>  // parent-number为主线序号，一般是第一个，也即是1
```


### 五、恢复管理

#### 1. 恢复本地

a.) 已修改，未暂存： 
```
$ git co <fileName|directory>  // 恢复单个文件或目录
```
```
$ git co .  // 恢复全部
```

b.) 已暂存，未提交：
```
$ git reset HEAD^   // 向前回滚一条记录，相当于HEAD~1
$ git co .
```

c.) 恢复前N条：
```
$ git reset HEAD~n   // 默认省略--soft参数，属于软恢复，n>=1整数
```

d.) 硬恢复：
```
$ git reset --hard HEAD~n|<commit-id>   // 不保留当前修改，要当心，确认当前修改没有用了
```

#### 2. 恢复远端

a.) 单次回滚： 
git revert 会产生一个新的与之前commit相反的操作，来抵消之前错误的提交。

1.) 回滚一个普通commit
```
$ git revert <commit-id>   // 也可以多次操作，达到恢复多个的目的
$ git push
```
2.) 回滚一个merge
```
$ git show <merge-id>   // 查看merge的commit-id，有一行显示：Merge: 96db08bb0 3669a27a5

// 核对一下以哪个为主线（主线的内容将会保留，而另一条分支的内容将被revert），若第一个为准，parent-number就是1
$ git revert -m <parent-number> <merge-id>
$ git push
```
> 值的注意的是：如果刚才revert掉的内容，后面如果要再合并过来，需要在之前执行revert所在的分支上再执行一次revert，否则会造成那次的内容丢失（git记忆功能）:
git revert <revert-commit-id>

b.) 指定位置：
```
$ git reset --hard <commit-id>   // 恢复到当前位置
$ git push -f      // 要加-f强制推送
```

c.) 挑选模式：
```
$ git rebase -i <commit-id>  // 会打开编辑，剔除挑选记录
$ git push -f      // 要加-f强制推送
```


### 六、其它命令

#### 1. 临时忽略文件

a.) 忽略跟踪：
.gitignore只能忽略那些原来没有被跟踪的文件，如果文件已被纳入版本管理中，则修改.gitignore是无效的。
```
$ git update-index --assume-unchanged <file>
```

b.) 恢复跟踪： 
```
$ git update-index --no-assume-unchanged <file>
```

c.) 查看被忽略的跟踪：
```
$ git ls-files -v|grep '^h'
```

d.) 另一个类似的功能：
```
$ git update-index --skip-worktree <file>  // 忽略

$ git update-index --no-skip-worktree <file>  // 恢复
```


#### 2. 清理

a.) 移除版本控制：
```
$ git rm <file> --cached    // -r：移除目录， --cached：保留本地的文件
```

b.) 清理工作树：
```
$ git clean -fd    // 移除未跟踪的文件和目录，如果加上-n参数来先看看会删掉哪些文件
```

c.) 垃圾回收：

Git 往磁盘保存对象时默认使用的格式叫松散对象(loose object)格式，当手动执行git gc 命令，或推送至远程服务器时，Git会将这些对象打包至一个叫packfile的二进制文件以节省空间并提高效率。
```
$ git gc --auto
$ git repack -d -l  
```

#### 3. 帮助

```
$ git help
```

或查看某一条指令参数：
```
git <command> -n
```


### 七、错误解决方案

#### 1. linux服务器clone时报git密钥0644权限问题：
```
WARNING: UNPROTECTED PRIVATE KEY FILE!
```
说明私钥权限太大，设为只有所有者有读和写以及执行的权限。

```
chmod 700 id_rsa*
```
