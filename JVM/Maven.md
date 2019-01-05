# Maven 知识点
## 1 使用mvn install命令把jar包安装到本地maven库中
```
mvn install:install-file -Dfile=alipay-sdk-1.5.jar -DgroupId=com.alipay -DartifactId=sdk -Dversion=1.5 -Dpackaging=jar
```