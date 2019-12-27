HttpServlet->ServletBean->FrameworkServlet->DispatcherServlet

public abstract class HttpServlet{
    public HttpServlet() {
    }

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_get_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    protected long getLastModified(HttpServletRequest req) {
        return -1L;
    }

    protected void doHead(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        NoBodyResponse response = new NoBodyResponse(resp);
        this.doGet(req, response);
        response.setContentLength();
    }

    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_post_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    protected void doPut(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_put_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    protected void doDelete(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_delete_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }
    //获取父类和自己所有的方法
    private static Method[] getAllDeclaredMethods(Class c) {
        if (c.equals(HttpServlet.class)) {
            return null;
        } else {
            Method[] parentMethods = getAllDeclaredMethods(c.getSuperclass());
            Method[] thisMethods = c.getDeclaredMethods();
            if (parentMethods != null && parentMethods.length > 0) {
                Method[] allMethods = new Method[parentMethods.length + thisMethods.length];
                System.arraycopy(parentMethods, 0, allMethods, 0, parentMethods.length);
                System.arraycopy(thisMethods, 0, allMethods, parentMethods.length, thisMethods.length);
                thisMethods = allMethods;
            }

            return thisMethods;
        }
    }
    //options 获取服务器可以执行的方法，allow字段
    protected void doOptions(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Method[] methods = getAllDeclaredMethods(this.getClass());
        boolean ALLOW_GET = false;
        boolean ALLOW_HEAD = false;
        boolean ALLOW_POST = false;
        boolean ALLOW_PUT = false;
        boolean ALLOW_DELETE = false;
        boolean ALLOW_TRACE = true;
        boolean ALLOW_OPTIONS = true;

        for(int i = 0; i < methods.length; ++i) {
            Method m = methods[i];
            if (m.getName().equals("doGet")) {
                ALLOW_GET = true;
                ALLOW_HEAD = true;
            }

            if (m.getName().equals("doPost")) {
                ALLOW_POST = true;
            }

            if (m.getName().equals("doPut")) {
                ALLOW_PUT = true;
            }

            if (m.getName().equals("doDelete")) {
                ALLOW_DELETE = true;
            }
        }

        String allow = null;
        if (ALLOW_GET && allow == null) {
            allow = "GET";
        }

        if (ALLOW_HEAD) {
            if (allow == null) {
                allow = "HEAD";
            } else {
                allow = allow + ", HEAD";
            }
        }

        if (ALLOW_POST) {
            if (allow == null) {
                allow = "POST";
            } else {
                allow = allow + ", POST";
            }
        }

        if (ALLOW_PUT) {
            if (allow == null) {
                allow = "PUT";
            } else {
                allow = allow + ", PUT";
            }
        }

        if (ALLOW_DELETE) {
            if (allow == null) {
                allow = "DELETE";
            } else {
                allow = allow + ", DELETE";
            }
        }

        if (ALLOW_TRACE) {
            if (allow == null) {
                allow = "TRACE";
            } else {
                allow = allow + ", TRACE";
            }
        }

        if (ALLOW_OPTIONS) {
            if (allow == null) {
                allow = "OPTIONS";
            } else {
                allow = allow + ", OPTIONS";
            }
        }

        resp.setHeader("Allow", allow);
    }
    //返回服务器收到的请求url，请求头
    protected void doTrace(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String CRLF = "\r\n";
        String responseString = "TRACE " + req.getRequestURI() + " " + req.getProtocol();

        String headerName;
        for(Enumeration reqHeaderEnum = req.getHeaderNames(); reqHeaderEnum.hasMoreElements(); responseString = responseString + CRLF + headerName + ": " + req.getHeader(headerName)) {
            headerName = (String)reqHeaderEnum.nextElement();
        }

        responseString = responseString + CRLF;
        int responseLength = responseString.length();
        resp.setContentType("message/http");
        resp.setContentLength(responseLength);
        ServletOutputStream out = resp.getOutputStream();
        out.print(responseString);
        out.close();
    }

    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                //自从上次请求发生了修改
                this.doGet(req, resp);
            } else {
                //没有修改
                long ifModifiedSince;
                try {
                    //查看请求头字段
                    ifModifiedSince = req.getDateHeader("If-Modified-Since");
                } catch (IllegalArgumentException var9) {
                    ifModifiedSince = -1L;
                }
                //比较时间
                if (ifModifiedSince < lastModified / 1000L * 1000L) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    //使用缓存
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            //没有找到处理的方法，该请求方法不存在
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }
    //通知浏览器发生了修改
    private void maybeSetLastModified(HttpServletResponse resp, long lastModified) {
        if (!resp.containsHeader("Last-Modified")) {
            if (lastModified >= 0L) {
                resp.setDateHeader("Last-Modified", lastModified);
            }

        }
    }

    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request;
        HttpServletResponse response;
        try {
            request = (HttpServletRequest)req;
            response = (HttpServletResponse)res;
        } catch (ClassCastException var6) {
            throw new ServletException("non-HTTP request or response");
        }

        this.service(request, response);
    }
  
}

重点：304 通知浏览器使用缓存
      501 服务器找不到处理的方法，也就是请求方法不在（post，get，put...中）
      服务器字段：lastModified 浏览器字段：If-Modified-Since
      浏览器请求时，服务器先查看lastmodified，如果不是-1，表示没有修改过，如果是-1，表是修改过，执行doget（）
      当服务器没有修改过的时候，拿到自己上次修改的时间，lastmodified，和浏览器的 if-modified-since（表示客户端可以接受的最早修改时间），如果lastmodifed《ifmodifed-since，
      浏览器doget，如果大于，返回304，同时通知客户端lastmodifed时间      
      200的时候也会通知客户端lastmodifed

public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
    protected final Log logger = LogFactory.getLog(this.getClass());
    private final Set<String> requiredProperties = new HashSet();
    private ConfigurableEnvironment environment;

    public HttpServletBean() {
    }

    protected final void addRequiredProperty(String property) {
        this.requiredProperties.add(property);
    }

    public final void init() throws ServletException {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Initializing servlet '" + this.getServletName() + "'");
        }

        try {
            PropertyValues pvs = new HttpServletBean.ServletConfigPropertyValues(this.getServletConfig(), this.requiredProperties);
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(this.getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.getEnvironment()));
            this.initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
        } catch (BeansException var4) {
            this.logger.error("Failed to set bean properties on servlet '" + this.getServletName() + "'", var4);
            throw var4;
        }

        this.initServletBean();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Servlet '" + this.getServletName() + "' configured successfully");
        }

    }

    protected void initBeanWrapper(BeanWrapper bw) throws BeansException {
    }

    public final String getServletName() {
        return this.getServletConfig() != null ? this.getServletConfig().getServletName() : null;
    }

    public final ServletContext getServletContext() {
        return this.getServletConfig() != null ? this.getServletConfig().getServletContext() : null;
    }

    protected void initServletBean() throws ServletException {
    }

    public void setEnvironment(Environment environment) {
        Assert.isInstanceOf(ConfigurableEnvironment.class, environment);
        this.environment = (ConfigurableEnvironment)environment;
    }

    public ConfigurableEnvironment getEnvironment() {
        if (this.environment == null) {
            this.environment = this.createEnvironment();
        }

        return this.environment;
    }

    protected ConfigurableEnvironment createEnvironment() {
        return new StandardServletEnvironment();
    }

    private static class ServletConfigPropertyValues extends MutablePropertyValues {
        public ServletConfigPropertyValues(ServletConfig config, Set<String> requiredProperties) throws ServletException {
            Set<String> missingProps = requiredProperties != null && !requiredProperties.isEmpty() ? new HashSet(requiredProperties) : null;
            Enumeration en = config.getInitParameterNames();

            while(en.hasMoreElements()) {
                String property = (String)en.nextElement();
                Object value = config.getInitParameter(property);
                this.addPropertyValue(new PropertyValue(property, value));
                if (missingProps != null) {
                    missingProps.remove(property);
                }
            }

            if (missingProps != null && missingProps.size() > 0) {
                throw new ServletException("Initialization from ServletConfig for servlet '" + config.getServletName() + "' failed; the following required properties were missing: " + StringUtils.collectionToDelimitedString(missingProps, ", "));
            }
        }
    }
}
      
    
    
public abstract class FrameworkServlet extends HttpServletBean {
    public static final String DEFAULT_NAMESPACE_SUFFIX = "-servlet";
    public static final Class<?> DEFAULT_CONTEXT_CLASS = XmlWebApplicationContext.class;
    public static final String SERVLET_CONTEXT_PREFIX = FrameworkServlet.class.getName() + ".CONTEXT.";
    private static final String INIT_PARAM_DELIMITERS = ",; \t\n";
    private String contextAttribute;
    private Class<?> contextClass;
    private String contextId;
    private String namespace;
    private String contextConfigLocation;
    private final ArrayList<ApplicationContextInitializer<ConfigurableApplicationContext>> contextInitializers;
    private String contextInitializerClasses;
    private boolean publishContext;
    private boolean publishEvents;
    private boolean threadContextInheritable;
    private boolean dispatchOptionsRequest;
    private boolean dispatchTraceRequest;
    private WebApplicationContext webApplicationContext;
    private boolean refreshEventReceived;

    public FrameworkServlet() {
        this.contextClass = DEFAULT_CONTEXT_CLASS;
        this.contextInitializers = new ArrayList();
        this.publishContext = true;
        this.publishEvents = true;
        this.threadContextInheritable = false;
        //默认不处理option，trace
        this.dispatchOptionsRequest = false;
        this.dispatchTraceRequest = false;
        this.refreshEventReceived = false;
    }

    public FrameworkServlet(WebApplicationContext webApplicationContext) {
        this.contextClass = DEFAULT_CONTEXT_CLASS;
        this.contextInitializers = new ArrayList();
        this.publishContext = true;
        this.publishEvents = true;
        this.threadContextInheritable = false;
        this.dispatchOptionsRequest = false;
        this.dispatchTraceRequest = false;
        this.refreshEventReceived = false;
        this.webApplicationContext = webApplicationContext;
    }

    public void setContextAttribute(String contextAttribute) {
        this.contextAttribute = contextAttribute;
    }

    public String getContextAttribute() {
        return this.contextAttribute;
    }

    public void setContextClass(Class<?> contextClass) {
        this.contextClass = contextClass;
    }

    public Class<?> getContextClass() {
        return this.contextClass;
    }

    public void setContextId(String contextId) {
        this.contextId = contextId;
    }

    public String getContextId() {
        return this.contextId;
    }

    public void setNamespace(String namespace) {
        this.namespace = namespace;
    }

    public String getNamespace() {
        return this.namespace != null ? this.namespace : this.getServletName() + "-servlet";
    }

    public void setContextConfigLocation(String contextConfigLocation) {
        this.contextConfigLocation = contextConfigLocation;
    }

    public String getContextConfigLocation() {
        return this.contextConfigLocation;
    }

    public void setContextInitializers(ApplicationContextInitializer<? extends ConfigurableApplicationContext>... contextInitializers) {
        ApplicationContextInitializer[] arr$ = contextInitializers;
        int len$ = contextInitializers.length;

        for(int i$ = 0; i$ < len$; ++i$) {
            ApplicationContextInitializer<? extends ConfigurableApplicationContext> initializer = arr$[i$];
            this.contextInitializers.add(initializer);
        }

    }

