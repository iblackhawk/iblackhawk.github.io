```
关于python配置目录  /hadoop-streaming
/opt/hadoop-2.6.0/share/hadoop/tools/lib

```



```
例：hadoop jar hadoop-streaming-2.6.0.jar -file /home/blackhawk/map.py -mapper map.py -file /home/blackhawk/red.py -reducer red.py  -input /input/dat0203.log.tem -output /output
```

> 一定要加上"-file"参数，否则Job会失败
>
> 参数一：-mapper  运行mapper文件
>
> 参数二：-reducer 运行Reducer文件
>
> 参数三：-input   <path>     DFS input ``file``(s) ``for` `the Map step`
>
> 参数四：-output  <path>     DFS output directory ``for` `the Reduce step`