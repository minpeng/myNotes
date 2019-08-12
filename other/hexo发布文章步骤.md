## hexo发布文章步骤

### 1.进入到bolg目录执行创建文章命令
```
hexo new "SpringBoot事务解析.md"
```

### 2.编辑文章


###３.部署文章到github

```
hexo clean //清除缓存文件 (db.json) 和已生成的静态文件 (public)
hexo g //生成缓存和静态文件
hexo d //重新部署到服务器

```