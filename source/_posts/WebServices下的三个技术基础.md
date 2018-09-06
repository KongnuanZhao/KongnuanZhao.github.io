title: WebServices下的三个技术基础
date: 2015-01-12 15:16:03
author: 赵空暖
tags: java
categories: java
---
#web services的三个技术基础#
* SOAP(Simple Object Access Protocol) - 简单对象访问协议
* WSDL(Web Services Definition Language) - WEB SERVICES定义语言
* UDDI(Universal Description,Discovery and Integration) - 统一描述、发现、集成


下面分开来说，
##SOAP##
![soap](/image/soap.png)
是一个基于XML的协议，其定义了一套编码规则，该规则定义如何将数据表示为消息，以及怎样通过http协议来传输SOAP消息，它主要由以下四部分组成:
* SOAP信封（Envelope）定义了一个框架，该框架描述了消息中的内容是什么，包括消息的内容、发送者、接受者、处理者以及如何处理等信息。
* SOAP编码规则，它定义了一种序列化的机制，用于交换应用程序所定义的数据类型的实例。
* SOAP RPC表示，它定义了用于表示远程调用和应答的协定。
* SOAP绑定，它定义了一种使用底层传输协议来完成在结点间交换SOAP信封的约定。
举个栗子，以下是 SOAP 1.2 请求和响应示例：
```xml
POST /WebServices/MobileCodeWS.asmx HTTP/1.1
Host: webservice.webxml.com.cn
Content-Type: application/soap+xml; charset=utf-8
Content-Length: length

<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
  <soap12:Body>
    <getMobileCodeInfo xmlns="http://WebXml.com.cn/">
      <mobileCode>string</mobileCode>
      <userID>string</userID>
    </getMobileCodeInfo>
  </soap12:Body>
</soap12:Envelope>
```

```xml
HTTP/1.1 200 OK
Content-Type: application/soap+xml; charset=utf-8
Content-Length: length

<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
  <soap12:Body>
    <getMobileCodeInfoResponse xmlns="http://WebXml.com.cn/">
      <getMobileCodeInfoResult>string</getMobileCodeInfoResult>
    </getMobileCodeInfoResponse>
  </soap12:Body>
</soap12:Envelope>
```
<b>HTTP GET</b>
以下是 HTTP GET 请求和响应示例。所显示的占位符需替换为实际值。
```bash
GET /WebServices/MobileCodeWS.asmx/getMobileCodeInfo?mobileCode=string&userID=string HTTP/1.1
Host: webservice.webxml.com.cn
```

```bash
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8
Content-Length: length

<?xml version="1.0" encoding="utf-8"?>
<string xmlns="http://WebXml.com.cn/">string</string>
```
<b>HTTP POST</b>
以下是 HTTP POST 请求和响应示例。所显示的占位符需替换为实际值。
```bash
POST /WebServices/MobileCodeWS.asmx/getMobileCodeInfo HTTP/1.1
Host: webservice.webxml.com.cn
Content-Type: application/x-www-form-urlencoded
Content-Length: length

mobileCode=string&userID=string
```

