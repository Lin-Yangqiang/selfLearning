# Swagger的简单使用

## 1 介绍

Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务(<https://swagger.io/>)。 它的主要作用是：

1. 使得前后端分离开发更加方便，有利于团队协作

2. 接口的文档在线自动生成，降低后端开发人员编写接口文档的负担

3. 功能测试 

   Spring已经将Swagger纳入自身的标准，建立了Spring-swagger项目，现在叫Springfox。通过在项目中引入Springfox ，即可非常简单快捷的使用Swagger。

### knife4j

knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案,前身是swagger-bootstrap-ui,取名kni4j是希望它能像一把匕首一样小巧,轻量,并且功能强悍!

目前，一般都使用knife4j框架。



## 2 使用步骤

1. 导入 knife4j 的maven坐标

   在pom.xml中添加依赖

   ```xml
   <dependency>
      <groupId>com.github.xiaoymin</groupId>
      <artifactId>knife4j-spring-boot-starter</artifactId>
   </dependency>
   ```

2. 在配置类中加入 knife4j 相关配置

   `WebMvcConfiguration.java`

   ```java
   /**
    * 通过knife4j生成接口文档
    * @return
    */
   @Bean
   public Docket docket() {
       log.info("准备生成接口文档...");
       ApiInfo apiInfo = new ApiInfoBuilder()
               .title("苍穹外卖项目接口文档")
               .version("2.0")
               .description("苍穹外卖项目接口文档")
               .build();
       Docket docket = new Docket(DocumentationType.SWAGGER_2)
               .apiInfo(apiInfo)   // 使用 apiInfo 方法设置 API 的基本信息
               .select()
               .apis(RequestHandlerSelectors.basePackage("com.sky.controller"))
               .paths(PathSelectors.any())  // 使用 select 方法选择需要生成文档的API接口
                                            // 常用的选择器有 RequestHandlerSelectors
                                            // 和 PathSelectors
               .build();
       return docket;
   }
   ```

   这段代码主要做了如下的事情：

   1. 构建API信息：使用 `ApiInfoBuilder` 构建 API 信息，包括标题、版本和描述。
   2. 创建 Docket 对象：使用 `Docket` 对象配置 Swagger 2 文档，指定 API 信息、选择器和路径。
   3. 返回 Docket 对象：返回配置好的 `Docket` 对象，用于生成接口文档。

   > [!IMPORTANT]
   >
   > **什么是 Docket 对象？**
   >
   > `Docket` 是 Swagger 2 中的一个核心类，用于配置和生成 API 文档。它主要用于以下几点：
   >
   > 1. **配置API信息：**通过`Docket`对象，可以设置 API 的基本信息，如标题、版本、描述等。
   > 2. **选择API接口**：可以指定哪些控制器和方法需要生成文档，通常通过 `RequestHandlerSelectors` 和 `PathSelectors` 来选择。
   > 3. **自定义文档配置**：可以添加安全配置、全局参数、响应消息等，以满足不同的需求。

   

3. 设置静态资源映射，否则接口文档页面无法访问

   `WebMvcConfiguration.java`

   ```java
   /**
        * 设置静态资源映射
        * @param registry
   */
   protected void addResourceHandlers(ResourceHandlerRegistry registry) {
           registry.addResourceHandler("/doc.html").addResourceLocations("classpath:/META-INF/resources/");
           registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
   }
   ```

   这段代码主要功能如下：

   1. 将`/doc.html` 映射到 `classpath:/META-INF/resources/` 目录
   2. 将 `/webjars/**` 映射到 `classpath:/META-INF/resources/webjars/` 目录

   这段代码中，静态资源映射主要目的是为了使应用程序能够正确访问和处理特定路径下的静态资源文件。具体来说：

   1. 访问文档页面：

      将 `/doc.html` 映射到 `classpath:/META-INF/resources/` 目录，使得用户可以通过访问 `/doc.html` 来查看位于该目录下的文档页面。

   2. 访问 WebJars 资源：

      将 `/webjars/**` 映射到 `classpath:/META-INF/resources/webjars/` 目录，使得用户可以通过访问 `/webjars/**` 来获取位于该目录下的 WebJars 资源文件。WebJars 是一种将前端库（如 jQuery、Bootstrap 等）打包成 JAR 文件的方式，方便在 Java 应用中管理和使用这些前端资源。

   

4. 访问测试

   接口文档访问路径为 http://ip:port/doc.html ---> http://localhost:8080/doc.html

   <img src="assets/image-20221107173033632.png" alt="image-20221107173033632" style="zoom:50%;" />

   接口测试:测试登录功能

   <img src="assets/image-20221107173137506.png" alt="image-20221107173137506" style="zoom:50%;" />

**思考：**通过 Swagger 就可以生成接口文档，那么我们就不需要 Yapi 了？

1、Yapi 是设计阶段使用的工具，管理和维护接口

2、Swagger 在开发阶段使用的框架，帮助后端开发人员做后端的接口测试



## 3 常用注解

通过注解可以控制生成的接口文档，使接口文档拥有更好的可读性，常用注解如下：

| **注解**            | **说明**                                               |
| ------------------- | ------------------------------------------------------ |
| `@Api`              | 用在类上，例如Controller，表示对类的说明               |
| `@ApiModel`         | 用在类上，例如entity、DTO、VO                          |
| `@ApiModelProperty` | 用在属性上，描述属性信息                               |
| `@ApiOperation`     | 用在方法上，例如Controller的方法，说明方法的用途、作用 |

接下来，使用上述注解，生成可读性更好的接口文档

在sky-pojo模块中

EmployeeLoginDTO.java

```java
package com.sky.dto;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.io.Serializable;

@Data
@ApiModel(description = "员工登录时传递的数据模型")
public class EmployeeLoginDTO implements Serializable {

    @ApiModelProperty("用户名")
    private String username;

    @ApiModelProperty("密码")
    private String password;

}

```

EmployeeLoginVo.java

```java
package com.sky.vo;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@ApiModel(description = "员工登录返回的数据格式")
public class EmployeeLoginVO implements Serializable {

    @ApiModelProperty("主键值")
    private Long id;

    @ApiModelProperty("用户名")
    private String userName;

    @ApiModelProperty("姓名")
    private String name;

    @ApiModelProperty("jwt令牌")
    private String token;

}
```

在sky-server模块中

EmployeeController.java

```java
package com.sky.controller.admin;

import com.sky.constant.JwtClaimsConstant;
import com.sky.dto.EmployeeLoginDTO;
import com.sky.entity.Employee;
import com.sky.properties.JwtProperties;
import com.sky.result.Result;
import com.sky.service.EmployeeService;
import com.sky.utils.JwtUtil;
import com.sky.vo.EmployeeLoginVO;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

/**
 * 员工管理
 */
@RestController
@RequestMapping("/admin/employee")
@Slf4j
@Api(tags = "员工相关接口")
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;
    @Autowired
    private JwtProperties jwtProperties;

    /**
     * 登录
     *
     * @param employeeLoginDTO
     * @return
     */
    @PostMapping("/login")
    @ApiOperation(value = "员工登录")
    public Result<EmployeeLoginVO> login(@RequestBody EmployeeLoginDTO employeeLoginDTO) 	{
        //..............

        
    }

    /**
     * 退出
     *
     * @return
     */
    @PostMapping("/logout")
    @ApiOperation("员工退出")
    public Result<String> logout() {
        return Result.success();
    }

}

```

启动服务：访问http://localhost:8080/doc.html

<img src="assets/image-20221107175649468.png" alt="image-20221107175649468" style="zoom:50%;" />

