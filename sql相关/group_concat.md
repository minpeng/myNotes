### group_concact 

#### 长度有限制 1024
```
 show variables like "%concat%";
 
```

#### 修改长度

```
 SET GLOBAL group_concat_max_len=1024000;
```
     