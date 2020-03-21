## 全部记录都clone
git svn clone svn地址  --no-metadata

## 根据版本号记录克隆
git svn clone -r 299663:HEAD svn地址  --no-metadata


## 增加远程仓库
git remote add origin git地址


git remote add origin git地址




## 先更新远程仓库
> 后面加上 --allow-unrelated-histories ， 把两段不相干的 分支进行强行合并
git pull origin master --allow-unrelated-histories

## 提交将本地的master分支推送到origin主机
> -f 强制覆盖
git push -u origin master -f 


分支规范
1.master主分支
2.develop开发
