---
title: webx framework 源码解析
date: 2017-06-24 20:23:49
tags:
---

### webx framework(基于servlet的web框架)
- 主要作用
    - 初始化spring容器
    - 初始化日志系统
    - 增强request、response、session功能
    - 提供pipeline流程处理机制
    - 异常处理

<!-- more -->

- webx应用启动过程
    - 启动时容器（tomcat jetty等）会去读web.xml，按照里面的配置去初始化listener，filter等
        - web.xml加载过程
            1. 读取`<listener></listener>`和`<context-param></context-param>`
            2. 创建一个ServletContext（application）
            3. 将`<context-param></context-param>`的name和value键值对存入ServletContext
            4. 创建`<listener></listener>`中的类实例
            5. 创建`<filter></filter>`中的过滤器
            6. 当发生第一次请求时实例化`<servlet></servlet>`,一般不被容器销毁
        - web.xml 

        ```
        <!-- 初始化日志系统 -->
        <listener>
            <listener-class>com.alibaba.citrus.logconfig.LogConfiguratorListener</listener-class>
        </listener>

        <!-- 装载/WEB-INF/webx.xml, /WEB-INF/webx-*.xml -->
        <listener>
            <listener-class>com.alibaba.citrus.webx.context.WebxContextLoaderListener</listener-class>
        </listener>
        ```

    - 初始化context过程
        - 容器首先调用由web.xml中指定的com.alibaba.citrus.webx.context.WebxContextLoaderListener(继承了spring的ContextLoaderListener，所以可以和spring通用)的contextInitialized方法，这个方法里面调用了createContextLoader方法（WebxContextLoaderListener覆盖了此方法），创建一个WebxComponentsLoader实例（所有组件共享)，并且调用了WebxComponentsLoader的initWebApplicationContext方法
        - ContextLoaderListener.java

        ```
        public class ContextLoaderListener extends ContextLoader implements ServletContextListener {

            public void contextInitialized(ServletContextEvent event) {
                this.contextLoader = this.createContextLoader();
                if(this.contextLoader == null) {
                    this.contextLoader = this;
                }

                this.contextLoader.initWebApplicationContext(event.getServletContext());
        }
        ```

        - WebxContextLoaderListener.java

        ```
        public class WebxContextLoaderListener extends ContextLoaderListener {
        @Override
            protected final ContextLoader createContextLoader() {
                return new WebxComponentsLoader() {

                    @Override
                    protected Class<? extends WebxComponentsContext> getDefaultContextClass() {
                        Class<? extends WebxComponentsContext> defaultContextClass = WebxContextLoaderListener.this
                            .getDefaultContextClass();

                        if (defaultContextClass == null) {
                            defaultContextClass = super.getDefaultContextClass();
                        }

                        return defaultContextClass;
                    }
                };
            }
        ```

        - WebxComponentsLoader的initWebApplicationContext方法中保存servletContext对象，再调用了init方法和父类的initWebApplicationContext方法,父类中的initWebApplicationContext方法会调用createWebApplicationContext()，determineContextClass()拿到WebxComponentsContext的class，再调BeanUtils.instantiateClass方法创建了WebxComponentsContext对象(在spring中原有的是xmlApplicationContext，地位一样)

        - WebxComponentsLoader.java

        ```
        @Override
        public WebApplicationContext initWebApplicationContext(ServletContext servletContext) throws IllegalStateException,BeansException {
            this.servletContext = servletContext;
            init();

            return super.initWebApplicationContext(servletContext);
        }
        protected void init() {
            setWebxConfigurationName(servletContext.getInitParameter("webxConfigurationName"));
        }
        ```

        - ContextLoader.java

        ```
        protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
            Class<?> contextClass = this.determineContextClass(sc);
            if(!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
                throw new ApplicationContextException("Custom context class [" + contextClass.getName() + "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
            } else {
                return (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
            }
        }
        ```

        - 在initWebApplicationContext方法中获得WebxComponentsContext对象，具体调用configureAndRefreshWebApplicationContext()->refresh(),创建加载Spring容器配置

        - AbstractApplicationContext.java

        ```
        public void refresh() throws BeansException, IllegalStateException {
            Object var1 = this.startupShutdownMonitor;
            synchronized(this.startupShutdownMonitor) {
                this.prepareRefresh();
                ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
                this.prepareBeanFactory(beanFactory);

                try {
                    this.postProcessBeanFactory(beanFactory);
                    this.invokeBeanFactoryPostProcessors(beanFactory);
                    this.registerBeanPostProcessors(beanFactory);
                    this.initMessageSource();
                    this.initApplicationEventMulticaster();
                    this.onRefresh();
                    this.registerListeners();
                    this.finishBeanFactoryInitialization(beanFactory);
                    this.finishRefresh();
                } catch (BeansException var5) {
                    this.destroyBeans();
                    this.cancelRefresh(var5);
                    throw var5;
                }
            }   
        }
        ```
        - WebxComponentsLoader覆盖了postProcessBeanFactory方法，实例化WebxComponentsCreator，并且调用了createComponents方法创建容器，父容器（root component，通过webx.xml建立）,子容器（sub component，通过webx-*.xml建立）创建子容器时，将子容器的父亲设置为了父容器，形成层级结构
        ```
        /** 初始化components。 */
        private WebxComponentsImpl createComponents(WebxConfiguration parentConfiguration,
                                                    ConfigurableListableBeanFactory beanFactory) {
            ComponentsConfig componentsConfig = getComponentsConfig(parentConfiguration);
    
            // 假如isAutoDiscoverComponents==true，试图自动发现components
            Map<String, String> componentNamesAndLocations = findComponents(componentsConfig, getServletContext());
    
            // 取得特别指定的components
            Map<String, ComponentConfig> specifiedComponents = componentsConfig.getComponents();
    
            // 实际要初始化的comonents，为上述两种来源的并集
            Set<String> componentNames = createTreeSet();
    
            componentNames.addAll(componentNamesAndLocations.keySet());
            componentNames.addAll(specifiedComponents.keySet());
    
            // 创建root controller
            WebxRootController rootController = componentsConfig.getRootController();
    
            if (rootController == null) {
                rootController = (WebxRootController) BeanUtils.instantiateClass(componentsConfig.getRootControllerClass());
            }
    
            // 创建并将components对象置入resolvable dependencies，以便注入到需要的bean中
            WebxComponentsImpl components = new WebxComponentsImpl(componentsContext,
                                                                   componentsConfig.getDefaultComponent(), rootController, parentConfiguration);
    
            beanFactory.registerResolvableDependency(WebxComponents.class, components);
    
            // 初始化每个component
            for (String componentName : componentNames) {
                ComponentConfig componentConfig = specifiedComponents.get(componentName);
    
                String componentPath = null;
                WebxController controller = null;
    
                if (componentConfig != null) {
                    componentPath = componentConfig.getPath();
                    controller = componentConfig.getController();
                }
    
                if (controller == null) {
                    controller = (WebxController) BeanUtils.instantiateClass(componentsConfig.getDefaultControllerClass());
                }
    
                WebxComponentImpl component = new WebxComponentImpl(components, componentName, componentPath,
                                                                    componentName.equals(componentsConfig.getDefaultComponent()), controller,
                                                                    getWebxConfigurationName());
    
                components.addComponent(component);
    
                prepareComponent(component, componentNamesAndLocations.get(componentName));
            }
    
            return components;
        }
        ```

        - 其他加载过程和spring一样，不同的是WebxApplicationContext指定寻找的配置项为/WEB-INF/webx-*.xml或者/WEB-INF/webx.xml
        - WebxApplicationContext.java

        ```
        public class WebxApplicationContext extends             ResourceLoadingXmlWebApplicationContext {
        public WebxApplicationContext() {}

            protected String[] getDefaultConfigLocations() {
                return this.getNamespace() != null?new String[]{"/WEB-INF/webx-*.xml".replace("*", this.getNamespace())}:new String[]{"/WEB-INF/webx.xml"};
            }
        }
        ```

        - 另外还重写了finishRefresh方法，执行各个模块的容器初始化工作,解析配置文件中的bean,这样根容器和所有的子容器都初始化完成了
        - WebxComponentsContext.java

        ```
        @Override
        protected void finishRefresh() {
            super.finishRefresh();
            getLoader().finishRefresh();
        }
        ```

        - WebxComponentsLoader.java
        
        ```
        /** 初始化所有components。 */
        public void finishRefresh() {
            components.getWebxRootController().onFinishedProcessContext();

            for (WebxComponent component : components) {
                logInBothServletAndLoggingSystem("Initializing Spring sub   WebApplicationContext: " + component.getName());

                WebxComponentContext wcc = (WebxComponentContext)   component.getApplicationContext();
                WebxController controller = component.getWebxController();

                wcc.refresh();
                controller.onFinishedProcessContext();
            }

            logInBothServletAndLoggingSystem("WebxComponents: initialization    completed");
        }
        ```


