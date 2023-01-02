# How to use git

## basic usage

```shell
git status
git add .
git commit -m "Something comment..."
git push origin master
```

branchを変更しているときはmasterにしないように注意する。

originがついているとリモートリポジトリを意味する


## branch

```shell
git branch
git branch bug-fix/XXX-111-something-worng
git checkout bug-fix/XXX-111-something-worng
# こうするとbranchとcheckoutを同時に行うことができる
# git checkout -b bug-fix/XXX-111-something-worng
```

branchの確認と新規branchの作成

checkoutはbranchの移動を行う


## merge

```shell
git checkout master
git merge bug-fix/XXX-111-something-worng
git push origin master
```

githubのようなものがある昨今はほとんど行わない？別ブランチをpushした後にpull requestを発行する方が主流と思われる。


## add

```shell
git add file.py #対象のファイル
git add folder #対象のフォルダ
git add folder1 folder1 #複数フォルダ
git add *.py #py拡張子全て
git add -A #更新ファイル全て
```

ファイルをステージに乗せる


## commit message

```shell
# gitconfig内に記述
[commit]
  gpgsign = true
  template = ~/.gitmessage
```

templateを設定後は`git commit`のみでコメントを求められる。

```shell
git commit -F- <<EOF
> Some title
> 
> Problem: something
> Solve: something
```

コマンドから複数行のコミットメッセージを入れる場合


## add remote

```shell
git init
git remote add origin https://github.com/userid/yourproject.git
git add .
git commit -m "First commit"
git push -f origin master
```


## stash

```shell
# list
git stash list
# 変更を退避 -u/--include-untracked
git stash -u
# addした変更は退避されない
git stash -k
# コメント付き
git stash save "message"
# 元に戻す
git stash apply <stash name>
# addした状態で戻す
git stash apply stash@{0} --index
# 退避した作業を消す
git stash drop stash@{0}
# 退避した作業を戻すと共にstashリストから消す
git stash pop stash@{0}
# 変更の確認
git stash show stash@{0} -p
# 退避した作業を全て消す
git stash clear
```


### 誤ったbranchで変更した場合(non stage/commit)

```shell
git stash save "worng branch"
git checkout <new branch>
git pop stash@{0}
```


### 誤ったbranchで変更した場合(stageに乗っている場合)

`git reset --soft HEAD^` を行ってからstashする


### 誤ったbranchで変更した場合(commitした場合)

mergeしてからmaster branchをreset --hardする


## reset

```shell
git reset --hard HEAD^ #コミットを取り消しワークディレクトリも元に戻す
git reset --soft HEAD^ #コミットを取り消しワークディレクトリの内容はそのまま
git commit --amend -m "Something comment..." #コミットの上書き
git push -f origin master
```

```shell
git push -f origin HEAD^:master
git reset --soft HEAD^
# Fix a mistake
git add something
git commit -m "some comment"
git push origin master
```

## revert

```bash
git revert f33a2bc -n #-nはコメント編集なし. -eコメント編集あり
git push -f origin master
```


## commitの整理

```shell
# commitを統合するケース
rebase rebase -i <commit id>
# fixup/squashでコミットを統合
```

```shell
# commitを一度リセットしcommitを組み替える
git reset --soft <commit id>
```

master branchへのforce pushは禁止した方がよい


## diff

```shell
git diff
# addした後に差分を確認
git diff --cached
git diff --cached <file>
git diff -- <file path>
gut diff branchA..branchB -- <file_path>
# commitとの差分
git diff HEAD^
# commit同士の比較
git diff 変更前SHA...変更後SHA
# branchの比較
git diff branchA..branchB
# 変更の量
git diff --stat
```


## following repository

```bash
$ git clone https://github.com/user/project
$ cd project
$ git branch -a
$ git remote add upstream https://github.com/author/project # Get from original project
$ git fetch upstream
$ git merge upstream/master
```


## submodule

```shell
# add submodule
git submodule add https://github.com/user/project.git directory
# git submodule with clone
git submodule add https://github.com/user/project.git directory --recursive
# forgot recursive
git submodule update --init
# get submodule new info
git submodule foreach git fetch
# update
git submodule update --remote <submodule path>
```

```shell
# delte
git submodule deinit -f sub-module
git rm -f sub-module
git config -f .gitmodules --remove-section submodule.sub-module
git commit -a
```


## git patch作成

```shell
# Case1 bashrc
gpta() { git reset --hard HEAD^ && git am --3way ./patch/$@ }
# Case2 gitconfig
patch = format-patch HEAD^ -o ./patch
```


## git sign

```bash
#!/bin/bash

PATCH_FILE_NAME=$1
SPLIT_FILE_NAME=(${PATCH_FILE_NAME//./ })
PATCH_SIGNED_FILE_NAME="${SPLIT_FILE_NAME[0]}-signed.${SPLIT_FILE_NAME[1]}"

git am --3way ./patch/${PATCH_FILE_NAME}
git interpret-trailers \
  --trailer "author: $(git log --pretty='%an <%ae>' -1)" \
  --trailer "sign: $(git config user.name) <$(git config user.email)>" \
  ./patch/${PATCH_FILE_NAME} > \
  ./patch/${PATCH_SIGNED_FILE_NAME}
```

```shell
# gitconfig内に記述
[trailer "author"]
  key = "Co-authored-by: "

[trailer "sign"]
  key = "Sign-off-by: "
```

あまり記憶にないがpatchに対してsign offできるスクリプト


## commit default editor

```shell
# gitconfig内に記述
[core]
  editor = vim
```


### get git log

```bash
# add your .bashrc
alias gitlog='git log --date=short --no-merges --pretty=format:"%cd %s %h (@%cn) "'
```


## .gitconfig

```bash
ammend = "!f () { git commit --ammend -m \"$1 ($default)\" $2;}; f"
cm = "!f () { git commit -m \"$1\" $2;}; f"
```


## diff log

```bash
#!/bin/sh

AUTHOR="bluelief"

LINES=$(git log --all --numstat --pretty="%H" --author="${AUTHOR}" --since="midnight" | awk 'NF==3 {plus+=$1; minus+=$2} END {printf("+%d, -%d\n", plus, minus)}')

MESSAGE="[insertions, deletions]: \\n ${LINES}\\n"

printf "${MESSAGE}"
```

`git log --all --numstat --pretty="%H" --author="bluelief" --since="" --until="" | awk 'NF==3 {plus+=$1; minus+=$2} END {printf("+%d, -%d\n", plus, minus)}'`


## git branch on terminal

```bash
function parse_git_branch {
    git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ [\1]/'
}

# Tested on Ubuntu
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\e[1;32m\]$(parse_git_branch)\[\033[00m\]\$ '
```


## blank commit

```shell
alias gitinit="git commit --allow-empty -m 'Init commit'"
```

一番最初に空コミットを行うことで戻りやすくする。


## change mod

```shell
alias gitchmod="git update-index --add --chmod=+x"
```

Change file or directory file mode(file permission)
