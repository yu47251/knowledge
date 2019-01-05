# Docker 知识点

## 常用命令
- docker run 是使用image进行初始化容器并且启动，如果该image已经有一个container在使用则启动不了。 
- docker ps -a 查看所有进程，包括没有启动的container
- docker start <container-name> 启动container