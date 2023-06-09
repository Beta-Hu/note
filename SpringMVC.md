# RequestMapping
- 功能: 用于关联一个请求和**唯一的**处理该请求的控制器方法
- SpringMVC关注的是RequestMapping的属性，而非被注解修饰的方法名。
- 属性
  - value
    - String[], 必填
    - 依据(**任一**)地址进行匹配
  - method
    - RequestMethod.GET | RequestMethod.POST, 可(多)选
    - 在地址匹配后依据方法匹配
    - GET与POST的区别
      - GET: 在地址后追加键值对。不安全。快速。容量小。不适用文件传输
      - POST: 使用单独的请求体。安全。低效。容量大。可用于文件传输。
    - _如果使用了put或delete，则默认按照GET处理_
  - params
    - String[], 可选
    - 在依据路径匹配后依据请求的参数匹配请求映射(**全部满足**)
    - 表达式形式
      - "param": 要求请求必须含有参数param
      - "!param": 要求请求必须不含参数param
      - "param=value": 要求必须含有参数param，且要求请求的参数param必须为value
      - "param!=value": 要求必须含有参数param，且要求请求的参数param必须不为value
  - headers
    - String[], 可选
    - 在依据路径匹配后依据请求头进行匹配
    - 表达式类型(与params相同)
- 模糊匹配
  - ?: 匹配任意单个字符(路径分隔符"/"除外)
  - \*: 匹配任意0个或多个字符(路径分隔符"/"除外)
  - \*\*: 匹配任意一层或多层路径。此时只能使用/\*\*/xx_path的形式
- 路径占位符
  - 功能: 将值作为路径的一部分传递到控制器(代替了?key=value的传参方式)
  - 获取: 使用{}和@PathVariable来获取占位符提供的值。支持多个占位符和多种类型
    ```java
    @RequestMapping("/toHello/{id}/target")
    public String toHellllo(@PathVariable("id") Integer id){
      System.out.println("id = " + id);
      return "hello";
    }
    ```
    
# SpringMVC获取请求参数
- 通过ServletApi获取
  ```java
    @RequestMapping("/testServletApi")
    public String testServletApi(HttpServletRequest request){
        String name = request.getParameter("name");
        String password = request.getParameter("password");
        System.out.println("name: " + name + ", password: " + password);
        return "hello";
    }
  ```
- 通过控制器形参的方式获取
  ```java
    // 形参名与请求属性名一致即可获取
    @RequestMapping("/getParamByController")
    public String getParamByController(String name, String password){
        System.out.println("name: " + name + ", password: " + password);
        return "hello";
    }
  ```
- POJO: 请求参数与类属性名称一致的实体类,旨在解决请求参数过多的问题
  - 必须要求参数与类属性名称一致
  ```java
  @RequestMapping("/")
  public String testPOJO(User user){
    System.out.println(user);
    return "success";
  }
  ```
  - 可以使用多个形参，但这往往没有意义

