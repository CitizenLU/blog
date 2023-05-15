XML(EXtensible Markup Language)，是一种可扩展的标记语言。
标记语言: 通过标签来描述数据的一门语言(标签有时我们也将其称之为元素)。
可扩展：标签的名字是可以自定义的,XML文件是由很多标签组成的,而标签名是可以自定义的。

例如

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--用于描述多个学生信息-->
<students>
    <!--第一个学生信息-->
    <student id="1">
        <name>张三</name>
        <age>23</age>
    </student>
    <!--第二个学生信息-->
    <student id="2">
        <name>李四</name>
        <age>24</age>
    </student>
</students>
```



XML解析：从xml中获取到数据。

常见的解析思想：

DOM(Document Object Model)文档对象模型：是把文档的各个组成部分看做成对应的对象。把xml文件全部加载到内存，在内存中形成一个树形结构，再获取对应的值。

Element对象，Arrtibute对象和Text对象，都继承于Node对象。

![image-20230323223054164](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20230323223054164.png)

常见的解析工具

+ JAXP: SUN公司提供的一套XML的解析的API
+ JDOM: 开源组织提供了一套XML的解析的API-jdom
+ DOM4J: 开源组织提供了一套XML的解析的API-dom4j,全称：Dom For Java
+ pull: 主要应用在Android手机端解析XML

```xml
<!-- https://mvnrepository.com/artifact/org.dom4j/dom4j -->
<dependency>
    <groupId>org.dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>2.1.3</version>
</dependency>
```



















































