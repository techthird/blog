# fast-mock一个可快速生成测试类、Mock方法的工具，提升你50%的单测效率



>fast-mock是一个快速生成测试类、Mock方法的工具，它使写集成、单元测试这件事情，变得简单而有趣！

一方面，写单元测试本身比较繁琐，技术挑战不大，很多研发工程师不愿意去写；  
另一方面，国内研发比较偏向“快、糙、猛” ，容易因为开发进度紧，导致单元测试的执行虎头蛇尾。  
关键问题还是研发团队需要建立对单元测试正确的认知，渐进明细，逐步改进。

fast-mock可以解决繁琐的单测编写，提升工作效率，让研发工程师更专注业务的实现。  
PS：团队前期没有单测的沉淀，可以先从整个接口的集成测试开始，逐步重构代码，往单测的方向靠近。

## 特点
* **快速生成测试类**： 生成一个包含设置Case名称、组装测试接口的参数、调用方法、断言的测试模板。
* **自动识别生成Mock方法**：根据测试接口自动识别需要Mock的方法，并且进行代码生成。
* **自动构造入参、出参实体**：根据入参、出参对象，自动进行数据构造。简单理解就是实例化对象，并进行set操作。
* **灵活切换**：支持同时Mock、查询数据库、Redis，RPC请求。


## 使用  
### 1、引入mock依赖：
```xml
<properties>
    <testable.version>0.6.2</testable.version>
</properties>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.8.47</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.alibaba.testable</groupId>
    <artifactId>testable-all</artifactId>
    <version>${testable.version}</version>
    <scope>test</scope>
</dependency>
```
### 2、依赖插件：
 ```xml
<plugins>
    <plugin>
        <groupId>io.github.techthird</groupId>
        <artifactId>fast-mock-maven-plugin</artifactId>
        <version>1.0.5</version>
        <configuration>
            <!-- 待测试类的包名com.test.demo.service.provider，或者测试指定类文件xxxx.java） -->
            <testPackageName>com.test.demo.service.provider.ItemReadServiceImpl.java</testPackageName>
            <!-- 指定生成Test方法，多个方法以,号分隔，默认生成一个类中所有方法 -->
            <testMethods>method1,method2</testMethods>
        </configuration>
    </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.22.2</version>
        <configuration>
            <argLine>-javaagent:${settings.localRepository}/com/alibaba/testable/testable-agent/${testable.version}/testable-agent-${testable.version}.jar</argLine>
            <skip>true</skip>
        </configuration>
    </plugin>
</plugins>
 ```

### 3、生成测试代码 
在引入插件的项目模块下运行maven插件 fast-mock:test 命令
```shell
mvn fast-mock:test
```
默认生成测试代码存放目录：test/你的测试类包名  

![image](https://img-blog.csdnimg.cn/img_convert/acd778aaec147f22f4776eb3b8928673.png)

![image](https://img-blog.csdnimg.cn/img_convert/c848781c7e0e3286872bb3cb2223d7f8.png)

![image](https://img-blog.csdnimg.cn/img_convert/9556b93030a3621cffd524af3de7fb91.png)

### 4、小提示
* 重复生成：一个被测类重复生成，为了防止代码的覆盖，将采用方法对比后进行追加处理。将始终一个Test类、一个Mock类。
* 尽量保证生成的测试、Mock方法唯一，对同名不同参数的方法支持不太友好。
* BaseTestCase类的说明：该类定义了启动Spring应用程序所需的配置。默认配置如下：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class BaseTestCase {

}
```

## 开源项目地址
[https://github.com/techthird/fast-mock-test](https://github.com/techthird/fast-mock-test)

## 问题反馈
目前项目处于初始阶段，Bug肯定是无法避免的，团队试用下来，不是特别偏门的接口，都能成功生成。  
如果您感兴趣或者有问题可加微信guiyong_chen沟通交流。 备注：fast-mock


## 感谢
1、一个堪称指哪打哪的Mock框架 [https://github.com/alibaba/testable-mock](https://github.com/alibaba/testable-mock) 
2、站在巨人的肩膀上，完成二次改造
[https://github.com/chenhaoxiang/primo/tree/master/primo-generator-mock-test](https://github.com/chenhaoxiang/primo/tree/master/primo-generator-mock-test)