---
title: springboot整合RabbitMQ（windows环境）
date: 2018-08-09 17:01:33
tags:
---

### 一、安装 Erlang和RabbitMQ Server ，启动RabbitMQ Server

**1、安装Erland，通过[官方下载页面](http://www.erlang.org/downloads)获取exe安装包，直接打开并完成安装。(好像要翻墙~自己解决)**
**2、安装完Erland，配置好它的环境变量，变量值是你自己的安装地址，然后再在path变量加入它**

> %ERLANG_HOME%\bin

![这里写图片描述](http://pd6p053wl.bkt.clouddn.com/erland%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F.png)

![这里写图片描述](http://pd6p053wl.bkt.clouddn.com/e%E5%8F%98%E9%87%8F2.jpg)

**3、然后再安装[RabbitMQ服务端](https://www.rabbitmq.com/download.html),注意如果下载高版本的，对erland的版本也是有要求的，当然你两者都下最新的就没问题了。**


**4、依然是配置好RabbitMQ的环境变量，和在path变量里新加**

> %RABBITMQ_SERVER%\sbin

![这里写图片描述](http://pd6p053wl.bkt.clouddn.com/r1.jpg)
**5、然后打开命令行（注意要用管理员的身份打开，不然后面会提示错误~）然后输入**

> rabbitmq-plugins.bat enable rabbitmq_management
<!--more-->
**开启Web管理插件，这样我们就可以通过浏览器来进行管理了，一般情况下这时候已经自动启动RabbitMQ服务了，我们也可以手动启动验证一下，输入**

> net start RabbitMQ

**如果是下图则已启动**
![这里写图片描述](http://pd6p053wl.bkt.clouddn.com/%E5%B7%B2%E5%90%AF%E5%8A%A8r1.jpg)
**要关闭服务的话，输入**
>net stop RabbitMQ

**6、打开web界面，地址 http://localhost:15672/  默认的用户名：guest 默认的密码为：guest，登录后你就可以看到下图了**
![这里写图片描述](http://pd6p053wl.bkt.clouddn.com/r%E7%9A%84%E9%A1%B5%E9%9D%A2.jpg)

**你可以点击Admin去创建一个新的用户，我下面配置文件就会用到新的用户的用户名跟密码**

-------------------------
### 二、springboot整合rabbitMQ

**1、新建一个springboot工程，并在`pom.xml` 中引入与rabbitmq有关的依赖**
``` xml
  <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
``` 
**2、修改application.yml,配置关于RabbitMQ的连接和用户信息（新建的工程配置文件不是这个名字，改成它即可。。。）**
```yml
spring:
  application:
    name: rabbitmq-test
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin
    
```

**3、创建RabbitMQ的配置类`RabbitConfig`**
```
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * RabbitMQ的配置类
 */
@Configuration
public class RabbitConfig {
    @Bean
    public Queue helloQueue() {
        return new Queue("hello");
    }
}
```
**4、我们建立一个消息生产类`Sender`，作为生产者的身边**
``` java
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Date;
/**
 * 消息生产类
 */
@Component
public class Sender {
    @Autowired
    private AmqpTemplate amqpTemplate;

    public void send() {
        String context = "hello " + new Date();
        System.out.println("Sender : " + context);
        amqpTemplate.convertAndSend("hello", context);
    }

}

```

**5、再建立一个消息消费类`Receiver`，作为消费者**

``` java
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * 消息消费类
 */
@Component
@RabbitListener(queues = "hello")
public class Receiver {

    @RabbitHandler
    public void process(String hello) {
        System.out.println("Receiver : " + hello);
    }
}
```

**6、最后再创建一个测试类`RabbitMQTest`**

```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class RabbitMQTest {
    @Autowired
    private Sender sender;

    @Test
    public void testRabbitMQ(){
        sender.send();
    }
}
```

**7、先启动主程序`RabbitmqApplication`,我这边碰到一个错误**
![](http://pd6p053wl.bkt.clouddn.com/%E7%AC%AC%E4%B8%80%E6%AC%A1%E8%BF%90%E8%A1%8C%E5%A4%B1%E8%B4%A5.jpg)
**原来是没给admin这个用户设置管理队列的权限，去set一把即可**
![](http://pd6p053wl.bkt.clouddn.com/%E8%AE%BE%E7%BD%AEuser%E6%9D%83%E9%99%90.jpg)

**再次运行主程序，成功连接RabbitMQ**
![](http://pd6p053wl.bkt.clouddn.com/%E8%BF%9E%E6%8E%A5ra%E6%88%90%E5%8A%9F.jpg)

**8、去run我们的测试用例方法，看到控制台输出，表示成功**
![](http://pd6p053wl.bkt.clouddn.com/%E5%8F%91%E9%80%81%E6%B6%88%E6%81%AF.jpg)
**再切换回主程序，也可以看到输出日志**
![](http://pd6p053wl.bkt.clouddn.com/%E6%94%B6%E5%88%B0%E6%B6%88%E6%81%AF.jpg)
**当然我们在RabbitMQ的页面上也可以看到队列的消费信息记录**
![](http://pd6p053wl.bkt.clouddn.com/%E5%90%8E%E5%8F%B0%E7%9C%8B%E5%88%B0.jpg)

--------------
#### 以上便是springboot对RabbitMQ的消息生产和消费的最简单的整合，想了解更多请查阅[RabbitMQ](https://www.rabbitmq.com/)官网