    public void setContextInitializerClasses(String contextInitializerClasses) {
        this.contextInitializerClasses = contextInitializerClasses;
    }

    public void setPublishContext(boolean publishContext) {
        this.publishContext = publishContext;
    }

    public void setPublishEvents(boolean publishEvents) {
        this.publishEvents = publishEvents;
    }

    public void setThreadContextInheritable(boolean threadContextInheritable) {
        this.threadContextInheritable = threadContextInheritable;
    }

    public void setDispatchOptionsRequest(boolean dispatchOptionsRequest) {
        this.dispatchOptionsRequest = dispatchOptionsRequest;
    }

    public void setDispatchTraceRequest(boolean dispatchTraceRequest) {
        this.dispatchTraceRequest = dispatchTraceRequest;
    }

    protected final void initServletBean() throws ServletException {
        this.getServletContext().log("Initializing Spring FrameworkServlet '" + this.getServletName() + "'");
        if (this.logger.isInfoEnabled()) {
            this.logger.info("FrameworkServlet '" + this.getServletName() + "': initialization started");
        }

        long startTime = System.currentTimeMillis();

        try {
            //初始化
            this.webApplicationContext = this.initWebApplicationContext();
            this.initFrameworkServlet();
        } catch (ServletException var5) {
            this.logger.error("Context initialization failed", var5);
            throw var5;
        } catch (RuntimeException var6) {
            this.logger.error("Context initialization failed", var6);
            throw var6;
        }

        if (this.logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            this.logger.info("FrameworkServlet '" + this.getServletName() + "': initialization completed in " + elapsedTime + " ms");
        }

    }

    protected WebApplicationContext initWebApplicationContext() {
        WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
        WebApplicationContext wac = null;
        if (this.webApplicationContext != null) {
            wac = this.webApplicationContext;
            if (wac instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)wac;
                if (!cwac.isActive()) {
                    if (cwac.getParent() == null) {
                        cwac.setParent(rootContext);
                    }

                    this.configureAndRefreshWebApplicationContext(cwac);
                }
            }
        }

        if (wac == null) {
            wac = this.findWebApplicationContext();
        }

        if (wac == null) {
            wac = this.createWebApplicationContext(rootContext);
        }

        if (!this.refreshEventReceived) {
            this.onRefresh(wac);
        }

        if (this.publishContext) {
            String attrName = this.getServletContextAttributeName();
            this.getServletContext().setAttribute(attrName, wac);
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Published WebApplicationContext of servlet '" + this.getServletName() + "' as ServletContext attribute with name [" + attrName + "]");
            }
        }

        return wac;
    }

    protected WebApplicationContext findWebApplicationContext() {
        String attrName = this.getContextAttribute();
        if (attrName == null) {
            return null;
        } else {
            WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext(), attrName);
            if (wac == null) {
                throw new IllegalStateException("No WebApplicationContext found: initializer not registered?");
            } else {
                return wac;
            }
        }
    }

    protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
        Class<?> contextClass = this.getContextClass();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Servlet with name '" + this.getServletName() + "' will try to create custom WebApplicationContext context of class '" + contextClass.getName() + "'" + ", using parent context [" + parent + "]");
        }

        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
            throw new ApplicationContextException("Fatal initialization error in servlet with name '" + this.getServletName() + "': custom WebApplicationContext class [" + contextClass.getName() + "] is not of type ConfigurableWebApplicationContext");
        } else {
            ConfigurableWebApplicationContext wac = (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
            wac.setEnvironment(this.getEnvironment());
            wac.setParent(parent);
            wac.setConfigLocation(this.getContextConfigLocation());
            this.configureAndRefreshWebApplicationContext(wac);
            return wac;
        }
    }

    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
        if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
            if (this.contextId != null) {
                wac.setId(this.contextId);
            } else {
                ServletContext sc = this.getServletContext();
                if (sc.getMajorVersion() == 2 && sc.getMinorVersion() < 5) {
                    String servletContextName = sc.getServletContextName();
                    if (servletContextName != null) {
                        wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + servletContextName + "." + this.getServletName());
                    } else {
                        wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + this.getServletName());
                    }
                } else {
                    wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + ObjectUtils.getDisplayString(sc.getContextPath()) + "/" + this.getServletName());
                }
            }
        }

        wac.setServletContext(this.getServletContext());
        wac.setServletConfig(this.getServletConfig());
        wac.setNamespace(this.getNamespace());
        wac.addApplicationListener(new SourceFilteringListener(wac, new FrameworkServlet.ContextRefreshListener()));
        ConfigurableEnvironment env = wac.getEnvironment();
        if (env instanceof ConfigurableWebEnvironment) {
            ((ConfigurableWebEnvironment)env).initPropertySources(this.getServletContext(), this.getServletConfig());
        }

        this.postProcessWebApplicationContext(wac);
        this.applyInitializers(wac);
        wac.refresh();
    }

    protected WebApplicationContext createWebApplicationContext(WebApplicationContext parent) {
        return this.createWebApplicationContext((ApplicationContext)parent);
    }

    protected void postProcessWebApplicationContext(ConfigurableWebApplicationContext wac) {
    }

    protected void applyInitializers(ConfigurableApplicationContext wac) {
        String globalClassNames = this.getServletContext().getInitParameter("globalInitializerClasses");
        String[] arr$;
        int len$;
        int i$;
        String className;
        if (globalClassNames != null) {
            arr$ = StringUtils.tokenizeToStringArray(globalClassNames, ",; \t\n");
            len$ = arr$.length;

            for(i$ = 0; i$ < len$; ++i$) {
                className = arr$[i$];
                this.contextInitializers.add(this.loadInitializer(className, wac));
            }
        }

        if (this.contextInitializerClasses != null) {
            arr$ = StringUtils.tokenizeToStringArray(this.contextInitializerClasses, ",; \t\n");
            len$ = arr$.length;

            for(i$ = 0; i$ < len$; ++i$) {
                className = arr$[i$];
                this.contextInitializers.add(this.loadInitializer(className, wac));
            }
        }

        AnnotationAwareOrderComparator.sort(this.contextInitializers);
        Iterator i$ = this.contextInitializers.iterator();

        while(i$.hasNext()) {
            ApplicationContextInitializer<ConfigurableApplicationContext> initializer = (ApplicationContextInitializer)i$.next();
            initializer.initialize(wac);
        }

    }

    private ApplicationContextInitializer<ConfigurableApplicationContext> loadInitializer(String className, ConfigurableApplicationContext wac) {
        try {
            Class<?> initializerClass = ClassUtils.forName(className, wac.getClassLoader());
            Class<?> initializerContextClass = GenericTypeResolver.resolveTypeArgument(initializerClass, ApplicationContextInitializer.class);
            if (initializerContextClass != null) {
                Assert.isAssignable(initializerContextClass, wac.getClass(), String.format("Could not add context initializer [%s] since its generic parameter [%s] is not assignable from the type of application context used by this framework servlet [%s]: ", initializerClass.getName(), initializerContextClass.getName(), wac.getClass().getName()));
            }

            return (ApplicationContextInitializer)BeanUtils.instantiateClass(initializerClass, ApplicationContextInitializer.class);
        } catch (Exception var5) {
            throw new IllegalArgumentException(String.format("Could not instantiate class [%s] specified via 'contextInitializerClasses' init-param", className), var5);
        }
    }

    public String getServletContextAttributeName() {
        return SERVLET_CONTEXT_PREFIX + this.getServletName();
    }

    public final WebApplicationContext getWebApplicationContext() {
        return this.webApplicationContext;
    }

    protected void initFrameworkServlet() throws ServletException {
    }

    public void refresh() {
        WebApplicationContext wac = this.getWebApplicationContext();
        if (!(wac instanceof ConfigurableApplicationContext)) {
            throw new IllegalStateException("WebApplicationContext does not support refresh: " + wac);
        } else {
            ((ConfigurableApplicationContext)wac).refresh();
        }
    }

    public void onApplicationEvent(ContextRefreshedEvent event) {
        this.refreshEventReceived = true;
        this.onRefresh(event.getApplicationContext());
    }

    protected void onRefresh(ApplicationContext context) {
    }

    public void destroy() {
        this.getServletContext().log("Destroying Spring FrameworkServlet '" + this.getServletName() + "'");
        if (this.webApplicationContext instanceof ConfigurableApplicationContext) {
            ((ConfigurableApplicationContext)this.webApplicationContext).close();
        }

    }

    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String method = request.getMethod();
        if (method.equalsIgnoreCase(RequestMethod.PATCH.name())) {
            this.processRequest(request, response);
        } else {
            super.service(request, response);
        }

    }

    protected final void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected final void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected final void doPut(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected final void doDelete(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected void doOptions(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        if (this.dispatchOptionsRequest) {
            this.processRequest(request, response);
            if (response.containsHeader("Allow")) {
                return;
            }
        }

        super.doOptions(request, new HttpServletResponseWrapper(response) {
            public void setHeader(String name, String value) {
                if ("Allow".equals(name)) {
                    value = (StringUtils.hasLength(value) ? value + ", " : "") + RequestMethod.PATCH.name();
                }

                super.setHeader(name, value);
            }
        });
    }

    protected void doTrace(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        if (this.dispatchTraceRequest) {
            this.processRequest(request, response);
            if ("message/http".equals(response.getContentType())) {
                return;
            }
        }

        super.doTrace(request, response);
    }

    protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        long startTime = System.currentTimeMillis();
        Throwable failureCause = null;
        LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
        LocaleContext localeContext = this.buildLocaleContext(request);
        RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes requestAttributes = this.buildRequestAttributes(request, response, previousAttributes);
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new FrameworkServlet.RequestBindingInterceptor());
        this.initContextHolders(request, localeContext, requestAttributes);

        try {
            this.doService(request, response);
        } catch (ServletException var18) {
            failureCause = var18;
            throw var18;
        } catch (IOException var19) {
            failureCause = var19;
            throw var19;
        } catch (Throwable var20) {
            failureCause = var20;
            throw new NestedServletException("Request processing failed", var20);
        } finally {
            this.resetContextHolders(request, previousLocaleContext, previousAttributes);
            if (requestAttributes != null) {
                requestAttributes.requestCompleted();
            }

            if (this.logger.isDebugEnabled()) {
                if (failureCause != null) {
                    this.logger.debug("Could not complete request", (Throwable)failureCause);
                } else if (asyncManager.isConcurrentHandlingStarted()) {
                    this.logger.debug("Leaving response open for concurrent processing");
                } else {
                    this.logger.debug("Successfully completed request");
                }
            }

            this.publishRequestHandledEvent(request, startTime, (Throwable)failureCause);
        }

    }

    protected LocaleContext buildLocaleContext(HttpServletRequest request) {
        return new SimpleLocaleContext(request.getLocale());
    }

    protected ServletRequestAttributes buildRequestAttributes(HttpServletRequest request, HttpServletResponse response, RequestAttributes previousAttributes) {
        return previousAttributes != null && !(previousAttributes instanceof ServletRequestAttributes) ? null : new ServletRequestAttributes(request);
    }

    private void initContextHolders(HttpServletRequest request, LocaleContext localeContext, RequestAttributes requestAttributes) {
        if (localeContext != null) {
            LocaleContextHolder.setLocaleContext(localeContext, this.threadContextInheritable);
        }

        if (requestAttributes != null) {
            RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);
        }

        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Bound request context to thread: " + request);
        }

    }

    private void resetContextHolders(HttpServletRequest request, LocaleContext prevLocaleContext, RequestAttributes previousAttributes) {
        LocaleContextHolder.setLocaleContext(prevLocaleContext, this.threadContextInheritable);
        RequestContextHolder.setRequestAttributes(previousAttributes, this.threadContextInheritable);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Cleared thread-bound request context: " + request);
        }

    }

    private void publishRequestHandledEvent(HttpServletRequest request, long startTime, Throwable failureCause) {
        if (this.publishEvents) {
            long processingTime = System.currentTimeMillis() - startTime;
            this.webApplicationContext.publishEvent(new ServletRequestHandledEvent(this, request.getRequestURI(), request.getRemoteAddr(), request.getMethod(), this.getServletConfig().getServletName(), WebUtils.getSessionId(request), this.getUsernameForRequest(request), processingTime, failureCause));
        }

    }

    protected String getUsernameForRequest(HttpServletRequest request) {
        Principal userPrincipal = request.getUserPrincipal();
        return userPrincipal != null ? userPrincipal.getName() : null;
    }

    protected abstract void doService(HttpServletRequest var1, HttpServletResponse var2) throws Exception;

    private class RequestBindingInterceptor extends CallableProcessingInterceptorAdapter {
        private RequestBindingInterceptor() {
        }

        public <T> void preProcess(NativeWebRequest webRequest, Callable<T> task) {
            HttpServletRequest request = (HttpServletRequest)webRequest.getNativeRequest(HttpServletRequest.class);
            if (request != null) {
                HttpServletResponse response = (HttpServletResponse)webRequest.getNativeRequest(HttpServletResponse.class);
                FrameworkServlet.this.initContextHolders(request, FrameworkServlet.this.buildLocaleContext(request), FrameworkServlet.this.buildRequestAttributes(request, response, (RequestAttributes)null));
            }

        }

        public <T> void postProcess(NativeWebRequest webRequest, Callable<T> task, Object concurrentResult) {
            HttpServletRequest request = (HttpServletRequest)webRequest.getNativeRequest(HttpServletRequest.class);
            if (request != null) {
                FrameworkServlet.this.resetContextHolders(request, (LocaleContext)null, (RequestAttributes)null);
            }

        }
    }

    private class ContextRefreshListener implements ApplicationListener<ContextRefreshedEvent> {
        private ContextRefreshListener() {
        }

        public void onApplicationEvent(ContextRefreshedEvent event) {
            FrameworkServlet.this.onApplicationEvent(event);
        }
    }
}
      
