## spring-security组件的使用

- 首先引入spring-security.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:security="http://www.springframework.org/schema/security"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security.xsd">

    <security:http pattern="/login.jsp" security="none"/>
    <security:http pattern="/css/**" security="none"/>
    <security:http pattern="/bower_components/**" security="none"/>
    <security:http pattern="/js/**" security="none"/>
    <security:http pattern="/image/**" security="none"/>
    <security:http pattern="/dist/**" security="none"/>
    <security:http pattern="/plugins/**" security="none"/>

    <security:http auto-config="true" use-expressions="false">
        <!--配置拦截规则-->
        <security:intercept-url pattern="/**" access="ROLE_USER,ROLE_ADMIN"/>

        <security:form-login
                login-page="/login.jsp"
                login-processing-url="/login"
                authentication-success-forward-url="/pages/index.jsp"
                authentication-failure-url="/login.jsp"
        />
        <!--是否开启跨域-->
        <security:csrf disabled="true"/>
        <!--退出-->
        <security:logout invalidate-session="true" logout-url="/logout" logout-success-url="/login.jsp"/>

    </security:http>

    <!--访问数据库user表-->
    <security:authentication-manager>
        <security:authentication-provider user-service-ref="userService">
            <!--加密方式-->
            <security:password-encoder ref="passwordEncoder"/>
        </security:authentication-provider>
    </security:authentication-manager>
    
    <!-- springsecurity 自带的加密算法  通过明文密码加盐 在进行加密-->
    <bean class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" id="passwordEncoder"/>
</beans>

```

- 在web.xml文件中配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

    <!--扫描配置文件，并注册到bean 容器-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:applicationConfig.xml,classpath*:spring-security.xml</param-value>
  </context-param>


<!-- 视图解析器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath*:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
<!--请求监听器-->
  <listener>
    <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
  </listener>

  <!--中文乱码过滤器-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

    <!-- 主要作用就是一个代理模式的应用,可以把servlet 容器中的filter同spring容器中的bean关联起来-->
  <filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>


</web-app>

```