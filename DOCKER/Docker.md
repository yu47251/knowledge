# Docker 知识点

## 常用命令
- docker run 是使用image进行初始化容器并且启动，如果该image已经有一个container在使用则启动不了。 
- docker ps -a 查看所有进程，包括没有启动的container
- docker start <container-name> 启动container
- docker rm <container-id> 删除某一个容器
- docker start/stop/restart <container-id> 启动/停止/重启容器
- docker inspact <container-id> 查看某一个容器详细内容



### docker启动logstash
```
docker run -it -d -p 5044:5044 --name logstash -v /data/app/logstash/logstash.yml:/data/app/logstash/logstash.yml -v /data/app/logstash/conf.d/:/data/app/logstash/conf.d/ elastic/logstash:6.3.2
```

### docker 启动kibana
```
docker run --name elastic/kibana -e ELASTICSEARCH_URL=http://10.10.32.219:9400 -p 5601:5601 -d elastic/kibana:6.3.2
```

