## java上传文件至hdfs

### 用java上传文件至hdfs时报错：

>报错:

```
java.io.IOException: Filesystem closed
```

>报错原因：

```
多个datanode在getFileSystem过程中，由于Configuration一样，会得到同一个FileSystem。如果有一个datanode在使用完关闭连接，其它的datanode在访问就会出现上述异常
```

> 解决方案：

1.改代码 使用newInstance方法
```
//error
FileSystem  fs = FileSystem.get(URI.create(hdfsFilePath), conf);

//right 
FileSystem  fs = FileSystem.newInstance(URI.create(hdfsFilePath), config);

```

2.修改配置
在hdfs core-site.xml里把fs.hdfs.impl.disable.cache设置为true

