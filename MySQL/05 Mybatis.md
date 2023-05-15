# JDBC

普通的调用。

```java
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        //1. 注册驱动
        // Class.forName("com.mysql.jdbc.Driver");
        //2. 获取连接
        String url = "jdbc:mysql://127.0.0.1:3306/db1";
        String username = "root";
        String password = "123456";
        Connection conn = DriverManager.getConnection(url, username, password);
        //3. 定义sql
        String sql = "update account set money = 2000 where id = 1";
        //4. 获取执行sql的对象 Statement
        Statement stmt = conn.createStatement();
        //5. 执行sql
        int count = stmt.executeUpdate(sql);//受影响的行数
        //6. 处理结果
        System.out.println(count);
        //7. 释放资源
        stmt.close();
        conn.close();
    }
```

查询

```java
private static void queryAccount() throws ClassNotFoundException, SQLException {
        //1. 注册驱动
        Class.forName("com.mysql.jdbc.Driver");
        //2. 获取连接
        String url = "jdbc:mysql://127.0.0.1:3306/db1";
        String username = "root";
        String password = "123456";
        Connection conn = DriverManager.getConnection(url, username, password);
        //3. 定义sql
        String sql = "select * from account";
        //4. 获取执行sql的对象 Statement
        Statement stmt = conn.createStatement();
        //5. 执行sql
        ResultSet resultSet = stmt.executeQuery(sql);
        //6. 处理结果，遍历resultSet中的所有数据
        while (resultSet.next()) {
            // 根据下标获取数据
            int id = resultSet.getInt(1);
            String name = resultSet.getString(2);
            double money = resultSet.getDouble(3);

            System.out.println("{id:["+id + "],name:[" + name + "],money:[" + money+"]}");
        }
        //7. 释放资源
        resultSet.close();
        stmt.close();
        conn.close();
    }
```

预编译sql

```java
private static void preparedStatement(String queryId) throws ClassNotFoundException, SQLException {
        //1. 注册驱动
        Class.forName("com.mysql.jdbc.Driver");
        //2. 获取连接
        String url = "jdbc:mysql://127.0.0.1:3306/db1?useSSL=false";
        String username = "root";
        String password = "123456";
        Connection conn = DriverManager.getConnection(url, username, password);
        //3. 定义sql
        String sql = "SELECT * FROM account WHERE id = ? ";
        //4. 获取执行sql的对象 Statement
        PreparedStatement stmt = conn.prepareStatement(sql);
        // 设置?的值
        stmt.setString(1,queryId);
        //5. 执行sql
        ResultSet resultSet = stmt.executeQuery();
        //6. 处理结果，遍历resultSet中的所有数据
        while (resultSet.next()) {
            // 根据下标获取数据
            int id = resultSet.getInt(1);
            String name = resultSet.getString(2);
            double money = resultSet.getDouble(3);

            System.out.println("{id:["+id + "],name:[" + name + "],money:[" + money+"]}");
        }
        //7. 释放资源
        resultSet.close();
        stmt.close();
        conn.close();
    }
```

# Mybatis

## 1、快速入门

目录结构：

![image-20230423202120718](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20230423202120718.png)

mybatis-config.xml配置文件。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 别名 -->
    <typeAliases>
        <package name="org.example.pojo"/>
    </typeAliases>
    <environments default="development">
        <!--
        environment：配置数据库连接环境信息，可以配置多个，通过default属性切换不同的environment
        -->
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!-- 数据库连接信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!-- 方法1：加载sql映射文件，不推荐。对应demo1。-->
<!--        <mapper resource="org/example/mapper/UserMapper.xml"/>-->

        <!-- 方法2：包扫描方式，会扫描路径下的所有mapper文件。对应demo2 -->
        <package name="org.example.mapper"/>
    </mappers>
</configuration>
```

UserMapper.xml配置文件，需要与mapper路径一致，且同名。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.example.mapper.UserMapper">
    <select id="selectAll" resultType="User">
        select * from tb_user;
    </select>
</mapper>
```

UserMapper接口，需要有一个和UserMapper.xml中同名的方法。

```java
public interface UserMapper {
    // selectAll与UserMapper.xml中mapper标签下，select标签的id相同
    List<User> selectAll();
}
```

demo1

