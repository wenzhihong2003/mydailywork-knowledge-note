# git 日常小知识点

### Fork过来的项目拉取源项目的更新
- 在Fork的代码库中添加上游代码库的remote源，其中upstream表示上游代码库名，可以任意：

```
git remote add upstream https://github.com/lj2007331/lnmp.git
```

- 将本地的修改提交 commit

- 在每次 Pull Request 前做如下操作，即可实现和上游版本库的同步。

```
git remote update upstream
git rebase upstream/{branch name}
```

需要注意的是在rebase操作之前，一定要将checkout到{branch name}所指定的branch，

- Push代码


### git本地记住密码
```
git config --global credential.helper store
```
