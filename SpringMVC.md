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