```java

private static void mybatisQuickStart1() throws IOException {
        // 1.加载mybatis核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2.获取对应SqlSession对象
        SqlSession session = sqlSessionFactory.openSession();

        // 3.执行sql。selectList的参数为UserMapper.xml中mapper标签的namespace值+方法id。不推荐该写法。
        List<User> users = session.selectList("org.example.mapper.UserMapper.selectAll");
        System.out.println(users);
        session.close();
    }
```

demo2

```java
/**
 * mybatis包扫描方式
 */
private static void mybatisQuickStart2() throws IOException {
        // 1.加载mybatis核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2.获取对应SqlSession对象
        SqlSession session = sqlSessionFactory.openSession();

        // 3.1获取UserMapper代理对象
        UserMapper userMapper = session.getMapper(UserMapper.class);
        List<User> users = userMapper.selectAll();
        System.out.println(users);
        session.close();
    }
```

后续主要使用包扫描的方式。

# Mybatis完成增删改查

表结构如下：

```mysql
-- 删除tb_brand表
drop table if exists tb_brand;
-- 创建tb_brand表
create table tb_brand
(
    -- id 主键
    id           int primary key auto_increment,
    -- 品牌名称
    brand_name   varchar(20),
    -- 企业名称
    company_name varchar(20),
    -- 排序字段
    ordered      int,
    -- 描述信息
    description  varchar(100),
    -- 状态：0：禁用  1：启用
    status       int
);
-- 添加数据
insert into tb_brand (brand_name, company_name, ordered, description, status)
values ('三只松鼠', '三只松鼠股份有限公司', 5, '好吃不上火', 0),
       ('华为', '华为技术有限公司', 100, '华为致力于把数字世界带入每个人、每个家庭、每个组织，构建万物互联的智能世界', 1),
       ('小米', '小米科技有限公司', 50, 'are you ok', 1);
```

实体类Brand包含如下字段

```java
public class Brand {
    // id 主键
    private Integer id;
    // 品牌名称
    private String brandName;
    // 企业名称
    private String companyName;
    // 排序字段
    private Integer ordered;
    // 描述信息
    private String description;
    // 状态：0：禁用  1：启用
    private Integer status;

// 省略set和get方法。
    @Override
    public String toString() {
        return "Brand{" +
                "id=" + id +
                ", brandName='" + brandName + '\'' +
                ", companyName='" + companyName + '\'' +
                ", ordered=" + ordered +
                ", description='" + description + '\'' +
                ", status=" + status +
                '}';
    }
}
```



## 1、查询

步骤：

1、编写接口方法：Mapper接口

2、编写SQL语句：SQL映射文件

3、执行方法进行测试



1、Mapper接口

```java
public interface BrandMapper {
    List<Brand> selectAll();
}
```

2、SQL语句。BrandMapper.xml文件。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.example.mapper.BrandMapper">
    <!-- 结果映射 -->
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>
    <select id="selectAll" resultMap="brandResultMap">
        select * from tb_brand;
    </select>
</mapper>
```

测试类

```java
    @Test
    public void testSelectAll() throws IOException {
        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        List<Brand> brands = brandMapper.selectAll();
        System.out.println(brands);

        //5. 释放资源
        sqlSession.close();
    }
```

## 2、带条件查询

BrandMapper

注意，使用`#`是预编译SQL。`$`是直接拼接SQL。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <select id="selectById" resultMap="brandResultMap">
        select *
        from tb_brand where id=#{id};
    </select>
</mapper>
```

BrandMapper接口加一个方法。

```java
Brand selectById(int id);
```

测试类

```java
    @Test
    public void testSelectById() throws IOException {
        int id=1;

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        Brand brands = brandMapper.selectById(id);
        System.out.println(brands);

        //5. 释放资源
        sqlSession.close();
    }
```

## 3、多个查询条件

- 3.1 多个参数

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <select id="selectByCondition" resultMap="brandResultMap">
        select *
        from tb_brand
        where status = #{status}
          and company_name like #{companyName}
          and brand_name like #{brandName};
    </select>
</mapper>
```

BrandMapper接口

```java
List<Brand> selectByCondition(@Param("status")int status,@Param("companyName")String companyName,@Param("brandName")String brandName);
```

测试类

```java
    @Test
    public void testSelectByStatus() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "华为";
        String brandName = "华为";

        companyName = "%" + companyName + "%";
        brandName = "%" + brandName + "%";

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        List<Brand> brands = brandMapper.selectByCondition(status, companyName, brandName);
        System.out.println(brands);

        //5. 释放资源
        sqlSession.close();
    }
```