- webx响应请求的过程
    1. 因为webxframework基于servletapi，所有当一个请求来时，会被web.xml中配置的filter拦截.这边配置了两个filter，按顺序一个请求先经过mdc（将这次请求的信息打印到日志），再经过webx处理
    ```
    <filter>
        <filter-name>mdc</filter-name>
        <filter-class>com.alibaba.citrus.webx.servlet.SetLoggingContextFilter</filter-class>
    </filter>

    <filter>
        <filter-name>webx</filter-name>
        <filter-class>com.alibaba.citrus.webx.servlet.WebxFrameworkFilter</filter-class>
        <init-param>
            <param-name>excludes</param-name>
            <param-value><!-- 需要被“排除”的URL路径，以逗号分隔，如/static, *.jpg。适合于映射静态页面、图片。 --></param-value>
        </init-param>
        <init-param>
            <param-name>passthru</param-name>
            <param-value><!-- 需要被“略过”的URL路径，以逗号分隔，如/myservlet, *.jsp。适用于映射servlet、filter。
                对于passthru请求，webx的request-contexts服务、错误处理、开发模式等服务仍然可用。 --></param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>mdc</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>webx</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ```

2. webx的class为WebxFrameworkFilter，当有请求来时，调用doFilter方法,首先判断是不是需要排除的path，是的话则转交给servlet处理，如果不是的话，调用root容器的service方法处理
    ```
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        String path = getResourcePath(request);

        if (isExcluded(path)) {
            log.debug("Excluded request: {}", path);
            chain.doFilter(request, response);
            return;
        } else {
            log.debug("Accepted and started to process request: {}", path);
        }

        try {
            getWebxComponents().getWebxRootController().service(request, response, chain);
        } catch (IOException e) {
            throw e;
        }
    ```

