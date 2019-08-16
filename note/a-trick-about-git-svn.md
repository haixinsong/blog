# git svn 慢死了怎么办

[^_^]: # (url:a-trick-about-git-svn)
[^_^]: # (tag:git,note,#tech)
[^_^]: # (excerpt:git svn 快速克隆大型svn项目的最新代码)

因为近期实习的公司使用了SVN, 必不可免我需要拉取SVN上面的项目. 然而我不想安装古老的SVN软件,恰好git对SVN有支持,那么自然我决定使用git处理它.  
但是当我兴冲冲地开始clone代码之后, 我等了很久, 甚至于两小时之后我吃完晚饭回来, 仍然看不见它有完成clone的迹象. 就此打住, 我需要另寻出路.  
事实证明我放弃等待的决定是明智的. 对于公司级别的代码库,一般都会比较大,用git svn拉取全部代码历史极其耗时.  
下面介绍如何用git快速拉取大型SVN项目的代码.

## 解决方案

使用以下命令, 即可快速拉取最新的代码.

```bash
git svn init http://your.svn.path/some/project # 从svn初始化git仓库
git svn fetch -r HEAD # 从svn拉取最新的代码
```

以下方式思路相同,clone操作可以使用参数-r$REVNUMBER:HEAD检出指定版本后的代码,需要先获取版本号`(git svn info http://your-svn )`,略繁琐  
(Notice! 下述方式笔者未验证,不保证其完全正确)

```bash
git svn fetch -r svn_number # 从svn拉取对应版本的代码
git svn clone -r 250:HEAD http://your.svn.path/some/project # 拉取从版本250开始,一直到最新代码的所有版本信息
```
