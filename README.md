Author: Ismayil Tahirli, Jafar Jafarov

Created: 07.01.2021

<h1>Billing-Gateway (Billing Selector)</h1>

## Table of contents ##

1. [About](#about)
2. [Technical](#technical)
3. [Installation](#installation)
4. [Property Files](#property-files)
5. [Database Structure](#database-structure)
6. [Flow](#flow)
7. [Log Based Support Instruction](#log-based-support-instruction)
8. [Services](#services-utils)
9. [Troubleshooting](#troubleshooting)

## About ##

Billing gateway application accept incoming billing request, parse it to json format (if it required) and redirect to required application via billing code.

## Technical ##


````
TEST Server IP: 10.254.42.53

PRODUCTION Server IP: 172.25.3.40

TEST Database IP:

PRODUCTION Database IP:


Database name: gateway_db

Application port: 8080

Application routings: /clear/routing,  /clear/property

Log path: /opt/$TOMCAT_PATH/log/billing-gateway

Java: Oracle 1.8 LTS

Mysql version: 5.6
````




## Installation ## 

1. Create log directory in /opt/$TOMCAT_PATH/log/billing-gateway/ if not exists

2. Create database gateway_db if not exists

````sql
 CREATE DATABASE IF NOT EXISTS gateway_db;
````
3. Create user for gateway_db

````sql 
CREATE USER 'gateway'@'localhost' IDENTIFIED BY 'Gateway2019!';
GRANT ALL PRIVILEGES ON gateway_db.* TO 'gateway'@'localhost';
FLUSH PRIVILEGES;
````
4. Create tables in gateway_db;

````sql 
use gateway_db;

CREATE TABLE IF NOT EXISTS routing_config (
merchant_name VARCHAR(255) NOT NULL, 
bill_code VARCHAR(55) NOT NULL PRIMARY KEY,
link TEXT NOT NULL 
);


CREATE TABLE IF NOT EXISTS billing_property (
billing_version VARCHAR(25) NOT NULL,
billing_interface VARCHAR(255) NOT NULL,
billing_info VARCHAR(255) NOT NULL
);
````

## Property Files ##


This application has 2 types of property files :
application.properties
application-[dev/production].properties

First give your attention to application.propertiesl file :

````properties
server.port: 8080
````

This is application’s port configuration.

````properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.platform=mysql
spring.datasource.url=jdbc:mysql://localhost:3306/gateway_db
````


This is application database configuration.

````properties
spring.profiles.active:  production
````

Here you can see application’s active profile

According to active profile look at the second properties file : application-production.properties

````properties
spring.datasource.username=gateway
spring.datasource.password=Gateway2019!
````


Which is database connection credentials


## Database Structure ## 

Name | Type | Description | Example |
--- | --- | --- | --- |
merchant_name | varchar(255)  | Merchant name| Azerishiq | 
bill_code  | varchar(55) | Merchant billing code | 914 |
link  | text | Redirect url | http://localhost/azerishiq/ | 
parse_to_json  | varchar(1) | check, that request needed to be parsed or not | 1 | 


Name | Type | Description | Example |
--- | --- | --- | --- |
billing_version | varchar(25) | Current billing version| 1.3 | 
billing_interface | varchar(255)  | Current billing interface | AZC_Billing - Azericard |
billing_info   | varchar(255)  | Additional billing information | Billing-Gateway | 


## Flow ## 



Incoming request from Billing
````xml
<mBilling Version="1.3">
<STAN>1001234561</STAN>
<Request Type = "account">
<RRN>10012345678910111</RRN>
<BillCode>914</BillCode>
<Terminal>100111111111</Terminal>
<Target>1002222222</Target>
<Amount>1.00</Amount>
<Currency>AZN</Currency>
<Date>30112020</Date>
<Time>130610</Time>
</Request>
</mBilling>
````


For additional information about all fields look at mBilling documentation (mBilling1_3.pdf)


Convert request to json or move in XML format at once.

Get value of convert_to_json from routing_config table via billing code and if it’s true convert Xml request to json.
Then redirect this json request to a link, which we get from routing_config table via bill code.


If value of convert_to_json from routing_config table is false, xml request redirected to a link the same way as a json request.

Otherwise service will return a “System error” response.

## Log Based Support Instruction ## 

Application logs are stored in the directory /opt/$TOMCAT_PATH/log/billing-gateway/. Live log file is “billing-gateway.log”

Catalina logs are stored in /opt/$TOMCAT_PATH/logs/. Live log file is “catalina.out”

Deployment logs are stored in "catalina.out" file.

Success application deployment log:

````

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)

2020-12-09 17:39:19.058  INFO 15532 --- [           main] c.a.k.BillingGatewayApplication          : Starting BillingGatewayApplication on LAPTOP-0UGLC5JA with PID 15532 (C:\Users\Ismail\IdeaProjects\billing-gateway\target\classes started by Ismail in C:\Users\Ismail\IdeaProjects\billing-gateway)
2020-12-09 17:39:19.060  INFO 15532 --- [           main] c.a.k.BillingGatewayApplication          : The following profiles are active: dev
2020-12-09 17:39:20.131  INFO 15532 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-12-09 17:39:20.151  INFO 15532 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-12-09 17:39:20.152  INFO 15532 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.16]
2020-12-09 17:39:20.157  INFO 15532 --- [           main] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [C:\Program Files\Java\jdk1.8.0_261\bin;C:\WINDOWS\Sun\Java\bin;C:\WINDOWS\system32;C:\WINDOWS;C:\Program Files\Common Files\Oracle\Java\javapath;C:\Program Files (x86)\Common Files\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Program Files\NVIDIA Corporation\NVIDIA NvDLISR;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;C:\Program Files\Git\cmd;C:\Program Files (x86)\NVIDIA Corporation\PhysX\Common;C:\Users\Ismail\AppData\Local\Microsoft\WindowsApps;;.]
2020-12-09 17:39:20.253  INFO 15532 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-12-09 17:39:20.253  INFO 15532 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1153 ms
2020-12-09 17:39:20.616  INFO 15532 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-12-09 17:39:20.851  INFO 15532 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-12-09 17:39:20.853  INFO 15532 --- [           main] c.a.k.BillingGatewayApplication          : Started BillingGatewayApplication in 2.116 seconds (JVM running for 2.858)
````

Now let's analyze this log step by step.

````
2020-12-09 17:39:19.058  INFO 15532 --- [           main] c.a.k.BillingGatewayApplication          : Starting BillingGatewayApplication on LAPTOP-0UGLC5JA with PID 15532 (C:\Users\Ismail\IdeaProjects\billing-gateway\target\classes started by Ismail in C:\Users\Ismail\IdeaProjects\billing-gateway)
````


Here you can see starting application name “Starting BillingGatewayApplication”, server’s hostname “on LAPTOP-0UGLC5JA” and application process id (PID)


------------------------------------------------------------------------


````


2020-12-09 17:39:19.060  INFO 15532 --- [           main] c.a.k.BillingGatewayApplication          : The following profiles are active: dev
````

Here you can see current application profile dev or prod. Note that if you are deploying an application on a production server , the active profile must be prod.

------------------------------------------------------------------------

````

2020-12-09 17:39:20.851  INFO 15532 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
````


Here you can see application active ports

## Services ## 
Not much going on in the services department. The application itself has 3 main services which are:

BillingGatewayService - is responsible for request proceeding. Incoming xml request is propagated 
to this service and depending on it's attributes is converted or not to json format and forwarded further
and depending on the response returns.
 

BillingPropertyService - is responsible for extraction of BillingProperty entity. 
BillingProperty entity stores in itself billing gateway metadata like billingVersion,
billingInterface, billingInfo and is being extracted from db via 
BillingPropertyService service - BillingPropertyDbService.

RoutingService - is responsible for extraction of Routing entities. 
Routing entity itself contains merchantName, billCode, url to which request will be forwarded
and a status depending on which the request will be or not converted from xml format to json.
Routing entity itself is being extracted from db via RoutingService service - RoutingDbService.

## Troubleshooting ## 
This application has common exceptions. By their names, messages and in the worth case stack trace
you can define what's wrong. List of these common exceptions:

CommonException - if you are seeing this exception, concentrate on the generated message and stacktrace.
You will not be seeing this exception a whole lot, as all other common errors extend this exception.
For example you might be seeing this exception if during flow execution something happened with database,
but it is not something that happens often.Wait, how do you know that something happened to database during
flow execution? Easy, you check out message and stacktrace of CommonException as the database error will be
wrapped into it.

CommonHttpConnectionRefusedException - as the name implies this exception occurs, if during flow execution 
in the moment of http request, server to which request has been proceeded, refused to create a connection.
In exception message can be viewed 

CommonHttpTimeoutException - as the name implies this exception occurs, if during flow execution on 
http request was not received response in given period of time. In exception message can be viewed await time
in milliseconds and url to which request has been proceeded.

CommonInvalidUriException - as the name implies this exception occurs, if during flow execution the uri format 
to which http request will be proceeded is in wrong format. In exception message can be viewed uri itself.

CommonParsingException - this exception occurs when some value was not been able to be converted from some
format to java object, for example if http request returned strange data instead of json formatted data.
In exception message you will see the value itself.

CommonXSDValidationException - this exception occurs when incoming request value was not been able to pass xml validation.
Exception message itself will contain incoming request value and exact reason why validation failed.

NoBillingPropertyException - occurs when not a single billing property was found.

PojoToXmlConvertException - as the name implies this exception occurs, if during flow java object was not been able 
to be converted to xml value. 

RoutingNotFoundByAttributeException - as the name implies this exception occurs, when Routing with
such attribute is not found with given value. Exception message itself will contain incoming attribute 
and its value.

UnsupportedRequestTypeException - is thrown, when mbiliing request came with invalid request type parameter.

XmlToPojoConvertException - as the name implies this exception occurs, if during flow xml value was not been able 
to be converted to java object.

Example of one of listed exceptions, more precisely of CommonHttpConnectionRefusedException. It will look in log file like this:

````
[INFO] 2021-01-14 05:08:00,815 com.azericard.billing.gateway.service.BillingGatewayService proceedRequest - Incoming request for URL {http://localhost:8081/}
  {<mBilling Version="1.3">
	<STAN>1001234561</STAN>
	<Request Type="account">
		<RRN>10012345678910111</RRN>
		<BillCode>test</BillCode>
		<Terminal>100111111111</Terminal>
		<Target>1002222222</Target>
		<Amount>1.00</Amount>
		<Currency>AZN</Currency>
		<Date>30112020</Date>
		<Time>130610</Time>
	</Request>
</mBilling>
}
[INFO] 2021-01-14 05:08:00,824 com.azericard.billing.gateway.service.BillingGatewayService proceedForJsonConvertingMBilling - Send JSON request(MBilling) : {BillingEntity(stan=1001234561, responseStatus=null, errorDescription=null, response=null, request=BodyData(type=account, rrn=10012345678910111, terminal=100111111111, target=1002222222, amount=1.00, currency=AZN, balance=null, billCode=test))}
[INFO] 2021-01-14 05:08:00,851 com.azericard.billing.gateway.configuration.common.rest.template.CommonLoggingClientHttpRequestInterceptor traceRequest - 
=======================request begin====================================================
Request/Response correlation id  : e606ea72-028c-4ab3-a6cf-cb78ee4c87cc
URI                              : http://localhost:8081/mbilling
Method                           : POST
Headers                          : [Accept:"application/json", Content-Type:"application/json", Content-Length:"190"]
Request body as string           : {"STAN":"1001234561","Request":{"Type":"account","RRN":"10012345678910111","Terminal":"100111111111","Target":"1002222222","Amount":"1.00","Currency":"AZN","Balance":null,"BillCode":"test"}}
=======================request end======================================================
[ERROR] 2021-01-14 05:08:02,875 com.azericard.billing.gateway.controller.BillingExceptionHandler logError - 
Rest handled exception for request with '10012345678910111' RRN
Rest handled exception canonical name : 'com.azericard.billing.gateway.exceptions.CommonHttpConnectionRefusedException'
Rest handled exception message : 'CONNECTION_REFUSED_DURING_JSON_FORWARD_CALL - Connection to 'http://localhost:8081/mbilling' url has been refused during json request forward call'
Rest handled exception stack trace : 
com.azericard.billing.gateway.exceptions.CommonHttpConnectionRefusedException: CONNECTION_REFUSED_DURING_JSON_FORWARD_CALL - Connection to 'http://localhost:8081/mbilling' url has been refused during json request forward call
	at com.azericard.billing.gateway.utils.CommonHttpClientWrapper.wrapResourceAccessException(CommonHttpClientWrapper.java:127)
	at com.azericard.billing.gateway.utils.CommonHttpClientWrapper.sendJsonRequest(CommonHttpClientWrapper.java:76)
	at com.azericard.billing.gateway.service.BillingGatewayService.proceedForJsonConvertingMBilling(BillingGatewayService.java:105)
	at com.azericard.billing.gateway.service.BillingGatewayService.proceedRequest(BillingGatewayService.java:79)
	at com.azericard.billing.gateway.controller.BillingController.getBillingRequest(BillingController.java:21)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:878)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:792)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:652)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:733)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:888)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1597)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)
Caused by: org.springframework.web.client.ResourceAccessException: I/O error on POST request for "http://localhost:8081/mbilling": Connection refused: connect; nested exception is java.net.ConnectException: Connection refused: connect
	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:746)
	at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:672)
	at org.springframework.web.client.RestTemplate.postForEntity(RestTemplate.java:447)
	at com.azericard.billing.gateway.utils.CommonHttpClientWrapper.sendJsonRequest(CommonHttpClientWrapper.java:61)
	... 53 more
