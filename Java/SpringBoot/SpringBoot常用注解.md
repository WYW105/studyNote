1. @Data 自动给数据生成构造器、getter、setter方法、toString方法、equels方法等，就不用手写。

2. @SpringBootApplication 注解在Application类上，表明是一个启动类，用于启动spring工程。

3. @RestController 用在请求处理类上，@RestController = @Controller + @ResponseBody。

4. @ResponseBody注解把方法的返回值都绑定到web响应的响应体上,通常是JSON格式。==> 所以@RestController的方法返回值都会被绑定到响应体中。

5. @RequestMapping 用在controller上或者方法上，用于映射url请求到正确的方法上。

6. @RequestParam 用于把请求中的参数和方法的形参绑定

7. @RequestBody 用于post方法请求体中数据的解析，主要用在content-type是JSON的请求上。
8. @PathVariable 用于把路径参数和方法形参绑定。

9. @Value的作用<https://blog.csdn.net/qq_43720551/article/details/132679766?ops_request_misc=&request_id=&biz_id=102&utm_term=%20@Value(%22classpath:autoAssigne&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-132679766.142^v101^pc_search_result_base1&spm=1018.2226.3001.4187：>

  (1). 把外部配置文件中键值对的值注入到spring管理的对象中，赋值给某个field。==>spring怎么知道要去那个文件中查找${servlet.single.max-file-size} ==> spring boot通过预定义的属性加载顺序来查找@Value注解中指定的属性值，依次查找命令行参数、Java系统属性、操作系统环境变量、application.properties或application.yml文件、@PropertySource注解指定的属性文件，后面的属性会覆盖前面的属性。

```java
    @Value("${servlet.single.max-file-size}")
    private String maxFileSize;
```

  (2). 能注入外部文件, 通过file:或classpath:前缀来指定外部文件的路径，赋值给Resource类型，进而访问外部文件的内容

  ```java
    @Value("file:/path/to/config.properties")
    private Resource configFile;

    @Value("classpath:autoAssignedMenu.json")
    private Resource resource;

    @EventListener
    public void init(ApplicationReadyEvent event) throws IOException {
        roleProvider.reloadRoleResources();
        dictProvider.reloadDict();
        menuProvider.initAutoAssignedMenus(IOUtils.toString(resource.getInputStream(), Charsets.UTF_8));
    }
  ```

10. @ConfigurationProperties跟@Value一样也是用来注入外部配置文件中的属性，但是@Value一次只能注入一个，@ConfigurationProperties可以批量的把外部属性注入到bean对象的属性里。

```java
// bean对象如下
package com.itheima.mybatisDemo.utils;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "security.jwt")
public class properties {
    private String secret;
    private String tokenDuration;
}

```

application.properties中内容如下

```
security.jwt.secret = secret1234567890qwertyuiopasdfghg
security.jwt.tokenDuration=86400
```