- 3.2 用对象封装

BrandMapper.xml不变。

BrandMapper接口。mybatis无法识别重载方法，需要将原方法注释掉。

```java
List<Brand> selectByCondition(Brand brand);
```

测试类

```java
    @Test
    public void testSelectByCondition() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "华为";
        String brandName = "华为";

        companyName = "%" + companyName + "%";
        brandName = "%" + brandName + "%";

        // 封装对象
        Brand brand = new Brand();
        brand.setStatus(status);
        brand.setCompanyName(companyName);
        brand.setBrandName(brandName);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        List<Brand> brands = brandMapper.selectByCondition(brand);
        System.out.println(brands);

        //5. 释放资源
        sqlSession.close();
    }
```

- 3.3 用Map封装

BrandMapper.xml不变。

BrandMapper接口。mybatis无法识别重载方法，需要将原方法注释掉。

```java
List<Brand> selectByCondition(Map map);
```

测试类

```java
    @Test
    public void testSelectByCondition() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "华为";
        String brandName = "华为";

        companyName = "%" + companyName + "%";
        brandName = "%" + brandName + "%";

        Map map=new HashMap<>();
        map.put("status",status);
        map.put("companyName",companyName);
        map.put("brandName",brandName);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        List<Brand> brands = brandMapper.selectByCondition(map);
        System.out.println(brands);

        //5. 释放资源
        sqlSession.close();
    }
```

## 4、动态条件查询——多条件查询

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <select id="selectByCondition" resultMap="brandResultMap">
        select *
        from tb_brand
        <!-- 使用where标签替换原来的where -->
        <where>
            <!-- if标签的test如果为true，则添加SQL语句 -->
            <if test="status != null">
                and status = #{status}
            </if>
            <if test="companyName != null and companyName != ''">
                and company_name like #{companyName}
            </if>
            <if test="brandName != null and brandName != ''">
                and brand_name like #{brandName}
            </if>
        </where>
    </select>
</mapper>
```

BrandMapper接口不变

测试类

```java
    @Test
    public void testSelectByCondition() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "华为";
        String brandName = "华为";

        companyName = "%" + companyName + "%";
        brandName = "%" + brandName + "%";

        Map map=new HashMap<>();
        map.put("brandName",brandName);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        List<Brand> brands = brandMapper.selectByCondition(map);
        System.out.println(brands);

        //5. 释放资源
        sqlSession.close();
    }
```

## 5、动态条件查询——单条件查询

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <select id="selectByConditionSingle" resultMap="brandResultMap">
        select *
        from tb_brand
        <where>
            <choose><!-- 相当于switch -->
                <when test="status != null"><!-- 相当于case -->
                    status=#{status}
                </when>
                <when test="companyName != null and companyName != ''"><!-- 相当于case -->
                    company_name like #{companyName}
                </when>
                <when test="brandName != null and brandName != ''"><!-- 相当于case -->
                    and brand_name like #{brandName};
                </when>
            </choose>
        </where>
    </select>
</mapper>
```

BrandMapper接口

```java
List<Brand> selectByConditionSingle(Brand brand);
```

测试类

```java
    @Test
    public void testSelectByConditionSingle() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "华为";
        String brandName = "华为";

        companyName = "%" + companyName + "%";
        brandName = "%" + brandName + "%";

        // 封装对象
        Brand brand = new Brand();
        brand.setStatus(status);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        List<Brand> brands = brandMapper.selectByConditionSingle(brand);
        System.out.println(brands);

        //5. 释放资源
        sqlSession.close();
    }
```

## 6、添加

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>
    <insert id="add">
        insert into tb_brand(brand_name,company_name,ordered,description,status)
        values (#{brandName},#{companyName},#{ordered},#{description},#{status});
    </insert>
</mapper>
```

BrandMapper接口

```java
void add(Brand brand);
```

测试类

```java
    @Test
    public void testAdd() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "波导手机";
        String brandName = "波导";
        String description = "手机中的战斗机";
        int ordered = 100;


        // 封装对象
        Brand brand = new Brand();
        brand.setStatus(status);
        brand.setCompanyName(companyName);
        brand.setBrandName(brandName);
        brand.setDescription(description);
        brand.setOrdered(ordered);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        brandMapper.add(brand);

        //5. 释放资源
        sqlSession.close();
    }