ContextLoadListener的作用：在启动web容器时，自动装配ApplicationContext的配置信息，因为他实现了ServletContextListener,ServletContext全局共享
Spring MVC的几个关键类：
  DispatcherServlet
  HandleMapping-》SimpleUrlHandleMapping
  HandlerExcutionChain
  SimpleControllerHandlerAdpater
  ModelAndView
 //执行实际逻辑的类
 public class HandlerExecutionChain {
    private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);
    private final Object handler;
    //拦截器
    @Nullable
    private HandlerInterceptor[] interceptors;
    @Nullable
    private List<HandlerInterceptor> interceptorList;
    private int interceptorIndex;

    public HandlerExecutionChain(Object handler) {
        this(handler, (HandlerInterceptor[])null);
    }

    public HandlerExecutionChain(Object handler, @Nullable HandlerInterceptor... interceptors) {
        this.interceptorIndex = -1;
        if (handler instanceof HandlerExecutionChain) {
            HandlerExecutionChain originalChain = (HandlerExecutionChain)handler;
            this.handler = originalChain.getHandler();
            this.interceptorList = new ArrayList();
            CollectionUtils.mergeArrayIntoCollection(originalChain.getInterceptors(), this.interceptorList);
            CollectionUtils.mergeArrayIntoCollection(interceptors, this.interceptorList);
        } else {
            this.handler = handler;
            this.interceptors = interceptors;
        }

    }

    public Object getHandler() {
        return this.handler;
    }

    public void addInterceptor(HandlerInterceptor interceptor) {
        this.initInterceptorList().add(interceptor);
    }

    public void addInterceptors(HandlerInterceptor... interceptors) {
        if (!ObjectUtils.isEmpty(interceptors)) {
            CollectionUtils.mergeArrayIntoCollection(interceptors, this.initInterceptorList());
        }

    }

    private List<HandlerInterceptor> initInterceptorList() {
        if (this.interceptorList == null) {
            this.interceptorList = new ArrayList();
            if (this.interceptors != null) {
                CollectionUtils.mergeArrayIntoCollection(this.interceptors, this.interceptorList);
            }
        }

        this.interceptors = null;
        return this.interceptorList;
    }

    @Nullable
    public HandlerInterceptor[] getInterceptors() {
        if (this.interceptors == null && this.interceptorList != null) {
            this.interceptors = (HandlerInterceptor[])this.interceptorList.toArray(new HandlerInterceptor[0]);
        }

        return this.interceptors;
    }
    //处理前进行过滤器调用
    boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = 0; i < interceptors.length; this.interceptorIndex = i++) {
                HandlerInterceptor interceptor = interceptors[i];
                if (!interceptor.preHandle(request, response, this.handler)) {
                    this.triggerAfterCompletion(request, response, (Exception)null);
                    return false;
                }
            }
        }

        return true;
    }
    //处理后进行过滤器调用
    void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = interceptors.length - 1; i >= 0; --i) {
                HandlerInterceptor interceptor = interceptors[i];
                interceptor.postHandle(request, response, this.handler, mv);
            }
        }

    }
    //处理完成后触发器
    void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = this.interceptorIndex; i >= 0; --i) {
                HandlerInterceptor interceptor = interceptors[i];

                try {
                    interceptor.afterCompletion(request, response, this.handler, ex);
                } catch (Throwable var8) {
                    logger.error("HandlerInterceptor.afterCompletion threw exception", var8);
                }
            }
        }

    }

    void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = interceptors.length - 1; i >= 0; --i) {
                if (interceptors[i] instanceof AsyncHandlerInterceptor) {
                    try {
                        AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor)interceptors[i];
                        asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
                    } catch (Throwable var6) {
                        logger.error("Interceptor [" + interceptors[i] + "] failed in afterConcurrentHandlingStarted", var6);
                    }
                }
            }
        }

    }

    public String toString() {
        Object handler = this.getHandler();
        StringBuilder sb = new StringBuilder();
        sb.append("HandlerExecutionChain with [").append(handler).append("] and ");
        if (this.interceptorList != null) {
            sb.append(this.interceptorList.size());
        } else if (this.interceptors != null) {
            sb.append(this.interceptors.length);
        } else {
            sb.append(0);
        }

        return sb.append(" interceptors").toString();
    }
}




//spring 加载部分
public class ContextLoader {
    public static final String CONTEXT_ID_PARAM = "contextId";
    public static final String CONFIG_LOCATION_PARAM = "contextConfigLocation";
    public static final String CONTEXT_CLASS_PARAM = "contextClass";
    public static final String CONTEXT_INITIALIZER_CLASSES_PARAM = "contextInitializerClasses";
    public static final String GLOBAL_INITIALIZER_CLASSES_PARAM = "globalInitializerClasses";
    private static final String INIT_PARAM_DELIMITERS = ",; \t\n";
    private static final String DEFAULT_STRATEGIES_PATH = "ContextLoader.properties";
    private static final Properties defaultStrategies;
    private static final Map<ClassLoader, WebApplicationContext> currentContextPerThread;
    @Nullable
    private static volatile WebApplicationContext currentContext;
    @Nullable
    private WebApplicationContext context;
    private final List<ApplicationContextInitializer<ConfigurableApplicationContext>> contextInitializers = new ArrayList();

    public ContextLoader() {
    }

    public ContextLoader(WebApplicationContext context) {
        this.context = context;
    }

