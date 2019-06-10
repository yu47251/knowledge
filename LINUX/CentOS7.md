# CentOS7 

## 常用命令

- htop
- rz

## ab压力测试命令
- ab
```
ab -n 5 -c 1 -p ins-list-sequip.json -T 'application/json' http://gw-sequip-dev.jiangtai.com/ins-svc/applicationforms/sequip
```





```
git log  --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat:  --since ==2019-3-24 --until=2019-4-24 --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
```