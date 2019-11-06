git fork后如何同步源的新更新


1、配置上游项目地址。即将你 fork 的源项目的地址给配置到自己的项目上。比如我 fork 了一个项目，原项目是 wabish/fork-demo.git，我的项目就是 cobish/fork-demo.git。使用以下命令来配置。
➜ git remote add upstream https://github.com/wabish/fork-demo.git
然后可以查看一下配置状况，很好，上游项目的地址已经被加进来了。
➜ git remote -v
origin  git@github.com:cobish/fork-demo.git (fetch)
origin  git@github.com:cobish/fork-demo.git (push)
upstream    https://github.com/wabish/fork-demo.git (fetch)
upstream    https://github.com/wabish/fork-demo.git (push)

2、获取上游项目更新。使用 fetch 命令更新，fetch 后会被存储在一个本地分支 upstream/master 上。
➜ git fetch upstream

3、合并到本地分支。切换到 master 分支，合并 upstream/master 分支。（可能需要解决冲突）
➜ git merge upstream/master

也可选择rebase
git rebase upstream/master

4、提交推送到自己fork出来的项目上。
➜ git push origin master

