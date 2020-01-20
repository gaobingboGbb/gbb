### 1.更改springMVC中htttp请求中的请求参数

​	主要通过HttpServletRequestWrapper实现

```java
public class RequestParametersFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String envVersion= httpServletRequest.getHeader("envVersion");
        String userToken= httpServletRequest.getHeader("Authorization");
        String longitude= httpServletRequest.getHeader("longitude");
        String latitude= httpServletRequest.getHeader("latitude");
        // 3.放行，把我们的requestWrapper1放到方法当中
        if (StrUtil.isNotBlank(userToken)||StrUtil.isNotBlank(envVersion)||StrUtil.isNotBlank(longitude)||StrUtil.isNotBlank(latitude)) {
            System.out.println("head方法进行过滤开始=============================");
            HeadToParamsRequestWrapper headToParamsRequestWrapper = new HeadToParamsRequestWrapper(
                    (HttpServletRequest) servletRequest);
            // 1.获取需要处理的参数
            // 2.把处理后的参数放回去
            headToParamsRequestWrapper.setParameter("envVersion",envVersion);
            headToParamsRequestWrapper.setParameter("userToken",userToken);
            headToParamsRequestWrapper.setParameter("longitude",longitude);
            headToParamsRequestWrapper.setParameter("latitude",latitude);
            filterChain.doFilter(headToParamsRequestWrapper,servletResponse);
        }else{
            filterChain.doFilter(servletRequest,servletResponse);
        }
        //请求方式有两种一种是get一种是post
//        if (httpServletRequest.getMethod().equalsIgnoreCase("post")) {
//            //处理json报文请求
//            HeadToParamsPostRequestWrapper headToParamsPostRequestWrapper = new HeadToParamsPostRequestWrapper(
//                    (HttpServletRequest) servletRequest);
//
//            // 读取请求内容
//            BufferedReader br;
//            br = headToParamsPostRequestWrapper.getReader();
//            String line = null;
//            StringBuilder sb = new StringBuilder();
//            while ((line = br.readLine()) != null) {
//                sb.append(line);
//            }
//            // 将json字符串转换为json对象
//            JSONObject jsonObject = JSON.parseObject(sb.toString());
//            Map<String, Object> map = new HashMap<String, Object>();
//            // 把json对象转换为Map集合
//            map = JSON.toJavaObject(jsonObject, Map.class);
//            if (ObjectUtil.isNull(map)) {
//                map = new HashMap<>();
//            }
//            map.put("envVersion", envVersion);
//            map.put("userToken", userToken);
//            map.put("longitude", longitude);
//            map.put("latitude", latitude);
//            // 把参数转换之后放到我们的body里面
//            String json = JSON.toJSONString(map);
//            headToParamsPostRequestWrapper.setBody(json.getBytes("UTF-8"));
//            // 放行
//            filterChain.doFilter(headToParamsPostRequestWrapper,servletResponse);
//        } else if (httpServletRequest.getMethod().equalsIgnoreCase("get")){
//            HeadToParamsRequestWrapper headToParamsRequestWrapper = new HeadToParamsRequestWrapper(
//                    (HttpServletRequest) servletRequest);
//            // 1.获取需要处理的参数
//            // 2.把处理后的参数放回去
//            headToParamsRequestWrapper.setParameter("envVersion",envVersion);
//            headToParamsRequestWrapper.setParameter("userToken",userToken);
//            headToParamsRequestWrapper.setParameter("longitude",longitude);
//            headToParamsRequestWrapper.setParameter("latitude",latitude);
//            // 3.放行，把我们的requestWrapper1放到方法当中
//            filterChain.doFilter(headToParamsRequestWrapper,servletResponse);
//        }
    }

    @Override
    public void destroy() {

    }
}
```

get请求

```java
public class HeadToParamsRequestWrapper extends HttpServletRequestWrapper {
    // 用于存储请求参数
    private Map<String , String[]> params = new HashMap<String, String[]>();
    /**
     * Constructs a request object wrapping the given request.
     *
     * @param request the {@link HttpServletRequest} to be wrapped.
     * @throws IllegalArgumentException if the request is null
     */
    public HeadToParamsRequestWrapper(HttpServletRequest request) {
        super(request);
        // 把请求参数添加到我们自己的map当中
        this.params.putAll(request.getParameterMap());
    }

    /**
     * 添加参数到map中
     * @param extraParams
     */
    public void setParameterMap(Map<String, Object> extraParams) {
        for (Map.Entry<String, Object> entry : extraParams.entrySet()) {
            setParameter(entry.getKey(), entry.getValue());
        }
    }

    /**
     * 添加参数到map中
     * @param name
     * @param value
     */
    public void setParameter(String name, Object value) {
        if (value != null) {
            System.out.println(value);
            if (value instanceof String[]) {
                params.put(name, (String[]) value);
            } else if (value instanceof String) {
                params.put(name, new String[]{(String) value});
            } else {
                params.put(name, new String[]{String.valueOf(value)});
            }
        }
    }


    /**
     * 重写getParameter，代表参数从当前类中的map获取
     * @param name
     * @return
     */
    @Override
    public String getParameter(String name) {
        String[]values = params.get(name);
        if(values == null || values.length == 0) {
            return null;
        }
        return values[0];
    }

    /**
     * 重写getParameterValues方法，从当前类的 map中取值
     * @param name
     * @return
     */
    @Override
    public String[] getParameterValues(String name) {
        return params.get(name);
    }
}
```

post请求：

​	post请求有一个流的坑，所以需要对应的Buffere

```java
public class HeadToParamsPostRequestWrapper extends HttpServletRequestWrapper {
    private byte[] body; //用于保存读取body中数据

    public HeadToParamsPostRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        //读取请求的数据保存到本类当中
        body = StreamUtil.readBytes(request.getReader(), "UTF-8");
    }

    //覆盖（重写）父类的方法
    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }
    //覆盖（重写）父类的方法
    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream bais = new ByteArrayInputStream(body);
        return new ServletInputStream() {
            @Override
            public boolean isFinished() {
                return false;
            }

            @Override
            public boolean isReady() {
                return false;
            }

            @Override
            public void setReadListener(ReadListener readListener) {

            }

            @Override
            public int read() throws IOException {
                return bais.read();
            }
        };
    }
    /**
     * 获取body中的数据
     *
     * @return
     */
    public byte[] getBody() {
        return body;
    }

    /**
     * 把处理后的参数放到body里面
     *
     * @param body
     */
    public void setBody(byte[] body) {
        this.body = body;
    }
}

```

