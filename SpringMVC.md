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
