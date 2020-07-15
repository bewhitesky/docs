* 前端传输时间戳和随机数和加密后的签名
==============================
* 设置失效时间，验证

``` java
        /**
         * 时间戳请求最大限制
         */
        private static final long MAX_REQUEST = 60 * 1000L;
        /**
         * 配置文件
         */
        @Resource
        private AppConfig appConfig;
        /**
         * 验签切点
         */
        @Pointcut("execution(@com.token.demo.aop.SignatureValidation * *(..))")
        private void verifyUserKey() {
        }
    
        /**
         * 开始验签
         */
        @Before("verifyUserKey()")
        public void doBasicProfiling() {
            HttpServletRequest request = ((ServletRequestAttributes) Objects
                                    .requireNonNull(RequestContextHolder.getRequestAttributes())).getRequest();
            String nonce = request.getHeader("nonce");
            String timestamp = request.getHeader("timestamp");
            String noncesign = request.getHeader("noncesign");
            try {
                Boolean check = checkToken(nonce, timestamp,noncesign);
                if (!check) {
                    throw new ErrorCodeException(ErrorCodeEnum.SIGN_ERROR);
                }
            } catch (Throwable throwable) {
                throw new ErrorCodeException(ErrorCodeEnum.SIGN_ERROR);
            }
        }
```

* 验证前端传过来的时间戳和随机数和签名
```java
    /**
     * 校验token
     *
     * @param nonce     随机数
     * @param timestamp 时间戳
     * @param noncesign 加密后的签名
     * @return 校验结果
     */
        List<String> list = new ArrayList<String>();//这边采用集合，实际情况请使用redis缓存机制
    
        private Boolean checkToken(String nonce, String timestamp,String noncesign) {
            if (StringUtils.isAnyBlank(nonce, timestamp,nonce)) {
                return false;
            }
    
            long now = System.currentTimeMillis();
            long time = Long.parseLong(timestamp);
            if (now - time > MAX_REQUEST) {
                log.error("时间戳已过期[{}][{}][{}]", now, time, (now - time));
                return false;
            }else {
                if (list.contains(nonce)){
                    log.error("存在防重放[{}][{}][{}]", now, time, (now - time));
                    return false;
                }else {
                    list.add(nonce);
                }
            }
    
        //此处采用MD5加密验证签名，实际情况请采用国密算法进行加解密
            String crypt = MD5Utils.getMD5(nonce);
            if (!StringUtils.equals(crypt, noncesign)) {
                log.error("请求token[{}],timestamp[{}]-vs-服务器token[{}]", nonce, timestamp, crypt);
                return false;
            }
            log.info("签名通过");
            return true;
        }
```