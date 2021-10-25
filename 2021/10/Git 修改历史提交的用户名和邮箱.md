# Git 修改历史提交的用户名和邮箱

## 一、裸需要修改的仓库

```bash
# 仓库地址 .git 可以不要
# git clone --bare https://github.com/qinweizhao/仓库地址.git
git clone --bare https://github.com/qinweizhao/qwz-spring-boot-sample.git 
# 进入该目录， 注意有 .git 
cd qwz-spring-boot-sample.git 
```

## 二、修改本地项目的邮箱和用户名

```bash
# !/bin/sh
git filter-branch --env-filter '
OLD_EMAIL="老的邮箱"
CORRECT_NAME="新用户名"
CORRECT_EMAIL="新邮箱"
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

例子

```bash
git filter-branch --env-filter '
  OLD_EMAILA="qinweizhao1997@126.com"
  OLD_EMAILB="qinweizhao1997@163.com"
  CORRECT_NAME="YVKG"
  CORRECT_EMAIL="yvkg@qq.com"
  if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAILA" ]
  then
      export GIT_COMMITTER_NAME="$CORRECT_NAME"
      export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
  fi
  if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAILA" ]
  then
      export GIT_AUTHOR_NAME="$CORRECT_NAME"
      export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
  fi 
  if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAILB" ]
  then
      export GIT_COMMITTER_NAME="$CORRECT_NAME"
      export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
  fi
  if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAILB" ]
  then
      export GIT_AUTHOR_NAME="$CORRECT_NAME"
      export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
  fi
  ' --tag-name-filter cat -- --branches --tags
```

## 三、提交修改

```bash
git push --force --tags origin 'refs/heads/*'
```

如果再次执行脚本，则会抛出异常

>Cannot create a new backup.
>A previous backup already exists in refs/original/
>Force overwriting the backup with -f

因为执行过一次 **git filter-branch**，所以在 **refs/original/** 有一个备份，只要删掉那个备份即可。

```bash
git update-ref -d refs/original/refs/heads/master
```

