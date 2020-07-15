1.实现过滤器`Filter`，如果存在注入字符，则终止请求
```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
throws IOException, ServletException {
    ...
    if(isSecure(httpRequest)) {
	    writeContent("非法参数，请输入正确参数！");
    }
    ...
}
```
2.`isSecure(request)`方法，对输入参数进行检查
```java
private boolean isSecure(HttpServletRequest request) {		
	Map map = request.getParameterMap();
	Set keyset = map.keySet();
	Iterator it = keyset.iterator();
	while (it.hasNext()) {
		String key = (String) it.next();
		String[] values = (String[]) map.get(key);
		String value = "";
		for (int i = 0; i < values.length; i++) {
			value += value + values[i];
		}
		if (SecureUtil.isSQL(value)||SecureUtil.isXss(value)) {
			return true;
		}
	}
	return false;
}    
```
3.使用正则验证参数
```java
//正则，
private final static Map<Integer, String> xssReg = new HashMap<Integer, String>();
private final static Map<Integer, String> sqlReg = new HashMap<Integer, String>();
static {
	xssReg.put(1, "<s.*?t.*?>.*</s.*?t>");
	xssReg.put(2, "javascript:.*");
	xssReg.put(3, "<(img|iframe).*>");
	xssReg.put(4,"onerror|onload|onfocus|onmousemove|onmouseout|onmouseover|onclick|href|src|alert|comfirm|prompt
	        |console.log");
	xssReg.put(5, "\"|\'|<|>|/");
	sqlReg.put(1, "[' -]");
	sqlReg.put(2, "select|insert|update|delete|drop|truncate|declare|xp_cmdshell|net user|/add|exec|execute|xp_|sp_|0x|0 x");
}

//检查方法
public static Boolean isXss(String param) {
	param = StringEscapeUtils.unescapeHtml3(param);
	Set keySet=xssReg.keySet();
	Iterator it=keySet.iterator();
	while (it.hasNext()){
		int key=(int) it.next();
		Pattern p = Pattern.compile(xssReg.get(key), Pattern.CASE_INSENSITIVE);
		Matcher m = p.matcher(param);
		if (m.find() || m.matches()) {
			return true;
		}
	}
	return false;
}
public static boolean isSQL(String param) {
	param = StringEscapeUtils.unescapeHtml3(param);
	Set keySet=sqlReg.keySet();
	Iterator it=keySet.iterator();
	while (it.hasNext()){
		int key=(int) it.next();
		Pattern p = Pattern.compile(sqlReg.get(key), Pattern.CASE_INSENSITIVE);
		Matcher m = p.matcher(param);
		if (m.find() || m.matches()) {
			return true;
		}
	}
	return false;
}
```