3. service方法中判断了url是不是属于passthru，如果不是的话调用handlerequest方法处理，如果是的话，则调用giveupControl放弃处理转交给servlet处理
    ```
    public final void service(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws Exception {
        RequestContext requestContext = null;

        try {
            requestContext = assertNotNull(getRequestContext(request, response), "could not get requestContext");

            // 如果请求已经结束，则不执行进一步的处理。例如，当requestContext已经被重定向了，则立即结束请求的处理。
            if (isRequestFinished(requestContext)) {
                return;
            }

            // 请求未结束，则继续处理...
            request = requestContext.getRequest();
            response = requestContext.getResponse();

            // 如果是一个内部请求，则执行内部请求
            if (handleInternalRequest(request, response)) {
                return;
            }

            // 如果不是内部的请求，并且没有被passthru，则执行handleRequest
            if (isRequestPassedThru(request) || !handleRequest(requestContext)) {
                // 如果请求被passthru，或者handleRequest返回false（即pipeline放弃请求），
                // 则调用filter chain，将控制交还给servlet engine。
                giveUpControl(requestContext, chain);
            }
    ```

4. WebxRootControllerImpl中的handleRequest方法，决定了这个请求由哪个子component处理，如果没有匹配到的则用默认defaultcomponent处理
    ```
    public class WebxRootControllerImpl extends AbstractWebxRootController {
    @Override
    protected boolean handleRequest(RequestContext requestContext) throws Exception {
        HttpServletRequest request = requestContext.getRequest();

        // Servlet mapping有两种匹配方式：前缀匹配和后缀匹配。
        // 对于前缀匹配，例如：/servlet/aaa/bbb，servlet path为/servlet，path info为/aaa/bbb
        // 对于前缀匹配，当mapping pattern为/*时，/aaa/bbb，servlet path为""，path info为/aaa/bbb
        // 对于后缀匹配，例如：/aaa/bbb.html，servlet path为/aaa/bbb.html，path info为null
        //
        // 对于前缀匹配，取其pathInfo；对于后缀匹配，取其servletPath。
        String path = ServletUtil.getResourcePath(request);

        // 再根据path查找component
        WebxComponent component = getComponents().findMatchedComponent(path);
        boolean served = false;

        if (component != null) {
            try {
                WebxUtil.setCurrentComponent(request, component);
                served = component.getWebxController().service(requestContext);
            } finally {
                WebxUtil.setCurrentComponent(request, null);
            }
        }

        return served;
    }
   }
    ```

  - WebxComponentsLoader.java
    ```
    public WebxComponent findMatchedComponent(String path) {
            if (!path.startsWith("/")) {
                path = "/" + path;
            }

            WebxComponent defaultComponent = getDefaultComponent();
            WebxComponent matched = null;

            // 前缀匹配componentPath。
            for (WebxComponent component : this) {
                if (component == defaultComponent) {
                    continue;
                }

                String componentPath = component.getComponentPath();

                if (!path.startsWith(componentPath)) {
                    continue;
                }

                // path刚好等于componentPath，或者path以componentPath/为前缀
                if (path.length() == componentPath.length() || path.charAt(componentPath.length()) == '/') {
                    matched = component;
                    break;
                }
            }

            // fallback to default component
            if (matched == null) {
                matched = defaultComponent;
            }

            return matched;
        }
    ```

  5. 找到处理的component后，调用component的controller来处理请求，调用pipeline来处理,按照webx-app1.xml导入的pipeline.xml中定义的pipeline顺序来处理
    - WebxContrillerImpl.java
    ```
    public boolean service(RequestContext requestContext) throws Exception {
        PipelineInvocationHandle handle = pipeline.newInvocation();

        handle.invoke();

        // 假如pipeline被中断，则视作请求未被处理。filter将转入chain中继续处理请求。
        return !handle.isBroken();
    }
    ```
  - pipeline.xml
    ```
    <services:pipeline>
        <!-- 初始化turbine rundata，并在pipelineContext中设置可能会用到的对象(如rundata、utils)，以便valve取得。 -->
        <pl-valves:prepareForTurbine />

        <!-- 分析URL，取得target。 -->
        <pl-valves:analyzeURL />
        ...

        <pl-valves:loop>
            <pl-valves:choose>
                <when>
                    <!-- 执行带模板的screen，默认有layout。 -->
                    <pl-conditions:target-extension-condition extension="null" />
                    <pl-valves:performAction />
                    <pl-valves:performTemplateScreen />
                    <pl-valves:renderTemplate />
                </when>
               
               ...

        </pl-valves:loop>
    </services:pipeline>
    ```

