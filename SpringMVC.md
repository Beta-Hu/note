# RequestMapping
- value
  - String[], 必填
  - 依据(任一)地址进行匹配
- method
  - RequestMethod.GET | RequestMethod.POST, 可(多)选
  - 在地址匹配后依据方法匹配
  - GET与POST的区别
    - GET: 在地址后追加键值对。不安全。快速。容量小。不适用文件传输
    - POST: 使用单独的请求体。安全。低效。容量大。可用于文件传输。