    public void setContextInitializers(@Nullable ApplicationContextInitializer<?>... initializers) {
        if (initializers != null) {
            ApplicationContextInitializer[] var2 = initializers;
            int var3 = initializers.length;

            for(int var4 = 0; var4 < var3; ++var4) {
                ApplicationContextInitializer<?> initializer = var2[var4];
                this.contextInitializers.add(initializer);
            }
        }

    }
    //根据根据ServletContext初始化Spring的Context
    public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
        if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
            throw new IllegalStateException("Cannot initialize context because there is already a root application context present - check whether you have multiple ContextLoader* definitions in your web.xml!");
        } else {
            servletContext.log("Initializing Spring root WebApplicationContext");
            Log logger = LogFactory.getLog(ContextLoader.class);
            if (logger.isInfoEnabled()) {
                logger.info("Root WebApplicationContext: initialization started");
            }

            long startTime = System.currentTimeMillis();

            try {
                if (this.context == null) {
                    this.context = this.createWebApplicationContext(servletContext);
                }

                if (this.context instanceof ConfigurableWebApplicationContext) {
                    ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)this.context;
                    if (!cwac.isActive()) {
                        if (cwac.getParent() == null) {
                            ApplicationContext parent = this.loadParentContext(servletContext);
                            cwac.setParent(parent);
                        }

                        this.configureAndRefreshWebApplicationContext(cwac, servletContext);
                    }
                }
                //将spring 的context绑定到ServletContext中
                servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
                ClassLoader ccl = Thread.currentThread().getContextClassLoader();
                if (ccl == ContextLoader.class.getClassLoader()) {
                    currentContext = this.context;
                } else if (ccl != null) {
                    currentContextPerThread.put(ccl, this.context);
                }

                if (logger.isInfoEnabled()) {
                    long elapsedTime = System.currentTimeMillis() - startTime;
                    logger.info("Root WebApplicationContext initialized in " + elapsedTime + " ms");
                }

                return this.context;
            } catch (Error | RuntimeException var8) {
                logger.error("Context initialization failed", var8);
                servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, var8);
                throw var8;
            }
        }
    }

    protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
        Class<?> contextClass = this.determineContextClass(sc);
        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
            throw new ApplicationContextException("Custom context class [" + contextClass.getName() + "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
        } else {
            return (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
        }
    }

    protected Class<?> determineContextClass(ServletContext servletContext) {
        String contextClassName = servletContext.getInitParameter("contextClass");
        if (contextClassName != null) {
            try {
                return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
            } catch (ClassNotFoundException var4) {
                throw new ApplicationContextException("Failed to load custom context class [" + contextClassName + "]", var4);
            }
        } else {
            contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());

            try {
                return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
            } catch (ClassNotFoundException var5) {
                throw new ApplicationContextException("Failed to load default context class [" + contextClassName + "]", var5);
            }
        }
    }

    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
        String configLocationParam;
        if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
            configLocationParam = sc.getInitParameter("contextId");
            if (configLocationParam != null) {
                wac.setId(configLocationParam);
            } else {
                wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + ObjectUtils.getDisplayString(sc.getContextPath()));
            }
        }
        //也可以根据springContext拿到ServletContext
        wac.setServletContext(sc);
        configLocationParam = sc.getInitParameter("contextConfigLocation");
        if (configLocationParam != null) {
            wac.setConfigLocation(configLocationParam);
        }

        ConfigurableEnvironment env = wac.getEnvironment();
        if (env instanceof ConfigurableWebEnvironment) {
            ((ConfigurableWebEnvironment)env).initPropertySources(sc, (ServletConfig)null);
        }

        this.customizeContext(sc, wac);
        wac.refresh();
    }

    protected void customizeContext(ServletContext sc, ConfigurableWebApplicationContext wac) {
        List<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>> initializerClasses = this.determineContextInitializerClasses(sc);
        Iterator var4 = initializerClasses.iterator();

        while(var4.hasNext()) {
            Class<ApplicationContextInitializer<ConfigurableApplicationContext>> initializerClass = (Class)var4.next();
            Class<?> initializerContextClass = GenericTypeResolver.resolveTypeArgument(initializerClass, ApplicationContextInitializer.class);
            if (initializerContextClass != null && !initializerContextClass.isInstance(wac)) {
                throw new ApplicationContextException(String.format("Could not apply context initializer [%s] since its generic parameter [%s] is not assignable from the type of application context used by this context loader: [%s]", initializerClass.getName(), initializerContextClass.getName(), wac.getClass().getName()));
            }

            this.contextInitializers.add(BeanUtils.instantiateClass(initializerClass));
        }

        AnnotationAwareOrderComparator.sort(this.contextInitializers);
        var4 = this.contextInitializers.iterator();

        while(var4.hasNext()) {
            ApplicationContextInitializer<ConfigurableApplicationContext> initializer = (ApplicationContextInitializer)var4.next();
            initializer.initialize(wac);
        }

    }

    protected List<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>> determineContextInitializerClasses(ServletContext servletContext) {
        List<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>> classes = new ArrayList();
        String globalClassNames = servletContext.getInitParameter("globalInitializerClasses");
        int var6;
        if (globalClassNames != null) {
            String[] var4 = StringUtils.tokenizeToStringArray(globalClassNames, ",; \t\n");
            int var5 = var4.length;

            for(var6 = 0; var6 < var5; ++var6) {
                String className = var4[var6];
                classes.add(this.loadInitializerClass(className));
            }
        }

        String localClassNames = servletContext.getInitParameter("contextInitializerClasses");
        if (localClassNames != null) {
            String[] var10 = StringUtils.tokenizeToStringArray(localClassNames, ",; \t\n");
            var6 = var10.length;

            for(int var11 = 0; var11 < var6; ++var11) {
                String className = var10[var11];
                classes.add(this.loadInitializerClass(className));
            }
        }

        return classes;
    }

    private Class<ApplicationContextInitializer<ConfigurableApplicationContext>> loadInitializerClass(String className) {
        try {
            Class<?> clazz = ClassUtils.forName(className, ClassUtils.getDefaultClassLoader());
            if (!ApplicationContextInitializer.class.isAssignableFrom(clazz)) {
                throw new ApplicationContextException("Initializer class does not implement ApplicationContextInitializer interface: " + clazz);
            } else {
                return clazz;
            }
        } catch (ClassNotFoundException var3) {
            throw new ApplicationContextException("Failed to load context initializer class [" + className + "]", var3);
        }
    }

    @Nullable
    protected ApplicationContext loadParentContext(ServletContext servletContext) {
        return null;
    }

    public void closeWebApplicationContext(ServletContext servletContext) {
        servletContext.log("Closing Spring root WebApplicationContext");
        boolean var6 = false;

        try {
            var6 = true;
            if (this.context instanceof ConfigurableWebApplicationContext) {
                ((ConfigurableWebApplicationContext)this.context).close();
                var6 = false;
            } else {
                var6 = false;
            }
        } finally {
            if (var6) {
                ClassLoader ccl = Thread.currentThread().getContextClassLoader();
                if (ccl == ContextLoader.class.getClassLoader()) {
                    currentContext = null;
                } else if (ccl != null) {
                    currentContextPerThread.remove(ccl);
                }

                servletContext.removeAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
            }
        }

        ClassLoader ccl = Thread.currentThread().getContextClassLoader();
        if (ccl == ContextLoader.class.getClassLoader()) {
            currentContext = null;
        } else if (ccl != null) {
            currentContextPerThread.remove(ccl);
        }

        servletContext.removeAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
    }

    @Nullable
    public static WebApplicationContext getCurrentWebApplicationContext() {
        ClassLoader ccl = Thread.currentThread().getContextClassLoader();
        if (ccl != null) {
            WebApplicationContext ccpt = (WebApplicationContext)currentContextPerThread.get(ccl);
            if (ccpt != null) {
                return ccpt;
            }
        }

        return currentContext;
    }

    static {
        try {
            ClassPathResource resource = new ClassPathResource("ContextLoader.properties", ContextLoader.class);
            defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
        } catch (IOException var1) {
            throw new IllegalStateException("Could not load 'ContextLoader.properties': " + var1.getMessage());
        }

        currentContextPerThread = new ConcurrentHashMap(1);
    }
}
//上面的类是监听器逻辑的实际实现，这里是监听器
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }

    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }

    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }

    public void contextDestroyed(ServletContextEvent event) {
        this.closeWebApplicationContext(event.getServletContext());
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }
}
以上两个类只负责创建Spring的WebApplication并记录到servletContext中
真正实现初始化由DisapatcherServlet实现：
  
