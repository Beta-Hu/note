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