6. 在analyzeURL这个valve中，首先将url转成驼峰格式，去掉下划线，并设置action和actionEvent
    ```
    public void invoke(PipelineContext pipelineContext) throws Exception {
        TurbineRunDataInternal rundata = (TurbineRunDataInternal) getTurbineRunData(request);
        String target = null;

        // 取得target，并转换成统一的内部后缀名。
        String pathInfo = ServletUtil.getResourcePath(rundata.getRequest()).substring(
                component.getComponentPath().length());

        if ("/".equals(pathInfo)) {
            pathInfo = getHomepage();
        }

        // 注意，必须将pathInfo转换成camelCase。
        int lastSlashIndex = pathInfo.lastIndexOf("/");

        if (lastSlashIndex >= 0) {
            pathInfo = pathInfo.substring(0, lastSlashIndex) + "/"
                       + StringUtil.toCamelCase(pathInfo.substring(lastSlashIndex + 1));
        } else {
            pathInfo = StringUtil.toCamelCase(pathInfo);
        }

        target = mappingRuleService.getMappedName(EXTENSION_INPUT, pathInfo);

        rundata.setTarget(target);

        // 取得action
        String action = StringUtil.toCamelCase(trimToNull(rundata.getParameters().getString(actionParam)));

        action = mappingRuleService.getMappedName(ACTION_MODULE, action);
        rundata.setAction(action);

        // 取得actionEvent
        String actionEvent = ActionEventUtil.getEventName(rundata.getRequest());
        rundata.setActionEvent(actionEvent);

        pipelineContext.invokeNext();
    }
    ```

