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
  - 