# 域对象共享数据
- 向请求域共享数据
  - 使用原生servletAPI
  ```java
  @RequestMapping("/testRequestByServletApi")
  public String testRequestByServletApi(HttpServletRequest request){
      request.setAttribute("textContext", "Hello Success!");
      return "success";
  }
  ```
  - 通过ModelAndView(推荐的方式)
  ```java
  @RequestMapping("/testModelAndView")
  public ModelAndView testModelAndView(){
      ModelAndView mov = new ModelAndView();
      mov.addObject("textContext", "Hello testModelAndView!");  // 设置请求域属性
      mov.setViewName("success");   // 设置视图名称
      return mov;   // 返回ModelAndView对象，而非之前方法返回的视图名称
  }
  ```
  - 通过Model
  ```java
  @RequestMapping("/testModel")
  // 传入模型，返回视图名称
  public String testModel(Model model){
      model.addAttribute("textContext", "Hello testModel!");
      return "success";
  }
  ```
  - 通过Map集合
  ```java
  @RequestMapping("/testMap")
  public String testMap(Map<String, Object> map){
      map.put("textContext", "Hello testMap!");
      return "success";
  }
  - 通过ModelMap
  ```java
  @RequestMapping("/testModelMap")
  public String testModelMap(ModelMap modelmap){
      modelmap.addAttribute("textContext", "Hello testModelMap!");
      return "success";
  }
  ```
- 向session域共享数据
  - 使用原生servletApi
  ```java
  @RequestMapping("/testSession")
  public String testSession(HttpSession session){
      session.setAttribute("textContext", "Hello testSession!");
      return "success";
  }
  // 注意: thymeleaf获取session属性需要使用"session."来指明范围，例如th:attr_name="${session.xxx}"
  ```
- 向application域共享数据
  - 使用原生servletApi
  ```java
  @RequestMapping("/testApplication")
  public String testApplication(HttpSession session){
      ServletContext context = session.getServletContext(); // 通过session获取application
      context.setAttribute("textContext", "Hello testApplication!");
      return "success";
  }
  ```

# 视图
- 分类: 转发视图、重定向视图
- Thymeleaf视图: 不含视图前缀时的默认视图类型
- 转发视图(控制器方法设置的视图名以forward:开头): InternalResourceView
 ```java
  @RequestMapping("/testForward")
  public String testForward(){
      return "forward:/testThymeleafView";
  }
  ```
- 重定向视图(控制器方法设置的视图名以redirect:开头): RedirectView
  ```java
  @RequestMapping("/testRedirect")
  public String testRedirect(){
      return "redirect:/testThymeleafView";
  }
  ```
- 视图控制器view-controller
  - 存在于servlet配置文件中，path控制路径，view-name控制资源
  ```xml
  <!-- xmlns:mvc="http://www.springframework.org/schema/mvc" -->
  <mvc:view-controller path="/" view-name="portal"/>
  <!-- 开启注解驱动，启用控制器中的请求映射。否则会出现404错误 -->
  <mvc:annotation-driven/>
  ```
  - 一般仅用于实现页面跳转

# RESTFul
- 提倡url使用统一的风格设计，从前向后使用斜杠分割，不使用问号和键值对方式携带请求参数
  ```java
  @Controller
  public class UserController {
      @RequestMapping(value = "/user", method = RequestMethod.GET)
      public String getUsers(){return "test_rest";}

      @RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
      public String getUserById(){return "test_rest";}

      @RequestMapping(value = "/user", method = RequestMethod.POST)
      public String setUser(){return "test_rest";}

      @RequestMapping(value = "/user", method = RequestMethod.PUT)
      public String modifyUser(){return "test_rest";}
  }
  ```
  ```html
  <a th:href="@{/user}">getUsers</a><br/>
  <a th:href="@{/user/1}">getUserById</a><br/>
  <form th:action="@{/user}" method="post">
      <input type="submit" value="submit"><br/>
  </form>
  <form th:action="@{/user}" method="post">
      <input type="hidden" name="_method" value="put">
      <input type="submit" value="submit"><br/>
  </form>
  ```
  ```xml
  <!-- 使用PUT和DELETE请求时，必须先在web.xml配置过滤器，然后使用<input type="hidden" name="_method" value="put">携带请求方式 -->
  <filter>
      <filter-name>HiddenHttpMethodFilter</filter-name>
      <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  <filter-mapping>
      <filter-name>HiddenHttpMethodFilter</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

# 报文信息转换器: HttpMessageConverter
- 将请求报文转为java对象，或将java对象转为响应报文
- 获取请求体: @RequestBody
  - 使用该注解标记控制器方法的形参，当前请求的请求体就会自动为形参赋值
  ```java
    @RequestMapping("/testRequestBody")
    public String getRequestBody(@RequestBody String requestBody){
        System.out.println(requestBody);
        return "test_converter";
    }
  ```
- 封装请求报文: RequestEntity
  - 将控制器方法形参设置为该类型，会将请求报文自动赋值给该形参，而后通过getHeaders()和getBody()分别获取请求头和请求体信息
  ```java
    @RequestMapping("/testRequestEntity")
    public String getRequestBody(RequestEntity<String> requestBody){
        System.out.println(requestBody.getHeaders());
        System.out.println(requestBody.getBody());
        return "test_converter";
    }
  ```
