javaUtility
===========

java Utility
1、在主容器中（applicationContext.xml），将Controller的注解排除掉<pre>
<context:component-scan base-package="com"> 
  <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" /> 
</context:component-scan> 

而在springMVC配置文件中将Service注解给去掉 
<context:component-scan base-package="com"> 
  <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" /> 
  <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service" /> 
  </context:component-scan> 
</pre>
因为spring的context是父子容器，所以会产生冲突，由ServletContextListener产生的是父容 器，springMVC产生的是子容器，子容器Controller进行扫描装配时装配了@Service注解的实例，而该实例理应由父容器进行初始化以 保证事务的增强处理，所以此时得到的将是原样的Service（没有经过事务加强处理，故而没有事务处理能力。 
其实就是一个加载顺序的问题
首先使用了spring MVC的项目是不需要配置action bean  而是通过spring mvc的配置文件进行扫描注解加载的
spring事务配置文件还有上下文都是通过org.springframework.web.context.ContextLoaderListener加载的，
而spring MVC的action是通过org.springframework.web.servlet.DispatcherServlet加载的 
这样就有个优先级的问题了  web是先启动ContextLoaderListener后启动DispatcherServlet
在ContextLoaderListener加载的时候action并没在容器中，所以现在使用AOP添加事务或者扫描注解都是无用的。
那么解决办法就是在DispatcherServlet 加载spring-MVC配置文件后再进行一次AOP事务扫描和注解事务扫描就OK了
2 把事务自动扫描配置放到springmvc中去<!-- enable transaction demarcation with annotations -->
  <pre>  <tx:annotation-driven transaction-manager="transactionManager"  proxy-target-class="false"  /> </pre>
