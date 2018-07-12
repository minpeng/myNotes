### Mysql 区分大小写


#### 建表时:1:SET utf8 COLLATE utf8_bin
```

`PAGE_URL` ARCHAR(250) CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL COMMENT '页面url'
```

#### 直接修改：2.加binary
```
ALTER TABLE XXX MODIFY PAGE_URL VARCHAR(250) binary COMMENT '页面url';
```