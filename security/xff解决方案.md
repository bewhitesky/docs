# xff防御方法
## 未设置代理：
```java
public String getRemortIP(HttpServletRequest request) {
      return request.getRemoteAddr();
}
```
## 设置多级代理
* nginx配置
```text
 location / {
                proxy_set_header        X-Real-IP       $remote_addr;
                proxy_set_header        Host            $host;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                ...
            }
```
* 客户端获取ip
```java
public String getRemortIP(HttpServletRequest request) {
      return request.getHeader("X-Real-IP");
}
```