异常捕获
* 封装自定义业务异常BusinessException
```java
    public class BusinessException extends RuntimeException{
        public BusinessException(Object obj ,Throwable cause){
        	super(obj.toString(),cause);
        }
    }
```

```java
        try {
            ......
        } catch (Exception e) {
            throw new BusinessException(......,e);
        }
```