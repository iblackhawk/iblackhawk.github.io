# Ubuntu配置ssh免密

```
root权限下
1.ssh localhost        生成本地文件
2.cd ~/.ssh            进入ssh目录下
3.ssh-keygen           生成密钥
4.cat id_rsa.pub > authorized_keys   将密钥重定向到授权文件中
5.ssh 本机hostname      完成免密登录
```

