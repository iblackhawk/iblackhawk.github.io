# hive创建表格时并以指定要求分割

作用：解决空值

> 例：创建mytable表格并“，”为分割点。

```
create table mytable(name string,date string,score int)row format delimited fields terminated by ',';
```

 