public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
    protected final Log logger = LogFactory.getLog(this.getClass());
    @Nullable
    private ConfigurableEnvironment environment;
    private final Set<String> requiredProperties = new HashSet(4);

    public HttpServletBean() {
    }

    protected final void addRequiredProperty(String property) {
        this.requiredProperties.add(property);
    }

    public void setEnvironment(Environment environment) {
        Assert.isInstanceOf(ConfigurableEnvironment.class, environment, "ConfigurableEnvironment required");
        this.environment = (ConfigurableEnvironment)environment;
    }

    public ConfigurableEnvironment getEnvironment() {
        if (this.environment == null) {
            this.environment = this.createEnvironment();
        }

        return this.environment;
    }

    protected ConfigurableEnvironment createEnvironment() {
        return new StandardServletEnvironment();
    }

    public final void init() throws ServletException {
    //解析init-param并封装pvs
        PropertyValues pvs = new HttpServletBean.ServletConfigPropertyValues(this.getServletConfig(), this.requiredProperties);
        if (!pvs.isEmpty()) {
            try {
                //转换为beanWrapper
                BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
                ResourceLoader resourceLoader = new ServletContextResourceLoader(this.getServletContext());
                //注册属性编辑器
                bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.getEnvironment()));
                //子类实现
                this.initBeanWrapper(bw);
                //属性注入
                bw.setPropertyValues(pvs, true);
            } catch (BeansException var4) {
                if (this.logger.isErrorEnabled()) {
                    this.logger.error("Failed to set bean properties on servlet '" + this.getServletName() + "'", var4);
                }

                throw var4;
            }
        }
        //子类实现
        this.initServletBean();
    }

    protected void initBeanWrapper(BeanWrapper bw) throws BeansException {
    }

    protected void initServletBean() throws ServletException {
    }

    @Nullable
    public String getServletName() {
        return this.getServletConfig() != null ? this.getServletConfig().getServletName() : null;
    }

    private static class ServletConfigPropertyValues extends MutablePropertyValues {
        public ServletConfigPropertyValues(ServletConfig config, Set<String> requiredProperties) throws ServletException {
            Set<String> missingProps = !CollectionUtils.isEmpty(requiredProperties) ? new HashSet(requiredProperties) : null;
            Enumeration paramNames = config.getInitParameterNames();

            while(paramNames.hasMoreElements()) {
                String property = (String)paramNames.nextElement();
                Object value = config.getInitParameter(property);
                this.addPropertyValue(new PropertyValue(property, value));
                if (missingProps != null) {
                    missingProps.remove(property);
                }
            }

            if (!CollectionUtils.isEmpty(missingProps)) {
                throw new ServletException("Initialization from ServletConfig for servlet '" + config.getServletName() + "' failed; the following required properties were missing: " + StringUtils.collectionToDelimitedString(missingProps, ", "));
            }
        }
    }
}
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
    public static final String DEFAULT_NAMESPACE_SUFFIX = "-servlet";
    public static final Class<?> DEFAULT_CONTEXT_CLASS = XmlWebApplicationContext.class;
    public static final String SERVLET_CONTEXT_PREFIX = FrameworkServlet.class.getName() + ".CONTEXT.";
    private static final String INIT_PARAM_DELIMITERS = ",; \t\n";
    @Nullable
    private String contextAttribute;
    private Class<?> contextClass;
    @Nullable
    private String contextId;
    @Nullable
    private String namespace;
    @Nullable
    private String contextConfigLocation;
    private final List<ApplicationContextInitializer<ConfigurableApplicationContext>> contextInitializers;
    @Nullable
    private String contextInitializerClasses;
    private boolean publishContext;
    private boolean publishEvents;
    private boolean threadContextInheritable;
    private boolean dispatchOptionsRequest;
    private boolean dispatchTraceRequest;
    private boolean enableLoggingRequestDetails;
    @Nullable
    private WebApplicationContext webApplicationContext;
    private boolean webApplicationContextInjected;
    private volatile boolean refreshEventReceived;
    private final Object onRefreshMonitor;

    public FrameworkServlet() {
        this.contextClass = DEFAULT_CONTEXT_CLASS;
        this.contextInitializers = new ArrayList();
        this.publishContext = true;
        this.publishEvents = true;
        this.threadContextInheritable = false;
        this.dispatchOptionsRequest = false;
        this.dispatchTraceRequest = false;
        this.enableLoggingRequestDetails = false;
        this.webApplicationContextInjected = false;
        this.refreshEventReceived = false;
        this.onRefreshMonitor = new Object();
    }

    public FrameworkServlet(WebApplicationContext webApplicationContext) {
        this.contextClass = DEFAULT_CONTEXT_CLASS;
        this.contextInitializers = new ArrayList();
        this.publishContext = true;
        this.publishEvents = true;
        this.threadContextInheritable = false;
        this.dispatchOptionsRequest = false;
        this.dispatchTraceRequest = false;
        this.enableLoggingRequestDetails = false;
        this.webApplicationContextInjected = false;
        this.refreshEventReceived = false;
        this.onRefreshMonitor = new Object();
        this.webApplicationContext = webApplicationContext;
    }

    public void setContextAttribute(@Nullable String contextAttribute) {
        this.contextAttribute = contextAttribute;
    }

    @Nullable
    public String getContextAttribute() {
        return this.contextAttribute;
    }

    public void setContextClass(Class<?> contextClass) {
        this.contextClass = contextClass;
    }

    public Class<?> getContextClass() {
        return this.contextClass;
    }

    public void setContextId(@Nullable String contextId) {
        this.contextId = contextId;
    }

    @Nullable
    public String getContextId() {
        return this.contextId;
    }

    public void setNamespace(String namespace) {
        this.namespace = namespace;
    }

    public String getNamespace() {
        return this.namespace != null ? this.namespace : this.getServletName() + "-servlet";
    }

    public void setContextConfigLocation(@Nullable String contextConfigLocation) {
        this.contextConfigLocation = contextConfigLocation;
    }

    @Nullable
    public String getContextConfigLocation() {
        return this.contextConfigLocation;
    }

    public void setContextInitializers(@Nullable ApplicationContextInitializer<?>... initializers) {
        if (initializers != null) {
            ApplicationContextInitializer[] var2 = initializers;
            int var3 = initializers.length;

            for(int var4 = 0; var4 < var3; ++var4) {
                ApplicationContextInitializer<?> initializer = var2[var4];
                this.contextInitializers.add(initializer);
            }
        }

    }

    public void setContextInitializerClasses(String contextInitializerClasses) {
        this.contextInitializerClasses = contextInitializerClasses;
    }

    public void setPublishContext(boolean publishContext) {
        this.publishContext = publishContext;
    }

    public void setPublishEvents(boolean publishEvents) {
        this.publishEvents = publishEvents;
    }

    public void setThreadContextInheritable(boolean threadContextInheritable) {
        this.threadContextInheritable = threadContextInheritable;
    }

    public void setDispatchOptionsRequest(boolean dispatchOptionsRequest) {
        this.dispatchOptionsRequest = dispatchOptionsRequest;
    }

    public void setDispatchTraceRequest(boolean dispatchTraceRequest) {
        this.dispatchTraceRequest = dispatchTraceRequest;
    }

    public void setEnableLoggingRequestDetails(boolean enable) {
        this.enableLoggingRequestDetails = enable;
    }

    public boolean isEnableLoggingRequestDetails() {
        return this.enableLoggingRequestDetails;
    }

    public void setApplicationContext(ApplicationContext applicationContext) {
        if (this.webApplicationContext == null && applicationContext instanceof WebApplicationContext) {
            this.webApplicationContext = (WebApplicationContext)applicationContext;
            this.webApplicationContextInjected = true;
        }

    }

    protected final void initServletBean() throws ServletException {
        this.getServletContext().log("Initializing Spring " + this.getClass().getSimpleName() + " '" + this.getServletName() + "'");
        if (this.logger.isInfoEnabled()) {
            this.logger.info("Initializing Servlet '" + this.getServletName() + "'");
        }

        long startTime = System.currentTimeMillis();

        try {
            //initWebApplicationContext关键
            this.webApplicationContext = this.initWebApplicationContext();
            //子类覆盖
            this.initFrameworkServlet();
        } catch (RuntimeException | ServletException var4) {
            this.logger.error("Context initialization failed", var4);
            throw var4;
        }

        if (this.logger.isDebugEnabled()) {
            String value = this.enableLoggingRequestDetails ? "shown which may lead to unsafe logging of potentially sensitive data" : "masked to prevent unsafe logging of potentially sensitive data";
            this.logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails + "': request parameters and headers will be " + value);
        }

        if (this.logger.isInfoEnabled()) {
            this.logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
        }

    }

    protected WebApplicationContext initWebApplicationContext() {
        //判断有没有
        WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
        WebApplicationContext wac = null;
        if (this.webApplicationContext != null) {
            wac = this.webApplicationContext;
            if (wac instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)wac;
                if (!cwac.isActive()) {
                    if (cwac.getParent() == null) {
                        cwac.setParent(rootContext);
                    }
                    //刷新上下文环境
                    this.configureAndRefreshWebApplicationContext(cwac);
                }
            }
        }

        if (wac == null) {
            wac = this.findWebApplicationContext();
        }

        if (wac == null) {
            wac = this.createWebApplicationContext(rootContext);
        }

        if (!this.refreshEventReceived) {
            synchronized(this.onRefreshMonitor) {
                this.onRefresh(wac);
            }
        }

        if (this.publishContext) {
            String attrName = this.getServletContextAttributeName();
            this.getServletContext().setAttribute(attrName, wac);
        }

        return wac;
    }
    //根据key值从ServletContext中查找
    @Nullable
    protected WebApplicationContext findWebApplicationContext() {
        String attrName = this.getContextAttribute();
        if (attrName == null) {
            return null;
        } else {
            WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext(), attrName);
            if (wac == null) {
                throw new IllegalStateException("No WebApplicationContext found: initializer not registered?");
            } else {
                return wac;
            }
        }
    }

    protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
        Class<?> contextClass = this.getContextClass();
        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
            throw new ApplicationContextException("Fatal initialization error in servlet with name '" + this.getServletName() + "': custom WebApplicationContext class [" + contextClass.getName() + "] is not of type ConfigurableWebApplicationContext");
        } else {
            //通过反射实例化
            ConfigurableWebApplicationContext wac = (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
            wac.setEnvironment(this.getEnvironment());
            wac.setParent(parent);
            String configLocation = this.getContextConfigLocation();
            if (configLocation != null) {
                wac.setConfigLocation(configLocation);
            }
            //无论是通过构造函数注入还是单独创建，都免不了会调用这个函数对已经创建的WebApplicationContext实例进行配置及刷新
            this.configureAndRefreshWebApplicationContext(wac);
            return wac;
        }
    }

    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
        if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
            if (this.contextId != null) {
                wac.setId(this.contextId);
            } else {
                wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + ObjectUtils.getDisplayString(this.getServletContext().getContextPath()) + '/' + this.getServletName());
            }
        }

        wac.setServletContext(this.getServletContext());
        wac.setServletConfig(this.getServletConfig());
        wac.setNamespace(this.getNamespace());
        wac.addApplicationListener(new SourceFilteringListener(wac, new FrameworkServlet.ContextRefreshListener()));
        ConfigurableEnvironment env = wac.getEnvironment();
        if (env instanceof ConfigurableWebEnvironment) {
            ((ConfigurableWebEnvironment)env).initPropertySources(this.getServletContext(), this.getServletConfig());
        }

        this.postProcessWebApplicationContext(wac);
        this.applyInitializers(wac);
        wac.refresh();
    }

    protected WebApplicationContext createWebApplicationContext(@Nullable WebApplicationContext parent) {
        return this.createWebApplicationContext((ApplicationContext)parent);
    }

    protected void postProcessWebApplicationContext(ConfigurableWebApplicationContext wac) {
    }

    protected void applyInitializers(ConfigurableApplicationContext wac) {
        String globalClassNames = this.getServletContext().getInitParameter("globalInitializerClasses");
        String[] var3;
        int var4;
        int var5;
        String className;
        if (globalClassNames != null) {
            var3 = StringUtils.tokenizeToStringArray(globalClassNames, ",; \t\n");
            var4 = var3.length;

            for(var5 = 0; var5 < var4; ++var5) {
                className = var3[var5];
                this.contextInitializers.add(this.loadInitializer(className, wac));
            }
        }

        if (this.contextInitializerClasses != null) {
            var3 = StringUtils.tokenizeToStringArray(this.contextInitializerClasses, ",; \t\n");
            var4 = var3.length;

            for(var5 = 0; var5 < var4; ++var5) {
                className = var3[var5];
                this.contextInitializers.add(this.loadInitializer(className, wac));
            }
        }

        AnnotationAwareOrderComparator.sort(this.contextInitializers);
        Iterator var7 = this.contextInitializers.iterator();

        while(var7.hasNext()) {
            ApplicationContextInitializer<ConfigurableApplicationContext> initializer = (ApplicationContextInitializer)var7.next();
            initializer.initialize(wac);
        }

    }

    private ApplicationContextInitializer<ConfigurableApplicationContext> loadInitializer(String className, ConfigurableApplicationContext wac) {
        try {
            Class<?> initializerClass = ClassUtils.forName(className, wac.getClassLoader());
            Class<?> initializerContextClass = GenericTypeResolver.resolveTypeArgument(initializerClass, ApplicationContextInitializer.class);
            if (initializerContextClass != null && !initializerContextClass.isInstance(wac)) {
                throw new ApplicationContextException(String.format("Could not apply context initializer [%s] since its generic parameter [%s] is not assignable from the type of application context used by this framework servlet: [%s]", initializerClass.getName(), initializerContextClass.getName(), wac.getClass().getName()));
            } else {
                return (ApplicationContextInitializer)BeanUtils.instantiateClass(initializerClass, ApplicationContextInitializer.class);
            }
        } catch (ClassNotFoundException var5) {
            throw new ApplicationContextException(String.format("Could not load class [%s] specified via 'contextInitializerClasses' init-param", className), var5);
        }
    }

    public String getServletContextAttributeName() {
        return SERVLET_CONTEXT_PREFIX + this.getServletName();
    }

    @Nullable
    public final WebApplicationContext getWebApplicationContext() {
        return this.webApplicationContext;
    }

    protected void initFrameworkServlet() throws ServletException {
    }
    
    public void refresh() {
        WebApplicationContext wac = this.getWebApplicationContext();
        if (!(wac instanceof ConfigurableApplicationContext)) {
            throw new IllegalStateException("WebApplicationContext does not support refresh: " + wac);
        } else {
            ((ConfigurableApplicationContext)wac).refresh();
        }
    }

    public void onApplicationEvent(ContextRefreshedEvent event) {
        this.refreshEventReceived = true;
        synchronized(this.onRefreshMonitor) {
            this.onRefresh(event.getApplicationContext());
        }
    }

    protected void onRefresh(ApplicationContext context) {
    }

    public void destroy() {
        this.getServletContext().log("Destroying Spring FrameworkServlet '" + this.getServletName() + "'");
        if (this.webApplicationContext instanceof ConfigurableApplicationContext && !this.webApplicationContextInjected) {
            ((ConfigurableApplicationContext)this.webApplicationContext).close();
        }

    }

    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
        if (httpMethod != HttpMethod.PATCH && httpMethod != null) {
            super.service(request, response);
        } else {
            this.processRequest(request, response);
        }

    }

    protected final void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected final void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected final void doPut(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected final void doDelete(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.processRequest(request, response);
    }

    protected void doOptions(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        if (this.dispatchOptionsRequest || CorsUtils.isPreFlightRequest(request)) {
            this.processRequest(request, response);
            if (response.containsHeader("Allow")) {
                return;
            }
        }

        super.doOptions(request, new HttpServletResponseWrapper(response) {
            public void setHeader(String name, String value) {
                if ("Allow".equals(name)) {
                    value = (StringUtils.hasLength(value) ? value + ", " : "") + HttpMethod.PATCH.name();
                }

                super.setHeader(name, value);
            }
        });
    }

    protected void doTrace(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        if (this.dispatchTraceRequest) {
            this.processRequest(request, response);
            if ("message/http".equals(response.getContentType())) {
                return;
            }
        }

        super.doTrace(request, response);
    }

    protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        long startTime = System.currentTimeMillis();
        Throwable failureCause = null;
        LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
        LocaleContext localeContext = this.buildLocaleContext(request);
        RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes requestAttributes = this.buildRequestAttributes(request, response, previousAttributes);
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new FrameworkServlet.RequestBindingInterceptor());
        this.initContextHolders(request, localeContext, requestAttributes);

        try {
            this.doService(request, response);
        } catch (IOException | ServletException var16) {
            failureCause = var16;
            throw var16;
        } catch (Throwable var17) {
            failureCause = var17;
            throw new NestedServletException("Request processing failed", var17);
        } finally {
            this.resetContextHolders(request, previousLocaleContext, previousAttributes);
            if (requestAttributes != null) {
                requestAttributes.requestCompleted();
            }

            this.logResult(request, response, (Throwable)failureCause, asyncManager);
            this.publishRequestHandledEvent(request, response, startTime, (Throwable)failureCause);
        }

    }

    @Nullable
    protected LocaleContext buildLocaleContext(HttpServletRequest request) {
        return new SimpleLocaleContext(request.getLocale());
    }

    @Nullable
    protected ServletRequestAttributes buildRequestAttributes(HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable RequestAttributes previousAttributes) {
        return previousAttributes != null && !(previousAttributes instanceof ServletRequestAttributes) ? null : new ServletRequestAttributes(request, response);
    }

    private void initContextHolders(HttpServletRequest request, @Nullable LocaleContext localeContext, @Nullable RequestAttributes requestAttributes) {
        if (localeContext != null) {
            LocaleContextHolder.setLocaleContext(localeContext, this.threadContextInheritable);
        }

        if (requestAttributes != null) {
            RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);
        }

    }

    private void resetContextHolders(HttpServletRequest request, @Nullable LocaleContext prevLocaleContext, @Nullable RequestAttributes previousAttributes) {
        LocaleContextHolder.setLocaleContext(prevLocaleContext, this.threadContextInheritable);
        RequestContextHolder.setRequestAttributes(previousAttributes, this.threadContextInheritable);
    }

    private void logResult(HttpServletRequest request, HttpServletResponse response, @Nullable Throwable failureCause, WebAsyncManager asyncManager) {
        if (this.logger.isDebugEnabled()) {
            String dispatchType = request.getDispatcherType().name();
            boolean initialDispatch = request.getDispatcherType().equals(DispatcherType.REQUEST);
            if (failureCause != null) {
                if (!initialDispatch) {
                    if (this.logger.isDebugEnabled()) {
                        this.logger.debug("Unresolved failure from \"" + dispatchType + "\" dispatch: " + failureCause);
                    }
                } else if (this.logger.isTraceEnabled()) {
                    this.logger.trace("Failed to complete request", failureCause);
                } else {
                    this.logger.debug("Failed to complete request: " + failureCause);
                }

            } else if (asyncManager.isConcurrentHandlingStarted()) {
                this.logger.debug("Exiting but response remains open for further handling");
            } else {
                int status = response.getStatus();
                String headers = "";
                if (this.logger.isTraceEnabled()) {
                    Collection<String> names = response.getHeaderNames();
                    if (this.enableLoggingRequestDetails) {
                        headers = (String)names.stream().map((name) -> {
                            return name + ":" + response.getHeaders(name);
                        }).collect(Collectors.joining(", "));
                    } else {
                        headers = names.isEmpty() ? "" : "masked";
                    }

                    headers = ", headers={" + headers + "}";
                }

                if (!initialDispatch) {
                    this.logger.debug("Exiting from \"" + dispatchType + "\" dispatch, status " + status + headers);
                } else {
                    HttpStatus httpStatus = HttpStatus.resolve(status);
                    this.logger.debug("Completed " + (httpStatus != null ? httpStatus : status) + headers);
                }

            }
        }
    }

    private void publishRequestHandledEvent(HttpServletRequest request, HttpServletResponse response, long startTime, @Nullable Throwable failureCause) {
        if (this.publishEvents && this.webApplicationContext != null) {
            long processingTime = System.currentTimeMillis() - startTime;
            this.webApplicationContext.publishEvent(new ServletRequestHandledEvent(this, request.getRequestURI(), request.getRemoteAddr(), request.getMethod(), this.getServletConfig().getServletName(), WebUtils.getSessionId(request), this.getUsernameForRequest(request), processingTime, failureCause, response.getStatus()));
        }

    }

    @Nullable
    protected String getUsernameForRequest(HttpServletRequest request) {
        Principal userPrincipal = request.getUserPrincipal();
        return userPrincipal != null ? userPrincipal.getName() : null;
    }

    protected abstract void doService(HttpServletRequest var1, HttpServletResponse var2) throws Exception;

    private class RequestBindingInterceptor implements CallableProcessingInterceptor {
        private RequestBindingInterceptor() {
        }

        public <T> void preProcess(NativeWebRequest webRequest, Callable<T> task) {
            HttpServletRequest request = (HttpServletRequest)webRequest.getNativeRequest(HttpServletRequest.class);
            if (request != null) {
                HttpServletResponse response = (HttpServletResponse)webRequest.getNativeResponse(HttpServletResponse.class);
                FrameworkServlet.this.initContextHolders(request, FrameworkServlet.this.buildLocaleContext(request), FrameworkServlet.this.buildRequestAttributes(request, response, (RequestAttributes)null));
            }

        }

        public <T> void postProcess(NativeWebRequest webRequest, Callable<T> task, Object concurrentResult) {
            HttpServletRequest request = (HttpServletRequest)webRequest.getNativeRequest(HttpServletRequest.class);
            if (request != null) {
                FrameworkServlet.this.resetContextHolders(request, (LocaleContext)null, (RequestAttributes)null);
            }

        }
    }

    private class ContextRefreshListener implements ApplicationListener<ContextRefreshedEvent> {
        private ContextRefreshListener() {
        }

        public void onApplicationEvent(ContextRefreshedEvent event) {
            FrameworkServlet.this.onApplicationEvent(event);
        }
    }
}


