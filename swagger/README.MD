本例为:springboot集成swagger2
官方github:https://github.com/yvasiyarov/swagger


swagger4j

从版本2.0.0开始，cpj-swagger更名为swagger4j，顶级包名也更名为com.cpjit.swagger4j，与版本1.x.x 完全不兼容。
swagger4j通过与swagger ui一起快速为您的web项目产生接口文档，并且支持在线测试接口。swagger4j可以很方便的与struts2、spring mvc、servlet集成使用，下面的教程将详细说明如何使用swagger4j。

目录

一. 加入依赖JAR文件
二. 配置
三. 标注你的接口
四. 访问接口文档
五. 核心API
六. 示例程序

一. 加入依赖JAR文件

maven：
<dependency>
	<groupId>com.cpjit</groupId>
	<artifactId>swagger4j</artifactId>
	<version>2.1.0</version>
</dependency>
二. 配置

1. 选择使用方式

您可以通过三种方式来使用swagger4j。

与strut2集成
如果您的web项目是基于strut2的，您可以在您的strut2配置文件中加入如下代码来快速集成swagger4j：
   <constant name="struts.enable.DynamicMethodInvocation" value="true" />
   <constant name="struts.devMode" value="true" />

	<package name="api" namespace="/api" extends="struts-default" >
		<action name="*" class="com.cpjit.swagger4j.support.struts2.ApiAction" method="{0}">
		</action>
	</package>
与spring mvc集成 
如果您的web项目是基于spring mvc的，您可以在您的spring mvc配置文件中加入如下代码来快速集成swagger4j：
 <context:component-scan base-package="com.cpjit.swagger4j.support.springmvc">
    	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
 </context:component-scan>
与servlet集成
如果您的web项目是基于servlet的，您可以在您的web.xml配置文件中加入如下代码来快速集成swagger4j：
<servlet>
    <servlet-name>apiServlet</servlet-name>
    <servlet-class>com.cpjit.swagger4j.support.servlet.ApiServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>apiServlet</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
2. 修改配置项

您需要在项目的源文件目录下添加一个swagger.properties文件，并加入以下配置项：

packageToScan=com.cpjit.swagger4j.action
apiDescription=Swagger Demo
apiTitle=Swagger Demo
apiVersion=1.0.0
teamOfService=www.cpj.com
devMode=true
swagger4j的配置信息都必须写在swagger.properties文件里面。具体的配置项及其说明如下：

键	说明
apiFile	扫描生成json文件的存放路径
packageToScan	扫描的包
apiDescription	接口文档描述
apiTitle	接口文档标题
apiVersion	接口版本
teamOfService	服务团队
apiHost	接口主机地址
apiBasePath	接口基路径
devMode	是否启用开发模式，如果开启则每次获取接口文档的时候都会扫描类
suffix	接口请求地址后缀，如.action
disabled	是否关闭swagger4j，一般用于生产环境
三. 标注你的接口

接下来需要用swagger4j提供的注解来标明你的接口，下面以spring mvc为例来说明如何标注接口，其他方式请参考示例程序。

@Controller
@APIs("/demo")
@RequestMapping("/demo")
public class DemoController {
	@API(value="login", summary="示例1", parameters={
			@Param(name="username", description="用户名", type="string"),
			@Param(name="password", description="密码", type="string", format="password"),
			@Param(name="image" , description="图片", type="file", format="binary")
	})
	@RequestMapping(value="login", method=RequestMethod.POST)
	public void login(HttpServletResponse response, String username, String password, MultipartFile image)
												throws Exception {
		response.setContentType("application/json;charset=utf-8");
		JSONWriter writer = new JSONWriter(response.getWriter());
		Map<String, String> user = new HashMap<String, String>();
		user.put("username", username);
		user.put("password", password);
		writer.writeObject(user);
		writer.flush();
		writer.close();
	}
}
四. 访问接口文档

在完成前面的工作之后，您可以部署好您的web项目，然后在浏览器输入以下地址就可以访问接口文档了： http://127.0.0.1:8080/您的项目名/api/index

五. 核心API

1. 注解 @com.cpjit.swagger4j.annotation.APIs

该注解放在一个类上面，用来表明类中包含作为HTTP接口的方法（被注解@com.cpjit.swagger4j.annotation.API标注了的方法）。 该注解的value用来设置接口的前缀，这和struts2的命名空间很像。使用示例如下：

@APIs("/demo")
public class DemoController {
  
}
2. 注解 @com.cpjit.swagger4j.annotation.API

该注解放在一个方法上面，用来表明方法是HTTP接口方法，注解的属性说明如下：

value

与注解@com.cpjit.swagger4j.annotation.APIs的value属性一起来指定接口的地址，例如有如下设置：

@APIs("/demo")
public class DemoController {
  	@API(value="login"})
	public void login()
	}
}
那么login方法对应的接口地址为： youhost/demo/login

parameters

用来指定接口的请求参数，详情参见注解Param的说明。

summary

接口功能简述。

description

接口功能详细说明。

method

请求方式，默认是POST。

consumes

允许的请求MIME，比如：multipart/form-data、application/xml、application/json默认是application/json; charset=utf-8。 
特别说明：
当为 multipart/form-data 时，[Param](#3-注解 @comcpjitswagger4jannotationparam) 的in属性必须为formData，但是in为path、header时Param不用遵循此规则。

deprecated

1.2.2引入的新属性，表示接口是否已经被废弃，默认是false。

hide

1.2.2引入的新属性，表示是否隐藏接口。

3. 注解 @com.cpjit.swagger4j.annotation.Param

用来说明请求参数，例如：

@API(value="login", summary="示例1", parameters={
		@Param(name="username", description="用户名", type="string"),
		@Param(name="password", description="密码", type="string", format="password")})
@RequestMapping(value="login", method=RequestMethod.POST)
public void login(HttpServletResponse response, String username, String password) throws Exception {
}
这表明该接口需要两个请求参数，及username、password。 注解@com.cpjit.swagger4j.annotation.Param的属性说明如下：

name

参数名

in

输入参数类型，可取如下值：

query - 参数拼接到url中
body - 参数直接放入请求体中
path - restful风格的参数传递
header - 参数放在请求头中
formData - 参数通过form表单提交
特别说明：
当前请求方式为POST的时候，默认值为formData
请求方式为非POST的时候，默认值为query
type

数据类型, 与format一起指定请求参数的数据类型。 type 和 format 的可选值如下：

通用名	type	format	备注
integer	integer	int32	signed 32 bits
long	integer	int64	signed 64 bits
float	number	float	
double	number	double	
string	string		
byte	string	byte	base64 encoded characters
binary	string	binary	any sequence of octets
boolean	boolean		
date	string	date	As defined by full-date - RFC3339
dateTime	string	date-time	As defined by date-time - RFC3339
password	string	password	Used to hint UIs the input needs to be obscured.
format

数据格式，type一起指定请求参数的数据类型。

dataType

1.2.2引入的新属性，替代type和format来指定数据类型。

description

参数说明

required

是否是必须参数， 默认是false

defaultValue

2.1.0引入的新属性，用于指定参数的默认值。

4.枚举 com.cpjit.swagger4j.annotation.DataType

可用的数据类型