- 生成响应体: @ResponseBody
  - 使用该注解标记的控制器方法的返回值不再是视图名称，而是响应到浏览器的内容
  ```java
    @RequestMapping("/testResponseBody")
    @ResponseBody
    public String getResponseBody(){
        return "response success";
    }
  ```
  - 通过将java对象转换为json数组可以实现向服务器返回"对象"，其具体步骤为
    - 添加jackson依赖
    ```xml
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.12.1</version>
        </dependency>
    ```
    - 开启mvc注解驱动。此时HandlerAdaptor中会自动装配消息转换器将java对象转换为json字符串
    ```xml
        <mvc: annotation-driven/>
    ```
    - 向控制器方法添加@ResponseBody注解，此控制器方法的返回值为特定的的Java类，并会自动转换为json字符串
    - _注意: 被返回的java类的属性必须是外部可访问的，或者提供其公开的getter，否则依然会报500错误_
- ResponseEntity
  - 用于控制器方法返回值的类型，该控制器方法的返回值就是响应到浏览器的响应报文
  - 实现文件下载功能
    - 下载
    ```java
    @RequestMapping("/downloadFile")
    public ResponseEntity<byte[]> downloadFile(HttpSession session){
        // 获取servletContext对象
        ServletContext context = session.getServletContext();
        // 获取服务器中文件的真实路径
        String realPath = context.getRealPath("/static/files/MathType.zip");

        try (InputStream is = Files.newInputStream(Paths.get(realPath))) {
            // 创建输入字节数组，其中is.available()为输入流的长度
            byte[] bytes = new byte[is.available()];
            is.read(bytes);
            // 设置响应头信息
            MultiValueMap<String, String> headers = new HttpHeaders();
            // 设置下载方式及下载文件的默认名称。“attachment”: 以附件形式下载
            headers.add("Content-Disposition", "attachment;filename=file.zip");
            // 设置响应状态码
            HttpStatus status = HttpStatus.OK;
            // 创建ResponseEntity对象
            return new ResponseEntity<>(bytes, headers, status);
        } catch (IOException e){
            e.printStackTrace();
            return null;
        }
    }
    ```
    - 上传(请求方式必为post)
      - 添加依赖以启用上传功能
      ```xml
      <dependency>
          <groupId>commons-fileupload</groupId>
          <artifactId>commons-fileupload</artifactId>
          <version>1.3.3</version>
      </dependency>
      ```
      - 配置文件上传解析器(解决File类无法转换为MultipartFile的问题)
      ```xml
      <!-- 由于SpringMVC依据id获取bean，因此bean的id必须为multipartResolver,否则会出现控制器方法参数为null的问题(500错误) -->
      <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
      ```
      ```java
      @RequestMapping("/uploadFile")
      public String uploadFile(MultipartFile file, HttpSession session) throws IOException {
          // MultipartFile形参名应当与html中input的name属性一致，否则需要使用@RequestParam()来修饰形参，注解的value为html中input的name
          String fileName=  file.getOriginalFilename();
          ServletContext context = session.getServletContext();
          String filePath = context.getRealPath("files");

          File serverFile = new File(filePath);
          if (!serverFile.exists())
              serverFile.mkdirs();

          String fullPath = filePath + File.separator + fileName;
          file.transferTo(new File(fullPath));
          return "success";
      }
      ```
- 处理ajax
- @RestController
  - 复合注解，用于标记控制器类，此时被标记的类的每个方法都自动被添加了@ResponseBody

# 拦截器
- 拦截器
  - 拦截器用于拦截控制器的执行，分别在控制器前(preHandle)、控制器后(postHandle)、视图渲染后(afterCompletion)执行
  - 需要实现HandlerInterceptor或继承HandlerInterceptorAdaptor
  - 必须在配置文件中进行配置
  - 拦截器返回false表示拦截，true表示放行
  - 可以视作AOP的实际应用，用于扩展控制器方法
- 创建拦截器
  ```java
  public class MyInterceptor implements HandlerInterceptor {
  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
      return HandlerInterceptor.super.preHandle(request, response, handler);
  }

  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
      HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
  }

  @Override
  public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
      HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
  }
}
  ```
- 配置拦截器
  ```xml
  <!-- 在spring.xml中添加。每个bean对应一个拦截器。不被mvc:interceptor包围的拦截器默认对所有请求进行拦截 -->
  <mvc:interceptors>
      <mvc:interceptor>
          <mvc:mapping path="/**"/>   <!-- 对所有请求进行拦截 -->
          <mvc:exclude-mapping path="/"/>   <!-- 对根路径请求放行 -->
          <bean class="indi.beta.demo03.interceptor.MyInterceptor"/>    <!-- 指定拦截器 -->
      </mvc:interceptor>
  </mvc:interceptors>
  ```