Caused by: java.net.ConnectException: Connection refused: connect
	at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
	at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:607)
	at sun.net.NetworkClient.doConnect(NetworkClient.java:175)
	at sun.net.www.http.HttpClient.openServer(HttpClient.java:463)
	at sun.net.www.http.HttpClient.openServer(HttpClient.java:558)
	at sun.net.www.http.HttpClient.<init>(HttpClient.java:242)
	at sun.net.www.http.HttpClient.New(HttpClient.java:339)
	at sun.net.www.http.HttpClient.New(HttpClient.java:357)
	at sun.net.www.protocol.http.HttpURLConnection.getNewHttpClient(HttpURLConnection.java:1226)
	at sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1162)
	at sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1056)
	at sun.net.www.protocol.http.HttpURLConnection.connect(HttpURLConnection.java:990)
	at org.springframework.http.client.SimpleBufferingClientHttpRequest.executeInternal(SimpleBufferingClientHttpRequest.java:76)
	at org.springframework.http.client.AbstractBufferingClientHttpRequest.executeInternal(AbstractBufferingClientHttpRequest.java:48)
	at org.springframework.http.client.AbstractClientHttpRequest.execute(AbstractClientHttpRequest.java:53)
	at org.springframework.http.client.BufferingClientHttpRequestWrapper.executeInternal(BufferingClientHttpRequestWrapper.java:63)
	at org.springframework.http.client.AbstractBufferingClientHttpRequest.executeInternal(AbstractBufferingClientHttpRequest.java:48)
	at org.springframework.http.client.AbstractClientHttpRequest.execute(AbstractClientHttpRequest.java:53)
	at org.springframework.http.client.InterceptingClientHttpRequest$InterceptingRequestExecution.execute(InterceptingClientHttpRequest.java:109)
	at com.azericard.billing.gateway.configuration.common.rest.template.CommonLoggingClientHttpRequestInterceptor.intercept(CommonLoggingClientHttpRequestInterceptor.java:29)
	at org.springframework.http.client.InterceptingClientHttpRequest$InterceptingRequestExecution.execute(InterceptingClientHttpRequest.java:93)
	at org.springframework.http.client.InterceptingClientHttpRequest.executeInternal(InterceptingClientHttpRequest.java:77)
	at org.springframework.http.client.AbstractBufferingClientHttpRequest.executeInternal(AbstractBufferingClientHttpRequest.java:48)
	at org.springframework.http.client.AbstractClientHttpRequest.execute(AbstractClientHttpRequest.java:53)
	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:737)
	... 56 more
