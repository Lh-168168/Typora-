# Git拉取

- 第一次创建

```
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Lh-168168/Typora-.git
git push -u origin main
```

- 后续拉取

```
git remote add origin https://github.com/Lh-168168/Typora-.git
git branch -M main
git push -u origin main
```

- 拉取出错的话

```
git branch dev    //创建新分支
git add .				//添加到暂存区
git commit -m 'dev分支的内容'  //提交到版本库，也就是当前分支
git push origin dev     //提交到远程
git checkout main    //切换回main分支
git merge dev       //将分支合并
git push -u origin main    //提交
git branch -D newbranch      //新建分支的朋友别忘了删除这个分支
```

