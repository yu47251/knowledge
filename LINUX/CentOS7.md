# CentOS7 

## 常用命令

- htop
- rz
```
获取文件, 从linux以外往linux发送文件
```
- sz
```
发送文件, 从linux机器往外发送
```
- pwgen
```
例: pwgen -c -n 10 -y
生成一个10位数字的密码, -y:包含最少一个特殊字符, -c:至少包含一个大写字母, -n:至少包含一个数字
```

## ab压力测试命令
- ab
```
ab -n 5 -c 1 http://gw.jiangtai.com/prod-svc/api/v1/attr/19PR357306/tour
```