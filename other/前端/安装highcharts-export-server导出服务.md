## 安装highcharts-export-server 导出服务

> 版本选择

```
node 版本:8.4.0
npm  版本:6.4.0
```

### 1.安装node/npm
```
1.下载安装包
https://npm.taobao.org/mirrors/node/ 

2.解压
tar -xvf

3.设置软连接
ln -s /home/resin/opt/node/8.4.0/node-v8.4.0-linux-x86/bin/node /usr/local/bin/node

ln -s /home/resin/opt/node/8.4.0/node-v8.4.0-linux-x86/bin/npm /usr/local/bin/npm
```



### 3.安装highcharts-export-server
```
##没有安装git可以子啊本地下载之后上传
git clone https://github.com/highcharts/node-export-server
npm install
npm link
```

### 4.修改cli.js

```
#!/usr/bin/env node
修改为
#!/usr/bin/env /usr/local/bin/node

```