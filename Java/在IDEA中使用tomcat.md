在IDEA中使用tomcat。

1.新建Maven工程，选择maven-archetype-webapp

![image-20211201233109576](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233109576.png)



2.一路正常next

![image-20211201233122638](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233122638.png)



3.出现BUILD SUCCESS，表示导入成功

![image-20211201233132747](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233132747.png)



4.配置tomcat

![image-20211201233144828](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233144828.png)



![image-20211201233156973](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233156973.png)





url一定要加上项目名_war，否则报404错误

![image-20211201233209504](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233209504.png)



![image-20211201233219608](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233219608.png)





完成后启动项目成功

![image-20211201233227157](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20211201233227157.png)