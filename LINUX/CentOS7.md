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

- tar
```

```

## ab压力测试命令
- ab
```
ab -n 5 -c 1 http://gw.jiangtai.com/prod-svc/api/v1/attr/19PR357306/tour
```

- 带请求头
```
ab -c10  -n100  -H 'userCode:20UC1929599390141123416'  -H 'branchCode:JT0000'  -H 'authorization:Bearer eyJhbGciOiJIUzI1NiJ9.eyJhIjoiODk1NkY4ODZDNzk0QTUyMzdFNjA5MEY0M0UyNkM5QjMiLCJiIjoiMjBVQzE5Mjk1OTkzOTAxNDExMjM0MTYiLCJjIjoiMTlSTDE3MjU4NTEwMDM1MTkyNDAyMDEsMjBSTDE5Mjk1NzY4NjczMzI2MjE0MTgsMjBSTDE5Mjk1NzY4NjczMzI2MjEzOTksMjBSTDE5Mjk1NzY4NjczMzI2MjEzODAsMjBSTDIxODEzNjMyNjA1NjY3OTkyMjUiLCJkIjoic3lzX2Nhc2VfbWFuIiwiZSI6ImNhc2UiLCJmIjoiMSIsImV4cCI6MTU5Mzc2OTcyNn0.Cesc9kAbL7lXGEmsPe3Mk8H8xUHL9IszhhBBojx7eBA'   http://case-web-test.jiangtai.com/_gw/case-svc/deal/reportRecord/reportNo
```

## 查看是否安装了rpm包
rpm -qa |grep nginx

## 删除yum安装的rpm
