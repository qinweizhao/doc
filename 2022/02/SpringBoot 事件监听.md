# SpringBoot 事件监听的4种实现方式

自定义事件和自定义监听器类的实现方式：

- 自定义事件：继承自ApplicationEvent抽象类，然后定义自己的构造器
- 自定义监听：实现`ApplicationListener<T>`接口，然后实现onApplicationEvent方法

## 一、向 ApplicationContext 中添加监听器

创建 MyListener1 类，在springboot应用启动类中获取ConfigurableApplicationContext上下文，装载监听

### 1、代码

```java
public class MyEvent extends ApplicationEvent {

    public MyEvent(Object source) {
        super(source);
    }
}
```

```java
public class MyListener1 implements ApplicationListener<MyEvent> {

    @Override
    public void onApplicationEvent(MyEvent event) {
        System.out.println(MyListener1.class.getName() + "监听到事件源：" + event.getSource());
    }
}
```

```java
@SpringBootApplication
public class FsbListenerApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(FsbListenerApplication.class, args);

        // 装载监听
        context.addApplicationListener(new MyListener1());

        // 发布事件
        context.publishEvent(new MyEvent("测试"));
    }

}
```

### 2、运行截图

![2022-02-08_173226](img.qinweizhao.com/2022/02/2022-02-08_173226.png)

## 二、将监听器装载入 Spring 容器

创建 MyListener2 类，并使用@Component注解将该类装载入spring容器中

### 1、代码

```java
// 添加注解
@Component
public class MyListener2 implements ApplicationListener<MyEvent>
{
	
    @Override
    public void onApplicationEvent(MyEvent event)
    {
        System.out.println(MyListener2.class.getName() + "监听到事件源：" + event.getSource());
    }
}
```

### 2、运行截图

![2022-02-08_173440](img.qinweizhao.com/2022/02/2022-02-08_173440.png)

## 三、在 application.properties 中配置监听器

### 1、代码

```java
public class MyListener3 implements ApplicationListener<MyEvent> {

    @Override
    public void onApplicationEvent(MyEvent event) {
        System.out.println(MyListener3.class.getName()+"监听到事件源："+event.getSource());
    }
}
```

```properties
context.listener.classes=com.qinweizhao.listener.MyListener3
```

### 2、运行截图

![2022-02-08_173614](img.qinweizhao.com/2022/02/2022-02-08_173614.png)

## 四、通过@EventListener注解实现事件监听

创建 MyListener4 类，该类无需实现 ApplicationListener 接口，使用 @EventListener 装饰具体方法

### 1、代码

```java
@Component
public class MyListener4 {

    @EventListener
    public void onApplicationEvent(MyEvent event) {
        System.out.println(MyListener4.class.getName() + "监听到事件源：" + event.getSource());
    }
}
```

### 2、运行截图

![2022-02-08_173750](img.qinweizhao.com/2022/02/2022-02-08_173750.png)