- 多个拦截器的顺序
  - 如果所有preHandler都返回true
    - **preHandler按照配置顺序执行**
    - **postHandler和afterCompletion按照配置的反序执行**
  - 如果某个preHandler返回false
    - 该拦截器(因为执行了才能返回false)及之前的preHandler都会顺序执行
    - postHandler都不执行
    - 该拦截器及之前的afterCompletion都会顺序执行(比preHandler少一个)

# 异常处理器
- 基于配置的异常处理
  ```xml
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <prop key="java.lang.ArithmeticException">error</prop>  <!-- 设置出现该类错误时跳转到的视图名 -->
            </props>
        </property>
        <property name="exceptionAttribute" value="ex"/>    <!-- 将错误信息转发到浏览器。通过<p th:text="${ex}"/>获取 -->
    </bean>
  ```
- 基于注解的异常处理
  ```java
  @ControllerAdvice
  public class ExceptionController {
      @ExceptionHandler(value = {ArithmeticException.class, NullPointerException.class})
      public String testException(Exception exception, Model model){
          model.addAttribute("ex", exception);  // 将错误信息转发到浏览器，通过<p th:text="${ex}"/>获取
          return "error";   // 跳转的视图名称
      }
  }
  ```
  
# 全注解开发SpringMVC
- 新建WebInit类
  ```java
  package indi.beta.config;

  import org.springframework.web.filter.CharacterEncodingFilter;
  import org.springframework.web.filter.HiddenHttpMethodFilter;
  import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

  import javax.servlet.Filter;

  // 用于代替web.xml
  public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
      @Override
      protected Class<?>[] getRootConfigClasses() {
          // 获取根配置，即Spring配置类
          return new Class[]{SpringConfig.class};
      }

      @Override
      protected Class<?>[] getServletConfigClasses() {
          // 获取SpringMVC配置类
          return new Class[]{WebConfig.class};
      }

      @Override
      protected String[] getServletMappings() {
          // 获取DispatcherServlet的url-pattern
          return new String[]{"/"};
      }

      @Override
      protected Filter[] getServletFilters() {
          // 注册过滤器
          CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
          characterEncodingFilter.setEncoding("UTF-8");
          characterEncodingFilter.setForceResponseEncoding(true);

          HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
          return new Filter[]{characterEncodingFilter, hiddenHttpMethodFilter};
      }
  }

  ```
- 新建SpringMVC配置类
  ```java
  package indi.beta.config;

  import indi.beta.interceptor.PageInterceptor;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.web.context.ContextLoader;
  import org.springframework.web.context.WebApplicationContext;
  import org.springframework.web.multipart.MultipartResolver;
  import org.springframework.web.multipart.commons.CommonsMultipartResolver;
  import org.springframework.web.servlet.HandlerExceptionResolver;
  import org.springframework.web.servlet.ViewResolver;
  import org.springframework.web.servlet.config.annotation.*;
  import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;
  import org.thymeleaf.spring5.SpringTemplateEngine;
  import org.thymeleaf.spring5.view.ThymeleafViewResolver;
  import org.thymeleaf.templatemode.TemplateMode;
  import org.thymeleaf.templateresolver.ITemplateResolver;
  import org.thymeleaf.templateresolver.ServletContextTemplateResolver;

  import java.util.List;
  import java.util.Properties;

  @Configuration
  // 开启组件扫描
  @ComponentScan("indi.beta")
  // 开启mvc注解驱动
  @EnableWebMvc
  public class WebConfig implements WebMvcConfigurer {
      @Bean
      public ITemplateResolver templateResolver(){
          // 配置模板解析器
          WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();
          ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(
                  context.getServletContext());

          templateResolver.setCharacterEncoding("UTF-8");
          templateResolver.setTemplateMode(TemplateMode.HTML);
          templateResolver.setPrefix("/WEB-INF/pages/");
          templateResolver.setSuffix(".html");
          return templateResolver;
      }

      @Bean
      public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver){
          // 配置模板引擎
          SpringTemplateEngine templateEngine = new SpringTemplateEngine();
          templateEngine.setTemplateResolver(templateResolver);
          return templateEngine;
      }

      @Bean
      public ViewResolver viewResolver(SpringTemplateEngine templateEngine){
          // 配置视图解析器
          ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
          viewResolver.setCharacterEncoding("UTF-8");
          viewResolver.setTemplateEngine(templateEngine);
          return viewResolver;
      }

      @Bean
      public MultipartResolver fileUploadResolver(){
          // 配置文件上传解析器
          return new CommonsMultipartResolver();
      }

      @Override
      public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
          // 配置异常解析器
          SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
          Properties properties = new Properties();
          // 添加一条错误映射规则
          properties.setProperty("java.lang.ArithmeticException", "error");
          resolver.setExceptionMappings(properties);
          // 设置请求域获取错误信息的字段名。html通过该字段即可获取具体的错误信息
          resolver.setExceptionAttribute("ex");
          resolvers.add(resolver);
      }

      @Override
      public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
          // 设置默认servlet可用
          configurer.enable();
      }

      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          // 添加拦截器并配置拦截规则
          registry.addInterceptor(new PageInterceptor()).addPathPatterns("/**").excludePathPatterns("/");
      }

      @Override
      public void addViewControllers(ViewControllerRegistry registry) {
          // 添加视图控制器，并设置视图名称
          registry.addViewController("/hello").setViewName("hello");
      }
  }

  ```
