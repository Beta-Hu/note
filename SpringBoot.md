# SpringBoot快速入门
- 导入maven依赖
  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>3.0.6</version>
  </parent>

  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
  </dependencies>

  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
      </plugins>
  </build>
  ```
- 编写控制器
  ```java
  @RestController
  public class HelloController {
      @RequestMapping("/hello")
      public String toHello(){
          return "Hello World!";
      }
  }
  ```
- 编写引导类
  ```java
  @SpringBootApplication
  public class HelloApplication {
      public static void main(String[] args) {
          SpringApplication.run(HelloApplication.class, args);
      }
  }
  ```

# 起步依赖原理
- spring-boot-start-parent
  - 定义了各种技术的版本，通过依赖继承获取父项目的依赖

# 配置文件
- 文件名必须是application
- 支持的格式(优先级从高到低，同名属性以高优先级为准)
  - properties
    - 通过xx.xx表示所属关系 
  - yml
    - 通过缩进表示所属关系
  - yaml
    - 通过缩进表示所属关系
- YAML
  - 要求
    - 大小写敏感
    - 缩进只能使用空格，不允许tab
    - 同级缩进相同即可，具体空格数不做要求
    - 冒号后必须要使用空格隔开
  - 数据格式
    - 对象
      ```yml
      person: 
       name: zhangs
      person: (name: zhang)
      ```
    - 数组
      ```yml
      address:
       - bj
       - sh
      ```
    - 纯量
      ```yml
      msg1: 'dadsadada'
      msg2: "dadsada"
      ```
