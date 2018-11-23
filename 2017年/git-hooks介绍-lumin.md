# Git Hooks介绍

什么是Git Hooks？从名称上理解是钩子，在git操作过程，在特定事件发生之前或之后执行特定脚本代码功能，类似于触发器的概念。 
 
按照Git Hooks脚本所在的位置可以分为两类：

- 本地Hooks，触发事件如commit、merge等。
- 服务端Hooks，触发事件如receive等。


### Git Hooks能做什么？
Git Hooks是定制化的脚本程序，所以它实现的功能与相应的git动作相关，可以编写脚本实现我们想要的功能，比如代码检查，提交规范，自动测试，自动部署等，举几个常用到的例子：

1. **pre-commit**: 检查每次的commit message是否有语法错误，或是否符合某种规范，会运行jshint或者eslint。
2. **commit-msg**: 检查提交git时comment的格式，必须符合特定规范格式，如爱又米前端的规范{action} [{module}] {description}
3. **post-receive**: 在服务端接听，每当有新的提交的时候可以做一些持续集成的工作

看到这些上面的例子应该有了大概的了解。

### Git Hooks是怎么工作的？
当新建一个git项目时执行git init会自动生成一个.git文件夹，文件夹下包含一个hooks目录，这里面就是放置hooks的地方，默认目录下的hooks都是以.sample结尾的，这些hooks不生效，如果要生效，将**.smaple**去掉即可。 

我们可以使用任意脚本语言（bash，pyhton，perl等）来编写自己的hooks，以下是爱又米commit-msg的hooks，我需要提交git的comment需要遵循一定的规范，比如:

* `+ [web] #AXD-5467，添加活动页面文件`
* `* [config] 修改jslint规则，允许var后面允许不空行`
* `* [app] fixed #YYHD-908，修改为空的判断`
* `- [app,h5] remove unused images`


hooks/commit-msg
```
#!/usr/bin/env bash

# Validate commit log
commit_regex='^Merge.+|[+*-] \[((app|h5|cps-h5|cps-site|app,h5|site|cps-bk|cps-ft|cps-pay|vue|doc|config|other|test|web|aicai-openapi)(,)?)+\] (\{\autobuild\})?[^\n]{4,50}'

if ! grep -iqE "$commit_regex" "$1"; then
    echo "ERROR: commit log format is not correct!"
    echo "using {action} [{module}] {description} "
    echo "format: $commit_regex"
    echo "{auto:前端分支:cps分支}"
    exit 1
fi
```
```bash
git commit -am '提交信息测试'
```

当我们提交commit时，因为有了commit-msg的hooks，所以会触发执行commit-msg的脚本，并且因为和脚本中的正则表达式不匹配，所以提交不过。

### 提交工作流Hooks
这四个钩子涉及提交的过程，会顺序触发。

- pre-commit 
钩子在键入提交信息前运行。 它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。 如果该钩子以非0值退出，Git 将放弃此次提交，不过你可以用 git commit --no-verify 来绕过这个环节。 你可以利用该钩子，来检查代码风格是否一致（运行类似 lint 的程序）、尾随空白字符是否存在（自带的钩子就是这么做的），或新方法的文档是否适当。

- prepare-commit-msg 
钩子在启动提交信息编辑器之前，默认信息被创建之后运行。 它允许你编辑提交者所看到的默认信息。 它对一般的提交来说并没有什么用；然而对那些会自动产生默认信息的提交，如提交信息模板、合并提交、压缩提交和修订提交等非常实用。 你可以结合提交模板来使用它，动态地插入信息。

- commit-msg
如果该钩子脚本以非0值退出，Git 将放弃提交，因此，可以用来在提交通过前验证项目状态或提交信息。 在本章的最后一节，我们将展示如何使用该钩子来核对提交信息是否遵循指定的模板。

- post-commit
钩子在整个提交过程完成后运行。 它不接收任何参数，但你可以很容易地通过运行 git log -1 HEAD 来获得最后一次的提交信息。 该钩子一般用于通知之类的事情。

### 服务器端Hooks
除了客户端钩子，作为系统管理员，你还可以使用若干服务器端的钩子对项目强制执行各种类型的策略。 这些钩子脚本在推送到服务器之前和之后运行。 推送到服务器前运行的钩子可以在任何时候以非零值退出，拒绝推送并给客户端返回错误消息，还可以依你所想设置足够复杂的推送策略。

- pre-receive
处理来自客户端的推送操作时，最先被调用的脚本是 pre-receive。 它从标准输入获取一系列被推送的引用。如果它以非零值退出，所有的推送内容都不会被接受。 你可以用这个钩子阻止对引用进行非快进（non-fast-forward）的更新，或者对该推送所修改的所有引用和文件进行访问控制。

- update
update 脚本和 pre-receive 脚本十分类似，不同之处在于它会为每一个准备更新的分支各运行一次。 假如推送者同时向多个分支推送内容，pre-receive 只运行一次，相比之下 update 则会为每一个被推送的分支各运行一次。如果 update 脚本以非零值退出，只有相应的那一个引用会被拒绝；其余的依然会被更新。

- post-receive
post-receive 挂钩在整个过程完结以后运行，可以用来更新其他系统服务或者通知用户。 它接受与 pre-receive 相同的标准输入数据。 它的用途包括给某个邮件列表发信，通知持续集成（continous integration）的服务器，或者更新问题追踪系统（ticket-tracking system） —— 甚至可以通过分析提交信息来决定某个问题（ticket）是否应该被开启，修改或者关闭。 该脚本无法终止推送进程，不过客户端在它结束运行之前将保持连接状态，所以如果你想做其他操作需谨慎使用它，因为它将耗费你很长的一段时间。

### 其他git Hooks
```
applypatch-msg
pre-applypatch
post-applypatch
pre-rebase
post-checkout
post-merge
pre-auto-gc
post-rewrite
```
见：[https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)

### 附：pre-commit的hooks
```
#!/usr/bin/env bash

hasErrors=0
files=$(git diff HEAD --name-only --diff-filter=ACM | grep  "\.js$\|\.vue$" && git ls-files --others --exclude-standard | grep "\.js$\|\.vue$")
hasVueChanged=0
for  file  in $files
do
    if [ ! -e "$file" ]  #检查文件是否存在
    then
        echo "$file does not exist"
        continue
    fi

    echo "开始lint: $file"
    if [[ "$file" =~ "axd-vue" ]]
    then
        hasVueChanged=1
    else
        echo 'jshint'
        lintRet=$(jshint $file | grep 'errors')
    fi
done

#如果有修改的vue文件，则全部检查一遍vue项目
if [ 1 -eq "${hasVueChanged}" ];
then
    echo "开始 eslint vue..."
    cd axd-vue
    lintRet=$(eslint --ext [.js,.vue] ./src  | grep '**errors**' )
fi

echo "${lintRet}"
if [ -n "${lintRet}" ];
then
    hasErrors=1
    echo 'JS lint有错误，请检查！'
fi

exit $hasErrors
```
上面的脚本会对修改过的js文件进行jshint代码检查，如果是axd-vue项目则会全量检查，如果有错误，则返回非0值退出，Git将放弃提交。