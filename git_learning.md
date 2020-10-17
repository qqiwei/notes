#### learning [git (**zh-cn**)](https://www.liaoxuefeng.com/wiki/896043488029600)  

```shell
# get started
git config --global user.name "qqiwei"
git config --global user.email "youremail@example.com"
```

download [tortoisegit](https://tortoisegit.org/download/), a GUI tool

```shell
# commonly
git status

git add/rm ./certain_file
git commit -m "description"
```

```shell
# personalization configurations
git config --global alias.resetback "reset HEAD^"
git config --global alias.logpretty "log --color --graph --pretty=format:'%Cred%h%Creset \
-%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.logoneline "log --color --graph --pretty=oneline --abbrev-commit"
```

```shell
# add ~/.ssh/id_rsa.pub into your github
ssh-keygen -t rsa -C "youremail@example.com"

# create a new repo, and link the local branches to the remote 
git clone https://github.com/qqiwei/notes.git
git push -u origin main_the_local_branch
git remote -v

git remote add origin git@github.com:qqiwei/notes.git
git branch --set-upstream-to=origin/main main
git branch -r 
git branch -vv
git branch -d branch_to_be_removed
git swtich another_branch
git branch a_new_branch_locally

git reflog
```

```shell
# merge branches
git merge another_branch_to_be_merged_into_the_current_branch
# rebase
git pull
git rebase
# stash
git stash list/pop/drop/apply
# pick
git cherry_pick commit_id
git checkout another_path path_here 

# tags
git tag/branch
git tag/branch new_tag/new_branch
git push origin --tags/tag_name/branch_name
git tag/branch -d tag_to_be_removed/branch_to_be_removed
git push origin :ref/tags/tag_to_be_removed_remotely

git tag new_tag commit_id
git tag -a new_tag -m "description" commit_id
git show tag_name
```