public class DispatcherServlet extends FrameworkServlet {
    public static final String MULTIPART_RESOLVER_BEAN_NAME = "multipartResolver";
    public static final String LOCALE_RESOLVER_BEAN_NAME = "localeResolver";
    public static final String THEME_RESOLVER_BEAN_NAME = "themeResolver";
    public static final String HANDLER_MAPPING_BEAN_NAME = "handlerMapping";
    public static final String HANDLER_ADAPTER_BEAN_NAME = "handlerAdapter";
    public static final String HANDLER_EXCEPTION_RESOLVER_BEAN_NAME = "handlerExceptionResolver";
    public static final String REQUEST_TO_VIEW_NAME_TRANSLATOR_BEAN_NAME = "viewNameTranslator";
    public static final String VIEW_RESOLVER_BEAN_NAME = "viewResolver";
    public static final String FLASH_MAP_MANAGER_BEAN_NAME = "flashMapManager";
    public static final String WEB_APPLICATION_CONTEXT_ATTRIBUTE = DispatcherServlet.class.getName() + ".CONTEXT";
    public static final String LOCALE_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".LOCALE_RESOLVER";
    public static final String THEME_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_RESOLVER";
    public static final String THEME_SOURCE_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_SOURCE";
    public static final String INPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".INPUT_FLASH_MAP";
    public static final String OUTPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".OUTPUT_FLASH_MAP";
    public static final String FLASH_MAP_MANAGER_ATTRIBUTE = DispatcherServlet.class.getName() + ".FLASH_MAP_MANAGER";
    public static final String EXCEPTION_ATTRIBUTE = DispatcherServlet.class.getName() + ".EXCEPTION";
    public static final String PAGE_NOT_FOUND_LOG_CATEGORY = "org.springframework.web.servlet.PageNotFound";
    private static final String DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties";
    private static final String DEFAULT_STRATEGIES_PREFIX = "org.springframework.web.servlet";
    protected static final Log pageNotFoundLogger = LogFactory.getLog("org.springframework.web.servlet.PageNotFound");
    private static final Properties defaultStrategies;
    private boolean detectAllHandlerMappings = true;
    private boolean detectAllHandlerAdapters = true;
    private boolean detectAllHandlerExceptionResolvers = true;
    private boolean detectAllViewResolvers = true;
    private boolean throwExceptionIfNoHandlerFound = false;
    private boolean cleanupAfterInclude = true;
    @Nullable
    private MultipartResolver multipartResolver;
    @Nullable
    private LocaleResolver localeResolver;
    @Nullable
    private ThemeResolver themeResolver;
    @Nullable
    private List<HandlerMapping> handlerMappings;
    @Nullable
    private List<HandlerAdapter> handlerAdapters;
    @Nullable
    private List<HandlerExceptionResolver> handlerExceptionResolvers;
    @Nullable
    private RequestToViewNameTranslator viewNameTranslator;
    @Nullable
    private FlashMapManager flashMapManager;
    @Nullable
    private List<ViewResolver> viewResolvers;

    public DispatcherServlet() {
        this.setDispatchOptionsRequest(true);
    }

    public DispatcherServlet(WebApplicationContext webApplicationContext) {
        super(webApplicationContext);
        this.setDispatchOptionsRequest(true);
    }

    public void setDetectAllHandlerMappings(boolean detectAllHandlerMappings) {
        this.detectAllHandlerMappings = detectAllHandlerMappings;
    }

    public void setDetectAllHandlerAdapters(boolean detectAllHandlerAdapters) {
        this.detectAllHandlerAdapters = detectAllHandlerAdapters;
    }

    public void setDetectAllHandlerExceptionResolvers(boolean detectAllHandlerExceptionResolvers) {
        this.detectAllHandlerExceptionResolvers = detectAllHandlerExceptionResolvers;
    }

    public void setDetectAllViewResolvers(boolean detectAllViewResolvers) {
        this.detectAllViewResolvers = detectAllViewResolvers;
    }

    public void setThrowExceptionIfNoHandlerFound(boolean throwExceptionIfNoHandlerFound) {
        this.throwExceptionIfNoHandlerFound = throwExceptionIfNoHandlerFound;
    }

    public void setCleanupAfterInclude(boolean cleanupAfterInclude) {
        this.cleanupAfterInclude = cleanupAfterInclude;
    }
  //刷新Spring在web功能实现中所必须使用的全局变量
    protected void onRefresh(ApplicationContext context) {
        this.initStrategies(context);
    }

