# CentOS7 

## 常用命令

- htop
- rz

## ab压力测试命令
- ab
```
ab -n 5 -c 1 http://gw.jiangtai.com/prod-svc/api/v1/attr/19PR357306/tour
```





```
git log  --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat:  --since ==2019-3-24 --until=2019-4-24 --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
```