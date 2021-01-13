基于SSH框架实现的鲜花订购系统
=


程序有问题联系[程序帮](http://ll032.cn/HZ6vHa)：QQ1022287044


项目介绍
----
> 1. 本系统使用Struts2+Spring+Hibernate架构，数据库使用MySQL,连接池使用c3p0。
> 2. 模仿花礼网进行前端设计与开发，实现网站导航、商品分类展示，商品详情、商品检索、购物车等功能。
> 3. 使用EasyUI实现后台对商品分类、商品信息、用户信息、订单信息的管理，包括增删改查，文件上传等。

项目适用人群
----
正在做毕设的学生，或者需要项目实战练习的Java学习者

开发环境
-----
1. jdk 8
2. intellij idea
3. tomcat 8.5.40
4. mysql 5.7

所用技术
-----
1. Struts2+Spring+Hibernate
2. js+ajax
3. easyUI

 项目架构
----

![](/src/image/项目架构.png)
 
项目截图
----

- 注册

![](/src/image/注册.png)

- 首页

![](/src/image/首页.png)


- 商品详情 

![](/src/image/商品详情页.png)


- 购物车

![](/src/image/购物车.png)


- 管理端-类别管理

![](/src/image/管理端-类别管理.png)


- 管理端-商品管理

![](/src/image/管理端-商品管理.png)


- 管理端-订单管理

![](/src/image/管理端-订单管理.png)



数据库配置
----
```
<!-- c3p0 数据源 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
    destroy-method="close">
    <property name="driverClass" value="com.mysql.jdbc.Driver" />
    <property name="jdbcUrl"
        value="jdbc:mysql://localhost:3306/db_flower?useUnicode=true&amp;characterEncoding=utf8" />
    <property name="user" value="root" />
    <property name="password" value="root123" />
    <!--初始化时获取的连接数，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
    <property name="initialPoolSize" value="1" />
    <!--连接池中保留的最小连接数。 -->
    <property name="minPoolSize" value="1" />
    <!--连接池中保留的最大连接数。Default: 15 -->
    <property name="maxPoolSize" value="300" />
    <!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
    <property name="maxIdleTime" value="60" />
    <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
    <property name="acquireIncrement" value="5" />
    <!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
    <property name="idleConnectionTestPeriod" value="60" />
</bean>
```


关键代码
-----
1. 分页对象
```diff
public class PageModel<T> {
	// 当前页号
	private int pageNo = 1;
	// 每页记录数
	private int pageSize = 10;
	// 总记录数
	private int recordCount;
	// 总页数
	private int pageCount;
	// 存放分页数据的集合
	private List<T> datas;
}
```

2. struts.xml
```diff
<struts>
	<!--使用spring创建管理struts2的action操作 -->
	<constant name="struts.objectFactory" value="spring" />
	<!-- 设置struts2的编码为UTF8 -->
	<constant name="struts.i18n.encoding" value="UTF-8"></constant>
	<!-- 不使用浏览器缓存 -->
	<constant name="struts.serve.static.browserCache" value="false"></constant>
	<!-- 每次重新加载xml配置文件 -->
	<constant name="struts.configuration.xml.reload" value="true"></constant>
	<!-- 启用开发者模式 -->
	<constant name="struts.devMode" value="true"></constant>
	<!-- 不使用struts2提供的主题ui -->
	<constant name="struts.ui.theme" value="simple"></constant>
	<!-- 提供对通配符的支持 -->
	<constant name="strutsenableDynamicMethodInvocation" value="true" />

	<!-- 需要拦截未登录用户的包 -->
	<package name="login" namespace="/" extends="struts-default">
		<!-- 配置拦截未登录用户的拦截器 -->
		<interceptors>
			<interceptor name="userInter"
				class="com.flowershopping.tool.UserInterceptor"></interceptor>
			<interceptor-stack name="userStack">
				<interceptor-ref name="defaultStack"></interceptor-ref>
				<interceptor-ref name="userInter"></interceptor-ref>
			</interceptor-stack>
		</interceptors>
		<!-- 需要拦截的action 注销 和 提交订单 -->
		<!-- 设置默认拦截器 -->
		<default-interceptor-ref name="userStack"></default-interceptor-ref>
		<!-- 拦截结果处理 -->
		<global-results>
			<result name="login" type="redirect">/jsp/login/login.jsp</result>
		</global-results>
		<!-- 注销 -->
		<action name="logout" class="userAction" method="logout">
			<result name="success">/jsp/index/index.jsp</result>
		</action>
		<!-- 提交订单 -->
		<action name="addOrder" class="ordersAction" method="addOrder">
			<result name="success">/jsp/shopping/orderAdded.jsp</result>
		</action>
	</package>

	<!-- 需要进行未登录管理员拦截的包 -->

	<package name="admin" namespace="/" extends="struts-default">
		<!-- 配置拦截未登录管理员的拦截器 -->
		<interceptors>
			<interceptor name="adminInter"
				class="com.flowershopping.tool.AdminInterceptor"></interceptor>
			<interceptor-stack name="adminStack">
				<interceptor-ref name="defaultStack"></interceptor-ref>
				<interceptor-ref name="adminInter"></interceptor-ref>
			</interceptor-stack>
		</interceptors>
		<!-- 需要拦截的action 查看所有用户 查看订单 添加商品 -->
		<!-- 设置默认拦截器 -->
		<default-interceptor-ref name="adminStack"></default-interceptor-ref>
		<!-- 拦截结果处理 -->
		<global-results>
			<result name="login" type="redirect">/jsp/login/admin.jsp</result>
		</global-results>
		<!-- 查看所有用户 -->
		<action name="findAllUsers" class="userAction" method="findAllUsers">
			<result name="success">/jsp/admin/manageUsers.jsp</result>
		</action>
		<!-- 查看订单 -->
		<action name="findAllOrders" class="ordersAction" method="findAllOrders">
			<result name="success">/jsp/admin/manageOrders.jsp</result>
		</action>
		<!-- 添加商品 -->
		<action name="addGoods" class="goodsAction" method="addGoods">
		</action>
	</package>
	<!-- 其余包 -->
	<package name="default" namespace="/" extends="struts-default,json-default"
		strict-method-invocation="false">
		<global-results>
			<result name="jsonMap" type="json">
				<param name="root">pageMap</param>
			</result>
			<result name="stream" type="stream">
				<param name="inputName">inputStream</param>
			</result>
		</global-results>
		<!-- 商品分类 -->
		<action name="category_*" class="categoryAction" method="{1}">
			<result name="findCategories_success">/jsp/index/header.jsp</result>
		</action>
		<!-- 商品信息 -->
		<action name="goods_*" class="goodsAction" method="{1}">
			<result name="findGoodsByCategory_success">/jsp/index/contentByCategory.jsp</result>
			<result name="findAllGoods_success">/jsp/index/content.jsp</result>
			<result name="findOne_success">/jsp/shopping/product.jsp</result>
			<result name="findGoodsByKey_success">/jsp/shopping/searchResult.jsp</result>
			<result name="findGoodsByKeys_success">/jsp/shopping/searchResult.jsp</result>
		</action>
		<!-- 用户 -->
		<action name="user_*" class="userAction" method="{1}">
			<result name="checkUser_success">/jsp/index/index.jsp</result>
			<result name="checkUser_error">/jsp/login/login.jsp</result>
			<result name="checkAdmin_success">/jsp/admin/main.jsp</result>
			<result name="checkAdmin_error">/jsp/login/admin.jsp</result>
			<result name="addUser_success">/jsp/index/index.jsp</result>
			<result name="updateUser_success">/jsp/login/userinfocenter.jsp</result>
		</action>
		<!-- 订单 -->
		<action name="orders_*" class="ordersAction" method="{1}">
			<result name="addToCart_success">/jsp/shopping/showCart.jsp</result>
			<result name="myOrder">/jsp/shopping/myOrder.jsp</result>
			<result name="updateCart_error">/jsp/shopping/showCartErro.jsp</result>
            <result name="login" type="redirect">/jsp/login/login.jsp</result>
		</action>
	</package>
</struts>
```
资源下载地址：https://download.csdn.net/download/code_200/14122933
程序有问题联系[程序帮](http://ll032.cn/HZ6vHa)

项目后续
----
其他ssh，ssm，springboot版本后续迭代更新，持续关注