    protected void initStrategies(ApplicationContext context) {
        this.initMultipartResolver(context);
        this.initLocaleResolver(context);
        this.initThemeResolver(context);
        this.initHandlerMappings(context);
        this.initHandlerAdapters(context);
        this.initHandlerExceptionResolvers(context);
        this.initRequestToViewNameTranslator(context);
        this.initViewResolvers(context);
        this.initFlashMapManager(context);
    }
    //MultiPartResolver用来处理文件上传，默认情况下，Spring没有multipart处理，因为一些开发者想要自己处理。如果想使用Spring的multipart，则需要zaiweb应用的上下文中添加multipart解析器，这样，每一个请求就会被检查是否包含multipart，如果包含，上下文中定义的MultipartResolver就会解析
    private void initMultipartResolver(ApplicationContext context) {
        try {
            this.multipartResolver = (MultipartResolver)context.getBean("multipartResolver", MultipartResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.multipartResolver);
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.multipartResolver.getClass().getSimpleName());
            }
        } catch (NoSuchBeanDefinitionException var3) {
            this.multipartResolver = null;
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No MultipartResolver 'multipartResolver' declared");
            }
        }

    }

    private void initLocaleResolver(ApplicationContext context) {
        try {
            this.localeResolver = (LocaleResolver)context.getBean("localeResolver", LocaleResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.localeResolver);
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.localeResolver.getClass().getSimpleName());
            }
        } catch (NoSuchBeanDefinitionException var3) {
            this.localeResolver = (LocaleResolver)this.getDefaultStrategy(context, LocaleResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No LocaleResolver 'localeResolver': using default [" + this.localeResolver.getClass().getSimpleName() + "]");
            }
        }

    }

    private void initThemeResolver(ApplicationContext context) {
        try {
            this.themeResolver = (ThemeResolver)context.getBean("themeResolver", ThemeResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.themeResolver);
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.themeResolver.getClass().getSimpleName());
            }
        } catch (NoSuchBeanDefinitionException var3) {
            this.themeResolver = (ThemeResolver)this.getDefaultStrategy(context, ThemeResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No ThemeResolver 'themeResolver': using default [" + this.themeResolver.getClass().getSimpleName() + "]");
            }
        }

    }

    private void initHandlerMappings(ApplicationContext context) {
        this.handlerMappings = null;
        if (this.detectAllHandlerMappings) {
            Map<String, HandlerMapping> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.handlerMappings = new ArrayList(matchingBeans.values());
                AnnotationAwareOrderComparator.sort(this.handlerMappings);
            }
        } else {
            try {
                HandlerMapping hm = (HandlerMapping)context.getBean("handlerMapping", HandlerMapping.class);
                this.handlerMappings = Collections.singletonList(hm);
            } catch (NoSuchBeanDefinitionException var3) {
            }
        }

        if (this.handlerMappings == null) {
            this.handlerMappings = this.getDefaultStrategies(context, HandlerMapping.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No HandlerMappings declared for servlet '" + this.getServletName() + "': using default strategies from DispatcherServlet.properties");
            }
        }

    }

    private void initHandlerAdapters(ApplicationContext context) {
        this.handlerAdapters = null;
        if (this.detectAllHandlerAdapters) {
            Map<String, HandlerAdapter> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerAdapter.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.handlerAdapters = new ArrayList(matchingBeans.values());
                AnnotationAwareOrderComparator.sort(this.handlerAdapters);
            }
        } else {
            try {
                HandlerAdapter ha = (HandlerAdapter)context.getBean("handlerAdapter", HandlerAdapter.class);
                this.handlerAdapters = Collections.singletonList(ha);
            } catch (NoSuchBeanDefinitionException var3) {
            }
        }

        if (this.handlerAdapters == null) {
            this.handlerAdapters = this.getDefaultStrategies(context, HandlerAdapter.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No HandlerAdapters declared for servlet '" + this.getServletName() + "': using default strategies from DispatcherServlet.properties");
            }
        }

    }

    private void initHandlerExceptionResolvers(ApplicationContext context) {
        this.handlerExceptionResolvers = null;
        if (this.detectAllHandlerExceptionResolvers) {
            Map<String, HandlerExceptionResolver> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerExceptionResolver.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.handlerExceptionResolvers = new ArrayList(matchingBeans.values());
                AnnotationAwareOrderComparator.sort(this.handlerExceptionResolvers);
            }
        } else {
            try {
                HandlerExceptionResolver her = (HandlerExceptionResolver)context.getBean("handlerExceptionResolver", HandlerExceptionResolver.class);
                this.handlerExceptionResolvers = Collections.singletonList(her);
            } catch (NoSuchBeanDefinitionException var3) {
            }
        }

        if (this.handlerExceptionResolvers == null) {
            this.handlerExceptionResolvers = this.getDefaultStrategies(context, HandlerExceptionResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No HandlerExceptionResolvers declared in servlet '" + this.getServletName() + "': using default strategies from DispatcherServlet.properties");
            }
        }

    }

    private void initRequestToViewNameTranslator(ApplicationContext context) {
        try {
            this.viewNameTranslator = (RequestToViewNameTranslator)context.getBean("viewNameTranslator", RequestToViewNameTranslator.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.viewNameTranslator.getClass().getSimpleName());
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.viewNameTranslator);
            }
        } catch (NoSuchBeanDefinitionException var3) {
            this.viewNameTranslator = (RequestToViewNameTranslator)this.getDefaultStrategy(context, RequestToViewNameTranslator.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No RequestToViewNameTranslator 'viewNameTranslator': using default [" + this.viewNameTranslator.getClass().getSimpleName() + "]");
            }
        }

    }

    private void initViewResolvers(ApplicationContext context) {
        this.viewResolvers = null;
        if (this.detectAllViewResolvers) {
            Map<String, ViewResolver> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, ViewResolver.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.viewResolvers = new ArrayList(matchingBeans.values());
                AnnotationAwareOrderComparator.sort(this.viewResolvers);
            }
        } else {
            try {
                ViewResolver vr = (ViewResolver)context.getBean("viewResolver", ViewResolver.class);
                this.viewResolvers = Collections.singletonList(vr);
            } catch (NoSuchBeanDefinitionException var3) {
            }
        }

        if (this.viewResolvers == null) {
            this.viewResolvers = this.getDefaultStrategies(context, ViewResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No ViewResolvers declared for servlet '" + this.getServletName() + "': using default strategies from DispatcherServlet.properties");
            }
        }

    }

    private void initFlashMapManager(ApplicationContext context) {
        try {
            this.flashMapManager = (FlashMapManager)context.getBean("flashMapManager", FlashMapManager.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.flashMapManager.getClass().getSimpleName());
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.flashMapManager);
            }
        } catch (NoSuchBeanDefinitionException var3) {
            this.flashMapManager = (FlashMapManager)this.getDefaultStrategy(context, FlashMapManager.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No FlashMapManager 'flashMapManager': using default [" + this.flashMapManager.getClass().getSimpleName() + "]");
            }
        }

    }

    @Nullable
    public final ThemeSource getThemeSource() {
        return this.getWebApplicationContext() instanceof ThemeSource ? (ThemeSource)this.getWebApplicationContext() : null;
    }

    @Nullable
    public final MultipartResolver getMultipartResolver() {
        return this.multipartResolver;
    }

    @Nullable
    public final List<HandlerMapping> getHandlerMappings() {
        return this.handlerMappings != null ? Collections.unmodifiableList(this.handlerMappings) : null;
    }

    protected <T> T getDefaultStrategy(ApplicationContext context, Class<T> strategyInterface) {
        List<T> strategies = this.getDefaultStrategies(context, strategyInterface);
        if (strategies.size() != 1) {
            throw new BeanInitializationException("DispatcherServlet needs exactly 1 strategy for interface [" + strategyInterface.getName() + "]");
        } else {
            return strategies.get(0);
        }
    }

    protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {
        String key = strategyInterface.getName();
        String value = defaultStrategies.getProperty(key);
        if (value == null) {
            return new LinkedList();
        } else {
            String[] classNames = StringUtils.commaDelimitedListToStringArray(value);
            List<T> strategies = new ArrayList(classNames.length);
            String[] var7 = classNames;
            int var8 = classNames.length;

            for(int var9 = 0; var9 < var8; ++var9) {
                String className = var7[var9];

                try {
                    Class<?> clazz = ClassUtils.forName(className, DispatcherServlet.class.getClassLoader());
                    Object strategy = this.createDefaultStrategy(context, clazz);
                    strategies.add(strategy);
                } catch (ClassNotFoundException var13) {
                    throw new BeanInitializationException("Could not find DispatcherServlet's default strategy class [" + className + "] for interface [" + key + "]", var13);
                } catch (LinkageError var14) {
                    throw new BeanInitializationException("Unresolvable class definition for DispatcherServlet's default strategy class [" + className + "] for interface [" + key + "]", var14);
                }
            }

            return strategies;
        }
    }

    protected Object createDefaultStrategy(ApplicationContext context, Class<?> clazz) {
        return context.getAutowireCapableBeanFactory().createBean(clazz);
    }

    protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
        this.logRequest(request);
        Map<String, Object> attributesSnapshot = null;
        if (WebUtils.isIncludeRequest(request)) {
            attributesSnapshot = new HashMap();
            Enumeration attrNames = request.getAttributeNames();

            label95:
            while(true) {
                String attrName;
                do {
                    if (!attrNames.hasMoreElements()) {
                        break label95;
                    }

                    attrName = (String)attrNames.nextElement();
                } while(!this.cleanupAfterInclude && !attrName.startsWith("org.springframework.web.servlet"));

                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }

        request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.getWebApplicationContext());
        request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
        request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
        request.setAttribute(THEME_SOURCE_ATTRIBUTE, this.getThemeSource());
        if (this.flashMapManager != null) {
            FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
            if (inputFlashMap != null) {
                request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
            }

            request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
            request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
        }

        try {
            this.doDispatch(request, response);
        } finally {
            if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted() && attributesSnapshot != null) {
                this.restoreAttributesAfterInclude(request, attributesSnapshot);
            }

        }

    }

    private void logRequest(HttpServletRequest request) {
        LogFormatUtils.traceDebug(this.logger, (traceOn) -> {
            String params;
            if (this.isEnableLoggingRequestDetails()) {
                params = (String)request.getParameterMap().entrySet().stream().map((entry) -> {
                    return (String)entry.getKey() + ":" + Arrays.toString((Object[])entry.getValue());
                }).collect(Collectors.joining(", "));
            } else {
                params = request.getParameterMap().isEmpty() ? "" : "masked";
            }

            String queryString = request.getQueryString();
            String queryClause = StringUtils.hasLength(queryString) ? "?" + queryString : "";
            String dispatchType = !request.getDispatcherType().equals(DispatcherType.REQUEST) ? "\"" + request.getDispatcherType().name() + "\" dispatch for " : "";
            String message = dispatchType + request.getMethod() + " \"" + getRequestUri(request) + queryClause + "\", parameters={" + params + "}";
            if (traceOn) {
                List<String> values = Collections.list(request.getHeaderNames());
                String headers = values.size() > 0 ? "masked" : "";
                if (this.isEnableLoggingRequestDetails()) {
                    headers = (String)values.stream().map((name) -> {
                        return name + ":" + Collections.list(request.getHeaders(name));
                    }).collect(Collectors.joining(", "));
                }

                return message + ", headers={" + headers + "} in DispatcherServlet '" + this.getServletName() + "'";
            } else {
                return message;
            }
        });
    }

    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Object dispatchException = null;

                try {
                    processedRequest = this.checkMultipart(request);
                    multipartRequestParsed = processedRequest != request;
                    mappedHandler = this.getHandler(processedRequest);
                    if (mappedHandler == null) {
                        this.noHandlerFound(processedRequest, response);
                        return;
                    }

                    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                    String method = request.getMethod();
                    boolean isGet = "GET".equals(method);
                    if (isGet || "HEAD".equals(method)) {
                        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                        if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                            return;
                        }
                    }

                    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }

                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }

                    this.applyDefaultViewName(processedRequest, mv);
                    mappedHandler.applyPostHandle(processedRequest, response, mv);
                } catch (Exception var20) {
                    dispatchException = var20;
                } catch (Throwable var21) {
                    dispatchException = new NestedServletException("Handler dispatch failed", var21);
                }

                this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
            } catch (Exception var22) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
            } catch (Throwable var23) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
            }

        } finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            } else if (multipartRequestParsed) {
                this.cleanupMultipart(processedRequest);
            }

        }
    }

    private void applyDefaultViewName(HttpServletRequest request, @Nullable ModelAndView mv) throws Exception {
        if (mv != null && !mv.hasView()) {
            String defaultViewName = this.getDefaultViewName(request);
            if (defaultViewName != null) {
                mv.setViewName(defaultViewName);
            }
        }

    }

    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv, @Nullable Exception exception) throws Exception {
        boolean errorView = false;
        if (exception != null) {
            if (exception instanceof ModelAndViewDefiningException) {
                this.logger.debug("ModelAndViewDefiningException encountered", exception);
                mv = ((ModelAndViewDefiningException)exception).getModelAndView();
            } else {
                Object handler = mappedHandler != null ? mappedHandler.getHandler() : null;
                mv = this.processHandlerException(request, response, handler, exception);
                errorView = mv != null;
            }
        }

        if (mv != null && !mv.wasCleared()) {
            this.render(mv, request, response);
            if (errorView) {
                WebUtils.clearErrorRequestAttributes(request);
            }
        } else if (this.logger.isTraceEnabled()) {
            this.logger.trace("No view rendering, null ModelAndView returned.");
        }

        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
                mappedHandler.triggerAfterCompletion(request, response, (Exception)null);
            }

        }
    }

    protected LocaleContext buildLocaleContext(HttpServletRequest request) {
        LocaleResolver lr = this.localeResolver;
        return lr instanceof LocaleContextResolver ? ((LocaleContextResolver)lr).resolveLocaleContext(request) : () -> {
            return lr != null ? lr.resolveLocale(request) : request.getLocale();
        };
    }

    protected HttpServletRequest checkMultipart(HttpServletRequest request) throws MultipartException {
        if (this.multipartResolver != null && this.multipartResolver.isMultipart(request)) {
            if (WebUtils.getNativeRequest(request, MultipartHttpServletRequest.class) != null) {
                if (request.getDispatcherType().equals(DispatcherType.REQUEST)) {
                    this.logger.trace("Request already resolved to MultipartHttpServletRequest, e.g. by MultipartFilter");
                }
            } else if (this.hasMultipartException(request)) {
                this.logger.debug("Multipart resolution previously failed for current request - skipping re-resolution for undisturbed error rendering");
            } else {
                try {
                    return this.multipartResolver.resolveMultipart(request);
                } catch (MultipartException var3) {
                    if (request.getAttribute("javax.servlet.error.exception") == null) {
                        throw var3;
                    }
                }

                this.logger.debug("Multipart resolution failed for error dispatch", var3);
            }
        }

        return request;
    }

    private boolean hasMultipartException(HttpServletRequest request) {
        for(Throwable error = (Throwable)request.getAttribute("javax.servlet.error.exception"); error != null; error = error.getCause()) {
            if (error instanceof MultipartException) {
                return true;
            }
        }

        return false;
    }

    protected void cleanupMultipart(HttpServletRequest request) {
        if (this.multipartResolver != null) {
            MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest)WebUtils.getNativeRequest(request, MultipartHttpServletRequest.class);
            if (multipartRequest != null) {
                this.multipartResolver.cleanupMultipart(multipartRequest);
            }
        }

    }

    @Nullable
    protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        if (this.handlerMappings != null) {
            Iterator var2 = this.handlerMappings.iterator();

            while(var2.hasNext()) {
                HandlerMapping mapping = (HandlerMapping)var2.next();
                HandlerExecutionChain handler = mapping.getHandler(request);
                if (handler != null) {
                    return handler;
                }
            }
        }

        return null;
    }

    protected void noHandlerFound(HttpServletRequest request, HttpServletResponse response) throws Exception {
        if (pageNotFoundLogger.isWarnEnabled()) {
            pageNotFoundLogger.warn("No mapping for " + request.getMethod() + " " + getRequestUri(request));
        }

        if (this.throwExceptionIfNoHandlerFound) {
            throw new NoHandlerFoundException(request.getMethod(), getRequestUri(request), (new ServletServerHttpRequest(request)).getHeaders());
        } else {
            response.sendError(404);
        }
    }

    protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
        if (this.handlerAdapters != null) {
            Iterator var2 = this.handlerAdapters.iterator();

            while(var2.hasNext()) {
                HandlerAdapter adapter = (HandlerAdapter)var2.next();
                if (adapter.supports(handler)) {
                    return adapter;
                }
            }
        }

        throw new ServletException("No adapter for handler [" + handler + "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
    }

    @Nullable
    protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) throws Exception {
        request.removeAttribute(HandlerMapping.PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE);
        ModelAndView exMv = null;
        if (this.handlerExceptionResolvers != null) {
            Iterator var6 = this.handlerExceptionResolvers.iterator();

            while(var6.hasNext()) {
                HandlerExceptionResolver resolver = (HandlerExceptionResolver)var6.next();
                exMv = resolver.resolveException(request, response, handler, ex);
                if (exMv != null) {
                    break;
                }
            }
        }

        if (exMv != null) {
            if (exMv.isEmpty()) {
                request.setAttribute(EXCEPTION_ATTRIBUTE, ex);
                return null;
            } else {
                if (!exMv.hasView()) {
                    String defaultViewName = this.getDefaultViewName(request);
                    if (defaultViewName != null) {
                        exMv.setViewName(defaultViewName);
                    }
                }

                if (this.logger.isTraceEnabled()) {
                    this.logger.trace("Using resolved error view: " + exMv, ex);
                } else if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Using resolved error view: " + exMv);
                }

                WebUtils.exposeErrorRequestAttributes(request, ex, this.getServletName());
                return exMv;
            }
        } else {
            throw ex;
        }
    }

    protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
        Locale locale = this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale();
        response.setLocale(locale);
        String viewName = mv.getViewName();
        View view;
        if (viewName != null) {
            view = this.resolveViewName(viewName, mv.getModelInternal(), locale, request);
            if (view == null) {
                throw new ServletException("Could not resolve view with name '" + mv.getViewName() + "' in servlet with name '" + this.getServletName() + "'");
            }
        } else {
            view = mv.getView();
            if (view == null) {
                throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a View object in servlet with name '" + this.getServletName() + "'");
            }
        }

        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Rendering view [" + view + "] ");
        }

        try {
            if (mv.getStatus() != null) {
                response.setStatus(mv.getStatus().value());
            }

            view.render(mv.getModelInternal(), request, response);
        } catch (Exception var8) {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Error rendering view [" + view + "]", var8);
            }

            throw var8;
        }
    }

    @Nullable
    protected String getDefaultViewName(HttpServletRequest request) throws Exception {
        return this.viewNameTranslator != null ? this.viewNameTranslator.getViewName(request) : null;
    }

    @Nullable
    protected View resolveViewName(String viewName, @Nullable Map<String, Object> model, Locale locale, HttpServletRequest request) throws Exception {
        if (this.viewResolvers != null) {
            Iterator var5 = this.viewResolvers.iterator();

            while(var5.hasNext()) {
                ViewResolver viewResolver = (ViewResolver)var5.next();
                View view = viewResolver.resolveViewName(viewName, locale);
                if (view != null) {
                    return view;
                }
            }
        }

        return null;
    }

    private void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable HandlerExecutionChain mappedHandler, Exception ex) throws Exception {
        if (mappedHandler != null) {
            mappedHandler.triggerAfterCompletion(request, response, ex);
        }

        throw ex;
    }

    private void restoreAttributesAfterInclude(HttpServletRequest request, Map<?, ?> attributesSnapshot) {
        Set<String> attrsToCheck = new HashSet();
        Enumeration attrNames = request.getAttributeNames();

        while(true) {
            String attrName;
            do {
                if (!attrNames.hasMoreElements()) {
                    attrsToCheck.addAll(attributesSnapshot.keySet());
                    Iterator var8 = attrsToCheck.iterator();

                    while(var8.hasNext()) {
                        String attrName = (String)var8.next();
                        Object attrValue = attributesSnapshot.get(attrName);
                        if (attrValue == null) {
                            request.removeAttribute(attrName);
                        } else if (attrValue != request.getAttribute(attrName)) {
                            request.setAttribute(attrName, attrValue);
                        }
                    }

                    return;
                }

                attrName = (String)attrNames.nextElement();
            } while(!this.cleanupAfterInclude && !attrName.startsWith("org.springframework.web.servlet"));

            attrsToCheck.add(attrName);
        }
    }

    private static String getRequestUri(HttpServletRequest request) {
        String uri = (String)request.getAttribute("javax.servlet.include.request_uri");
        if (uri == null) {
            uri = request.getRequestURI();
        }

        return uri;
    }

    static {
        try {
            ClassPathResource resource = new ClassPathResource("DispatcherServlet.properties", DispatcherServlet.class);
            defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
        } catch (IOException var1) {
            throw new IllegalStateException("Could not load 'DispatcherServlet.properties': " + var1.getMessage());
        }
    }
}
初始化最后的刷新要完成以下功能：
        this.initMultipartResolver(context);
        this.initLocaleResolver(context);
        this.initThemeResolver(context);
        this.initHandlerMappings(context);
        this.initHandlerAdapters(context);
        this.initHandlerExceptionResolvers(context);
        this.initRequestToViewNameTranslator(context);
        this.initViewResolvers(context);
        this.initFlashMapManager(context);