````

In application.properties file you can find to additional configurations
````
common.rest.template.connect.read.timeout.period.ms=4000
common.rest.template.add.common.logging.client.http.request.interceptor=true
````
common.rest.template.connect.read.timeout.period.ms - configuration property is responsible for amount of 
milliseconds that http client is going to wait for response.

common.rest.template.add.common.logging.client.http.request.interceptor - configuration property accepts true/false
values and is responsible for printing http client outgoing requests and their responses. Lower will be mentioned example
of http client outgoing request and its response, if the property is set to true. 

````
[INFO] 2021-01-15 18:16:06,528 com.azericard.billing.gateway.service.BillingGatewayService proceedForJsonConvertingMBilling - Send JSON request(MBilling) : {BillingEntity(stan=1001234561, responseStatus=null, errorDescription=null, response=null, request=BodyData(type=account, rrn=10012345678910111, terminal=100111111111, target=1002222222, amount=1.00, currency=AZN, balance=null, billCode=test))}
[INFO] 2021-01-15 18:16:06,529 com.azericard.billing.gateway.configuration.common.rest.template.CommonLoggingClientHttpRequestInterceptor traceRequest - 
=======================request begin====================================================
Request/Response correlation id  : 032429d7-2625-404d-8ea5-461c956fac94
URI                              : http://localhost:8081/mbilling
Method                           : POST
Headers                          : [Accept:"application/json", Content-Type:"application/json", Content-Length:"190"]
Request body as string           : {"STAN":"1001234561","Request":{"Type":"account","RRN":"10012345678910111","Terminal":"100111111111","Target":"1002222222","Amount":"1.00","Currency":"AZN","Balance":null,"BillCode":"test"}}
=======================request end======================================================
[INFO] 2021-01-15 18:16:06,535 com.azericard.billing.gateway.configuration.common.rest.template.CommonLoggingClientHttpRequestInterceptor traceResponse - 
=======================response begin===================================================
Request/Response correlation id  : 032429d7-2625-404d-8ea5-461c956fac94
Status code                      : 200 OK
Headers                          : [Content-Type:"application/json", Content-Length:"3", Date:"Fri, 15 Jan 2021 14:16:06 GMT", Keep-Alive:"timeout=60", Connection:"keep-alive"]
Request execution period in ms   : 2
Response body as string          : kek
=======================response end=====================================================
````
