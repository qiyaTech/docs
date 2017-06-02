# swagger 自动生成Api文档

本文针对spring boot 1.4 进行集成

## maven安装

加入spring fox、swagger包、spring fox swagger ui包

```
        <springfox-version>2.6.1</springfox-version>
        <swagger-core-version>1.5.10</swagger-core-version>
```

```xml
        <!-- Api文档 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>${springfox-version}</version>
        </dependency>
        <dependency>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-core</artifactId>
            <version>${swagger-core-version}</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>${springfox-version}</version>
        </dependency>
```

## spring boot 配置

* **在Application类中增加@EnableSwagger2**

```java
@ComponentScan({ "com.qiya.*" })
@SpringBootApplication
@EnableJms
@EnableSwagger2
public class ApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiApplication.class, args);
    }
```

* **增加一个config类型新类，针对swagger配置说明，以及针配置相关全局参数、以及配置对应的文档分组。**

  * 增加Api说明

  ```java
  private ApiInfo bizApiInfo() {
          return new ApiInfoBuilder()
                  .title("闪信Api")   
                  .contact(new Contact("奇伢", "http://www.qiya.tech", "ddy@qiya.tech"))
                  .version("1.0.0")
                  .build();
  ```

* 增加相应需要中上Api说明的controller地址,可以用正则匹配

```java
private Predicate<String> bizPaths() {
        return or(
                regex("/getLeftMessageCount.*"),
                regex("/getBuyLeftCount.*"),
                regex("/getMessageOrder.*"),
                regex("/sendMsgV1.*"),
                regex("/sendMessageAgain.*"),
                regex("/createOrderBefore.*"),
                regex("/createOrder.*"),
                regex("/updateOrderStatus.*"),
                regex("/updateOrderFail.*"),
                regex("/orderDealSuccess.*"),
                regex("/sendMessage.*"),
                regex("/addMsgStatus.*"),
                regex("/pushIOSMsg.*"),
                regex("/getSmsStatus.*"),
                regex("/getMessageHistory.*")
            );
    }
```

* 把之前两步合并生成文档类，
  * 如果要增加全局参数，如果token，可以在这块增加。
  * 如果有一些实体类嵌套比较深，也可以这里增加实体的引用

```java
@Bean
    public Docket springfoxBiz() {
        ArrayList<Parameter> arrayList = new ArrayList();
        arrayList.add((new ParameterBuilder()
                .name(tokenName)
                .description("存储用户信息的token,仅在需要登录操作中使用")
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false)
                .build()));

         return (new Docket(DocumentationType.SWAGGER_2)
                .groupName("flashApi")
                .useDefaultResponseMessages(false)
                .apiInfo(bizApiInfo())
                .select()
                .paths(bizPaths())
                .build())
                .additionalModels(typeResolver.resolve(User.class))
                .globalOperationParameters(arrayList);
    }
```