```bash
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8
Content-Length: length

<?xml version="1.0" encoding="utf-8"?>
<string xmlns="http://WebXml.com.cn/">string</string>
```
##WSDL##
![webservices](/image/webservices.png)
一次Web Services的调用实际上是发送SOAP消息（即XML文档），以一个[国内手机号码归属地查询WEB服务 通讯和通信](http://webservice.webxml.com.cn/WebServices/MobileCodeWS.asmx?wsdl)Services为例：
分析一个portType结点
```xml
<wsdl:portType name="MobileCodeWSSoap">
 <wsdl:operation name="getMobileCodeInfo">
   <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">...</wsdl:documentation>
   <wsdl:input message="tns:getMobileCodeInfoSoapIn"/>
   <wsdl:output message="tns:getMobileCodeInfoSoapOut"/>
 </wsdl:operation>
 <wsdl:operation name="getDatabaseInfo">
   <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">
   <br /><h3>获得国内手机号码归属地数据库信息</h3><p>输入参数：无；返回数据：一维字符串数组（省份 城市 记录数量）。</p><br />
  </wsdl:documentation>
  <wsdl:input message="tns:getDatabaseInfoSoapIn"/>
  <wsdl:output message="tns:getDatabaseInfoSoapOut"/>
 </wsdl:operation>
</wsdl:portType>
```
该结点里面有两个operation结点，分别为`getMobileCodeInfo`和`getDatabaseInfo`,子节点`getMobileCodeInfo`下有input和output即输入和输出，怎么知道输入的是什么呢？往上面去找message元素，即
```xml
<wsdl:message name="getMobileCodeInfoSoapIn">
	<wsdl:part name="parameters" element="tns:getMobileCodeInfo"/>
</wsdl:message>
```
可知这是一个参数，具体是什么类型，查看type元素schema定义
```xml
<s:element name="getMobileCodeInfo">
	<s:complexType>
		<s:sequence>
		 <s:element minOccurs="0" maxOccurs="1" name="mobileCode" type="s:string"/>
		 <s:element minOccurs="0" maxOccurs="1" name="userID" type="s:string"/>
		</s:sequence>
	</s:complexType>
</s:element>
```
可知这个输入参数是个复杂类型，有两个字符串类型的输入参数包含在一个有序序列当中，即`name="mobileCode"`和`name="userID"`最小出现0次，最多出现1次
即传入的消息是：
```xml
<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
  <soap12:Body>
    <getMobileCodeInfo xmlns="http://WebXml.com.cn/">
      <mobileCode>string</mobileCode>
      <userID>string</userID>
    </getMobileCodeInfo>
  </soap12:Body>
</soap12:Envelope>
```
类似的分析output返回的消息,返回的消息即：
```xml
HTTP/1.1 200 OK
Content-Type: application/soap+xml; charset=utf-8
Content-Length: length

<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
  <soap12:Body>
    <getMobileCodeInfoResponse xmlns="http://WebXml.com.cn/">
      <getMobileCodeInfoResult>string</getMobileCodeInfoResult>
    </getMobileCodeInfoResponse>
  </soap12:Body>
</soap12:Envelope>
```
测试：
![telnum](/image/telnum.png)
返回：183********：山东 潍坊 山东移动全球通卡

调用一次web services的本质：
1.客户端把调用方法参数，转换XML文档片段（SOAP消息）--该文档片段必须符合WSDL定义的格式，即input消息。
2.通过网络，把XML文档片段传给服务器。
3.服务器收到XML文档片段。
4.服务器解析XML文档片段，提取其中的数据。并把数据转换成web services所需要的参数值。
5.服务器端执行方法。
6.把执行方法得到的返回值，再次转换生成为XML文档片段（SOAP消息）即output消息。
7.通过网络把XML文档片段传给客户端
8.客户端接收到XML片段。
9.客户端解析XML片段，提取其中的数据，并把数据转换成调用Web Services的返回值。

从上面的调用本质来看，
1.和具体的语言无关，唯一的要求是该语言支持XML文档的生成、解析、支持网络传输。
2.客户端只是发送和接收消息，实际调用处理方法是在服务器端进行的。

所以，WSDL文档描述了
* what：该web services包含什么操作？
* how: 该web services怎样调用？
* where：该web services的服务地址

即只要得到了web services的WSDL文档，程序就可以调用web services。

##UDDI##
要调用一个web Service需要几方面的信息。首先需要找到满足需要的Web Service,还需要了解传送请求的模式，即如何调用这个Web Service。web是一个不固定的环境，新的web service在不断增加，旧的web service在不断删除，已有的web service的调用方式可能会随时发生变化，所系客观上需要在web service的发布者和调用者之间建立一种便于查找和发布的机制。UDDI规范为解决这些问题提供了一种途径，web Service的发布者可以将其注册到注册中心，是一种目录服务，web service的调用者可以从注册中心查找到需要的web service并进行调用。