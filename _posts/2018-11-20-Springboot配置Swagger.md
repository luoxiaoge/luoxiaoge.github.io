## Springboot2整合Swagger2
 >   前提准备：
   1 已经搭建好springboot项目
     [springboot官网有demo](https://spring.io/projects/spring-boot#overview)  
   2.使用maven管理jar包 
   3.[源代码地址](https://github.com/luoxiaoge/Springboot2.git)

### 配置Maven Pom.xml文件 添加依赖
```   
      <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.8.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.8.0</version>
        </dependency>
```
### 通过bean配置信息
```

@Configuration

@EnableSwagger2

public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // 自行修改为自己的包路径
                .apis(RequestHandlerSelectors.basePackage("com.luoc.hello"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("springBoot系统接口文档")
                .description(" 使用RESTful APIs")
                .termsOfServiceUrl("NO terms of service")
                .contact(new Contact("luoc", "http://www.baidu.com", "1012293013@qq.com"))
                .version("1.0")
                .build();
    }
}
```
### 配置请求接口注解
```

@RestController

@Api(tags="测试接口模块")

public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();
    private static final Logger log = LoggerFactory.getLogger(GreetingController.class);

    @Autowired
    private UserMapper mapper;

    @Autowired
    private MongoTemplate mongoTemplate;

    @Autowired
    private UserRepository userRepository;

    @GetMapping("/greeting")
    @Auth(auth = true)
    @ApiOperation(value="测试信息", notes = "测试信息")
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        log.info("该用户信息:{}",mapper.findUserById(3));
        log.info(JSON.toJSONString(mongoTemplate.findAll(User.class)));
        return new Greeting(counter.incrementAndGet(),
                 String.format(template, name));
    }
}
```

### 启动项目类
    根据自己设定的端口进行访问：http://localhost:<server-port>/swagger-ui.html

 