- 建立必要的页面控制器
  ```java
  package indi.beta.controller;

  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;

  @Controller
  public class PageController {
      @RequestMapping("/")
      public String toPortal(){
          return "portal";
      }
  }
  ```
- 注意
  - 有时部署后会出现500错误，显示html资源找不到。此情况建议使用maven打包为war后，将tomcat里的部署从"工件"切换为打包的war
  - 这一操作需要提前在pom中添加以下内容
    ```xml
        <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.2.0</version>

                <configuration>
                    <!-- 跳过web.xml进行打包，因为全注解开发将不会存在web.xml -->
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```
# SpringMVC的执行流程
- SpringMVC的常用组件
  - DispatcherServlet: 前端控制器。用于统一处理请求和响应，是整个流程的控制中心，用于调用其他组件处理用户请求
  - HandlerMapping: 处理器映射器。依据请求的url、method等信息查找控制器(Handler或Controller)
  - Handler: 处理器。又称Controller，用于在DispatcherServlet的控制下对具体的用户请求进行处理。需要工程师进行开发
  - HandlerAdapter: 处理器适配器。通过适配器来调用控制器方法
  - ViewResolver: 视图解析器。进行视图解析，得到相应的视图。常用的入Thymeleaf、InternalResourceView、RedirectView
  - View: 视图。将模型数据通过页面展示给用户
- DispatcherServlet初始化流程
  - FrameworkServlet创建WebApplicationContext
  - 调用onRefresh(wac)，刷新容器
  - 调用initStrategies(context)，初始化DispatcherServlet各个组件
- SpringMVC运行流程
  - 用户向服务器发送请求，请求被DispatcherServlet捕获
  - DispatcherServlet对URL进行解析，得到请求资源标识符(URI)，并判断URI对应的映射
    - 不存在时
      - 判断是否配置过默认控制器(mvc:default-servlet-handler)
        - 不存在时，报404
        - 存在时，访问目标资源(一般是静态的)，找不到是依然报404(提示映射到DefaultServletHttpRequestHandler)
    - 存在时
      - 依据URI调用控制器映射器获取控制器及相关的所有配置(控制器、过滤器、拦截器)，以控制器执行链的形式返回
      - DispatcherServlet获取控制器，分配合适的控制器适配器，获取成功则开始执行拦截器的preHandler()
      - 提取Request的模型数据并填充到控制器方法的形参，开始执行控制器方法。一句配置，Spring将会自动完成一些工作
        - HttpMessageConverter: 将请求消息转换为对象，将对象转换为响应信息
        - 数据转换: 将请求消息进行转换，例如String转换为Integer
        - 数据格式化: 例如将字符串转换为日期或数字
        - 数据校验: 验证数据的有效性
      - 控制器执行完成后，向DispatcherServlet返回一个ModelAndView对象
      - 执行拦截器方法的postHandler()
      - 依据ModelAndView(如果不存在异常的话，存在异常时执行异常处理器并跳转到相应页面)选择合适的视图解析器进行解析，根据Model和View渲染视图
      - 渲染完成，执行拦截器的afterCompletion()
      - 将渲染结果返回给客户端
