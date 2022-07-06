初始化git

```bash
git init
```

克隆远程仓库

```bash
git clone <远程仓库地址>
git clone --branch <branch name> <远程仓库地址> #拉取特定分支
```

推送到远程仓库

```bash
git push origin <branch name>
#如果新建的temp分支没有上游分支，则在上游（github）新建temp分支再push过去
git push --set-upstream origin temp
```

文件加入暂存区

```bash
git add <FileName>
或全部添加
git add *
```

代码回滚

```bash
git reset --hard <commit id>
```

### 设置个人信息
在没有GPG签名情况下 user.email 决定了github的commit的人是谁。（甚至可以假装成Linus）
```bash
git config --global user.name <"你的匿名">
git config --global user.email <"你的邮箱">
#也可以在当前项目里设置信息，局部优先于global
git config user.name <"你的匿名">
git config user.email <"你的邮箱">
```

### 分支操作

```bash
# 根据某个分支commit id创建新的分支
git branch <new branch name> <commit id>
# 查看分支
git branch
# 删除分支
git branch -d <branch name>
# 切换分支
git checkout <branch name>
```

### 查看各种信息

```bash
git status # 查看文件状态
git branch # 查看分支
git reflog # 本地操作（commit、checkout、rebase等）历史查看
git log # 项目当前分支指向结点的commit历史
```

### git remote origin数据源

```bash
# 查看当前remote origin
git remote --verbose
# 关联远程仓库
git remote add <origin> <ssh或https地址>
# 更换remote中origin例子
git remote set-url origin git@github.com:hanyisong/testGPGSigned.git
```

### commit自动署名

```bash
git commit --signoff -m 'message content'
#加上GPG signed
git commit -s -S -m 'message content'
```

如果忘记署名，可以通过下面命令编辑commit消息

```bash
git commit --amend -s # --amend修改之前的commit -s签名
```

### 合并commit

参考[git 合并commit - 码农教程 (manongjc.com)](http://www.manongjc.com/detail/25-jeukveeivdswfjm.html)



1、查看commit历史

```bash
git log #项目当前结点的commit历史
```

#### 第一种合并commit（rebase变基）

```bash
# 没有指定endpoint默认为当前分支HEAD所指向的commit，可以HEAD~x，也可以具体的commit id（git reflog或git log查看）
git rebase -i [startpoint] [endpoint] # 变基将startpoint设为他们新的基础结点（原来是HEAD）

#比如当前commit有 HEAD@{0}、HEAD@{1}、HEAD@{2}、HEAD@{3}、HEAD@{4}，序号越大越早存在
#合并前3个（可git reflog查看x编号），实际合并的是 HEAD@{0}、HEAD@{1}、HEAD@{2}
git rebase -i HEAD@{3} #可以push到远程main分支上
#合并HEAD@{2}、HEAD@{3}（注意！会丢失HEAD@{0}、HEAD@{1}，因为基点由HEAD变为HEAD@{4}）
git rebase -i HEAD@{4} HEAD@{2} #会变成一个分离的结点，导致不能push到远程main分支上
```

3、然后就自动进入了编辑界面，把要并入的commit都将pick改成squash，具体修改方法git有给出。（谁并入谁）

4、接下来就是修改commit消息，把之前的commit消息注释掉即可。

5、推送到远程仓库。

```
# feat-xxx为你的分支，-f是强制force update谨慎使用 -f 提交，因为会覆盖别人的代码
git push origin feat-xxx -f 
```

#### 第二种合并commit(软回退)

软回退，就是将提交信息回退到指定commitid，但是代码会被放在暂存区。如上我们可以这般

```bash
git reset --soft <commit id>
git log #发现commit log 也回去了
```

虽然回退了，但是莫慌，看看你代码，还是最新的，代码并没有回退。代码已经被保存到暂存区了

然后这个时候再重新commit和强推即可

```bash
git commit -m 'feat: 将几次修改readme的提交合并成一个'
git push -f
```

如果不强推报以下错误

```bash
! [rejected]  main -> main (non-fast-forward)
error: failed to push some refs to 'https://github.com/xxxx/test.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```



### Tips

1、commit id可以简短的5位或6位

2、HEAD@{x} == HEAD~x == commit id（x 需要指向该id）

### other

![20200227084538507](C:\Users\han\Desktop\workspace\git\20200227084538507.png)