```

## 7、添加后，返回主键的值

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>
    <!-- useGeneratedKeys设置为true，要返回的列为id -->
    <insert id="add" useGeneratedKeys="true" keyProperty="id">
        insert into tb_brand(brand_name,company_name,ordered,description,status)
        values (#{brandName},#{companyName},#{ordered},#{description},#{status});
    </insert>
</mapper>
```

测试类

```java
    @Test
    public void testAdd() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "波导手机";
        String brandName = "波导";
        String description = "手机中的战斗机";
        int ordered = 100;


        // 封装对象
        Brand brand = new Brand();
        brand.setStatus(status);
        brand.setCompanyName(companyName);
        brand.setBrandName(brandName);
        brand.setDescription(description);
        brand.setOrdered(ordered);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        brandMapper.add(brand);
        Integer id = brand.getId();
        System.out.println(id); // 打印主键的值

        //5. 释放资源
        sqlSession.close();
    }
```

 ## 8、修改全部字段

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <update id="update">
        update tb_brand
        set brand_name=#{brandName},
            company_name=#{companyName},
            ordered=#{ordered},
            description=#{description},
            status=#{status}
        where id = #{id};
    </update>
</mapper>

```

BrandMapper接口

```java
int update(Brand brand);
```

测试类

```java
    @Test
    public void testUpdate() throws IOException {
        // 接收参数
        int status = 1;
        String companyName = "波导手机";
        String brandName = "波导";
        String description = "波导手机，手机中的战斗机";
        int ordered = 200;
        int id=5;


        // 封装对象
        Brand brand = new Brand();
        brand.setStatus(status);
        brand.setCompanyName(companyName);
        brand.setBrandName(brandName);
        brand.setDescription(description);
        brand.setOrdered(ordered);
        brand.setId(id);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        int count = brandMapper.update(brand);

        System.out.println(count);

        //5. 释放资源
        sqlSession.close();
    }
```

## 9、修改动态字段

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <update id="update">
        update tb_brand
        <set>
            <if test="brandName !=null and brandName != '' ">
                brand_name=#{brandName},
            </if>
            <if test="companyName !=null and companyName != '' ">
                company_name=#{companyName},
            </if>
            <if test="ordered !=null">
                ordered=#{ordered},
            </if>
            <if test="description !=null and description != '' ">
                description=#{description},
            </if>
            <if test="status !=null ">
                status=#{status}
            </if>
        </set>
            where id = #{id};
    </update>
</mapper>
```

测试类

```java
    @Test
    public void testUpdate() throws IOException {
        // 接收参数
        int status = 0;
        String companyName = "波导手机";
        String brandName = "波导";
        String description = "波导手机，手机中的战斗机";
        int ordered = 200;
        int id=5;


        // 封装对象
        Brand brand = new Brand();
        brand.setStatus(status);
        brand.setId(id);

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        int count = brandMapper.update(brand);

        System.out.println(count);

        //5. 释放资源
        sqlSession.close();
    }
```

## 10、删除一个

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <delete id="deleteById">
        delete
        from tb_brand
        where id = #{id};
    </delete>
</mapper>

```

BrandMapper接口

```java
void deleteById(int id);
```

测试类

```java
    @Test
    public void testDeleteById() throws IOException {
        // 接收参数
        int id=6;

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        brandMapper.deleteById(id);

        //5. 释放资源
        sqlSession.close();
    }
```

## 11、批量删除

BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.BrandMapper">
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
    </resultMap>

    <!-- mybatis 会将数组参数封装为一个map集合。
    使用@Param注解改变map集合的默认key的名称
    -->
    <delete id="deleteByIds">
        delete from tb_brand where id in
        <foreach collection="ids" item="id" separator="," open="(" close=")">
            #{id}
        </foreach>;
    </delete>
</mapper>
```

BrandMapper接口

```java
void deleteByIds(@Param("ids") int[] ids);
```

测试类

```java
    @Test
    public void testDeleteById() throws IOException {
        // 接收参数
        int[] ids={3,5,7};

        // 1.加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //3. 获取Mapper接口的代理对象
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

        //4. 执行方法
        brandMapper.deleteByIds(ids);

        //5. 释放资源
        sqlSession.close();
    }
}
```

























