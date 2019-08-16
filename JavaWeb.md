### 1. 关于RequestDispatcher.forward(request, response)方法和response.sendRedirect(location)方法的比较：
- RequestDispatcher.forward方法通常叫做请求转发，而response.sendRedirect方法通常叫做重定向
- 请求转发，整个过程在同一个请求中，即request对象是同一个
- 重定向，实际上会向服务器端发送两个请求
- RequestDispatcher是通过调用HttpServletRequeset对象的getRequestDispatcher()方法获得的，是属于请求对象的方法；sendRedirect()是HttpServletResponse对象的方法，即响应对象的方法，既然调用了响应对象的方法，那就表明整个请求过程已经结束了，服务器开始向客户端返回执行的结果。

### 2.JavaBean对象的存活范围
- page
- request
- session
- application

#### 3. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAwNjI5NDk5Ml19
-->