7. performAction根据action在action文件夹下找到对应的action类,performScreen根据target在screen文件夹找到对应的screen类，执行execute方法
    ```
    <when>
        <!-- 执行不带模板的screen，无layout。 -->
        <pl-conditions:target-extension-condition extension="do" />
        <pl-valves:performAction />
        <pl-valves:performScreen />
    </when>
    ```
    
- webx响应请求时的默认约定
    1. 根据url寻找对应的component组件处理时（WebxComponentsLoad.WebxComponentsImpl.findMatchedComponent()）
        - 遍历所有的组件，若url path等于component path或者url path以component path/开头，则找到组件
        - 遍历完都没有找到的话，返回默认组件处理（webx.xml中配置）
    2. pipeline中分析URL时，设置请求的target和action。target在以后寻找template，screen，action时需要用到（AnalyzeURLValve.invoke()）
        - 取出url中除了component path的剩余部分。这边需要注意，如果当前是默认component处理的话，则component path为空，取出整个url。其他component，component path为component的name
        - 如果取出来的path为空，则赋值为homepage（pipeline中设置）
        - 将path的最后一段'/'内容去掉下划线转成驼峰格式
        - 将path根据mapping rule（id=extension.input）设置target（mapping rule在webx-component-and-root.xml中设置）
        - 取出request中的action，根据mapping rule（id=action）设置action
    3. performAction寻找action（performActionValve.invoke()）
        - 将action根据mapping rule（id=action，direct-module-rule同名寻找），在moduleLoadService（webx-xxx.xml中配置）中配置的包地址中寻找同名的action，并调用execute()
    4. performScreen寻找screen（performScreenValve.invoke()）
        - 将target根据mapping rule（id=screen.notemplate，direct-module-rule同名寻找），在moduleLoadService（webx-xxx.xml中配置）中配置的包地址中寻找同名的screen，并调用execute()
    5. performTemplateScreen寻找带模板的screen（PerformTemplateScreenValve.invoke()）
        - 将target根据mapping rule（id=screen，fallback-module-rule向上寻找），在moduleLoadService（webx-xxx.xml中配置）中配置的包地址中寻找screen，并调用execute()
        - fallback-module-rule寻找规则，向上匹配寻找，对于xxx/yyy/zzz的path，依次匹配的路径为xxx/yyy/zzz，xxx/yyy/default,xxx/default,default
    6. renderTemplate渲染模板（RenderTemplateValve.invoke()）
        - 将target根据mapping rule（id=screen.template，direct-template-rule同名寻找），在template/screen文件夹下寻找同名的模板文件，并渲染
        - 将target根据mapping rule（id=layout.template，fallback-template-rule向上寻找），在template/layout文件夹下向上匹配寻找模板文件，并渲染
    

 
