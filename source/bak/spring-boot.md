1: 使用 idea 新建一个spring boot项目

```
@SpringBootApplication
public class GirlApplication {

	public static void main(String[] args) {
		SpringApplication.run(GirlApplication.class, args);
	}
}

```

2: 加一个controller，`@RequestMapping`的两种匹配方式

```
@RestController
@RequestMapping(value = "/say")
public class HelloController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String sayHello() {
        return "hello";
    }

    @GetMapping(value = "/yes")
    public String sayYes(){
        return "yes";
    }

}

```

3: 启动方式。

	* IDE 自动 run
	* maven spring-boot:run; 进入到项目pom文件目录，运行
	* maven install ; java -jar [project name].jar; 进入到pom文件下，`mvn install`，之后进入到target目录，运行java -jar [project name].jar

4: 属性配置

a. application.properties

```
server.port=8081
server.context-path=/girl
```

b. application.yml

```
server:
  port: 8082
  context-path: /girl2
```

c.配置初始值


5: 多环境部署
`java -jar target/girl-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev`


6:spring-Data-Jpa
Jpa

7:maven打包跳过测试
-Dmaven.test.skip=true
	
	
	
	
	