initHandlerMapping
Map<String, HandlerMapping> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
matchingBeans得到了String和HandlerMapping的键值对
每个handlerMapping里有一个urlMap

initHandlerAdapter
一共四个：
  HttpRequestHandlerAdapter：简单地将Http请求对象和相应传递给Http请求处理器，不需要返回值
  SimpleControllerHandlerAdapter：将Http请求适配到一个控制器的实现进行处理，这里控制器的实现是一个简单的控制器接口的实现，客户化的业务逻辑通常是在控制器接口的实现类中实现的
  AnnotationMethodHandlerAdapter：基于注解的实现，通过解析声明在注解控制器的请求映射信息来解析相应的处理器方法来处理当前的HTTP请求，通过反射来发现探测处理器方法的参数，调用处理器方法，并且映射返回值到魔性和控制器对象
  HandleFunctionAdapter
  
initHandlerExceptionResolvers
  基于HandlerExceptionResolver接口的异常处理，使用这种方式只需要实现resolveException方法，该方法返回一个ModelAndView对象，在方法内部对异常的类型进行判断，然后生成对应ModelAndView对象

initRequestToViewNameTranslator();
  当Controller处理器方法没有返回一个View对象或逻辑视名称，并且在该方法中没有直接网response的输出流里面写数据的时候，spring就会约定提供一个逻辑视图平成
  
initViewResolvers
  controller将请求处理结果放入到ModelAndView中以后，DispatcherServlet会根据ModelAndView选择合适的view进行渲染
  
dispatche过程：
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Object dispatchException = null;

                try {
                    //检验是否是文件请求
                    processedRequest = this.checkMultipart(request);
                    multipartRequestParsed = processedRequest != request;
                    //这里的mappedHandler是一个HandlerExecutionChain
                    mappedHandler = this.getHandler(processedRequest);
                    if (mappedHandler == null) {
                        this.noHandlerFound(processedRequest, response);
                        return;
                    }

                    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                    String method = request.getMethod();
                    boolean isGet = "GET".equals(method);
                    if (isGet || "HEAD".equals(method)) {
                        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                        if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                            return;
                        }
                    }

                    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }

                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }

                    this.applyDefaultViewName(processedRequest, mv);
                    mappedHandler.applyPostHandle(processedRequest, response, mv);
                } catch (Exception var20) {
                    dispatchException = var20;
                } catch (Throwable var21) {
                    dispatchException = new NestedServletException("Handler dispatch failed", var21);
                }

                this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
            } catch (Exception var22) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
            } catch (Throwable var23) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
            }

        } finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            } else if (multipartRequestParsed) {
                this.cleanupMultipart(processedRequest);
            }

        }
    }
 //根据请求获取HandlerExecutionChain  HandlerExcutionChain中封装了controller中的方法和所有的拦截器，然后根据Method获取HandlerAdapter（HandlerAdapter是否支持该Method）
 //然后HandlerAdaper执行handler，回到了这个方法 protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod)
 //这个方法内部使用了invoke
 如果返回数据没有视图，modelAndView 为null，数据在response中
 有视图，modelAndView getView
 viewResolver拿到视图
 protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        if (this.handlerMappings != null) {
            Iterator var2 = this.handlerMappings.iterator();
            //便利HandlerMapping 列表，直到能找到某个HandlerMapping里有该Request的url映射
            while(var2.hasNext()) {
                HandlerMapping mapping = (HandlerMapping)var2.next();
                HandlerExecutionChain handler = mapping.getHandler(request);
                if (handler != null) {
                    return handler;
                }
            }
        }

        return null;
    }
  
      
