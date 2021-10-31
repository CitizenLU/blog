在IDEA中使用tomcat。

1.新建Maven工程，选择maven-archetype-webapp

![image-20201103210353286](https://i.loli.net/2021/10/31/iZ9hSXG8gm2HPcz.png)

2.一路正常next

![image-20201103210543027](https://raw.githubusercontent.com/CitizenLU/blog/main/images/image-20201103210543027.png)

3.出现BUILD SUCCESS，表示导入成功

![image-20201103210736033](https://raw.githubusercontent.com/CitizenLU/blog/main/images/image-20201103210736033.png)

4.配置tomcat

![image-20201103210822841](https://raw.githubusercontent.com/CitizenLU/blog/main/images/image-20201103210822841.png)

![image-20201103211025810](https://raw.githubusercontent.com/CitizenLU/blog/main/images/image-20201103211025810.png)

url一定要加上项目名_war，否则报404错误

![image-20201103213637391](https://raw.githubusercontent.com/CitizenLU/blog/main/images/image-20201103213637391.png)

![image-20201103211444475](https://raw.githubusercontent.com/CitizenLU/blog/main/images/image-20201103211444475.png)

完成后启动项目成功

![image-20201103213743824](https://raw.githubusercontent.com/CitizenLU/blog/main/images/image-20201103213743824.png)