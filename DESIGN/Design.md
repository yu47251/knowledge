# 设计模式
## 单例模式
### 基础版
```java
public class Singleton{
    private static Singleton singleton = new Singleton();
    private Singleton(){
    }
    public static Singleton getInstance(){
        return singleton;
    }
}
```
### 懒加载
```java
public class Singleton{
    private static Singleton singleton;
    private Singleton(){
    }
    public static Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}
```
### 线程安全版
```java
public class Singleton{
    private static Singleton singleton;
    private Singleton(){
    }
    public static synchronized Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}
```
### 线程安全，双check版
```java
public class Singleton{
    private static Singleton singleton;
    private Singleton(){
    }
    // sync块内部得检测if，是为了防止有两个进程，同时判断singleton == null进入内部
    public static Singleton getInstance(){
        if(singleton == null){
            synchronized(Singleton.class){
                if(singleton == null){
                    singleton = new Singleton();                        
                }                    
            }
        }
        return singleton;
    }
}
```
### 内部类方式
```java
public class Singleton{
    private Singleton singleton;
    private Singleton(){
    }

    private static class SingletonHandler(){
        private static Singleton instance = new Singleton();
    }

    public Singleton getInstance(){
        return SingletonHandler.instance;
    }
}
```
### 枚举版
```java
public class Singleton{
    public enum EasySingleton{
        INSTANCE;
    }
}
```
# 设计原则
## 单一职责
## 开闭原则
## 里氏替换原则
## 依赖倒置原则
## 接口隔离原则
## 迪米特原则