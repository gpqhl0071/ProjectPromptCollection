# 后台管理系统框架详细描述

## 一、技术栈

### 1. 基础环境
- **Java版本**: 11
- **构建工具**: Maven 3.8+
- **项目结构**: Spring Boot 标准目录结构

### 2. 核心框架
- **Spring Boot**: 2.7.17
- **Spring Security**: 与Spring Boot版本配套 (2.7.17)
- **Spring Data JPA**: 与Spring Boot版本配套 (2.7.17)
- **JWT**: jjwt 0.9.1，用于身份验证
- **MySQL**: 8.0.33 驱动
- **Redis**: Spring Boot Data Redis

### 3. 辅助工具及库
- **Lombok**: 简化代码编写
- **Springdoc-openapi**: 1.6.15，API文档生成
- **Knife4j**: 4.3.0，提供增强的Swagger UI界面
- **Apache POI**: 5.2.3，用于Excel操作
- **Bean Validation**: 数据验证框架

## 二、项目规范

### 1. 包结构设计
```
com.bbx.admin
  ├── config         # 配置类
  ├── constant       # 常量定义
  ├── controller     # 控制器
  ├── dto            # 数据传输对象
  ├── entity         # 实体类
  ├── exception      # 异常处理
  ├── model          # 模型
  │   ├── request    # 请求模型
  │   └── response   # 响应模型
  ├── repository     # 数据访问层
  ├── security       # 安全相关
  ├── service        # 业务逻辑层
  └── util           # 工具类
```

### 2. 请求和响应规范

#### 请求规范
- 使用RESTful风格API
- URL前缀统一为`/api/`
- 接口按模块分组，如`/api/auth/`、`/api/user/`等
- 请求参数校验使用`@Valid`和`javax.validation`注解
- 文件上传大小限制为20MB

#### 响应规范
- 统一使用`Result<T>`封装响应数据
- 响应状态码：0表示成功，其他表示失败
- JSON格式，使用UTF-8编码
- 分页查询使用`PageResult<T>`统一封装

```json
{
  "code": 0,            // 状态码：0-成功，其他-失败
  "message": "操作成功",  // 响应消息
  "data": {}            // 响应数据，可为null
}
```

### 3. 异常处理机制
- 使用`@RestControllerAdvice`全局异常处理
- 业务异常使用`BusinessException`
- 参数验证异常统一捕获并格式化返回
- 认证异常和权限异常使用专门的处理方法
- 所有异常响应使用`Result.fail()`或`Result.error()`方法

### 4. 安全框架设计
- 基于Spring Security + JWT的认证授权体系
- 登录流程：用户名/密码 + 图形验证码认证
- 无状态会话管理，使用JWT令牌
- 角色-权限多对多映射，细粒度权限控制
- API接口使用`@PreAuthorize`注解进行权限检查
- JWT令牌有效期：86400秒（可配置）

### 5. 日志处理
- 使用SLF4J + Logback日志框架
- 日志级别：生产环境INFO，开发环境DEBUG
- 日志存储路径：./logs
- 使用`@Slf4j`注解自动注入Logger对象
- 错误日志记录详细堆栈信息

### 6. 文档规范
- 使用OpenAPI 3.0（Springdoc）生成API文档
- 使用Knife4j增强文档展示
- 每个API接口必须添加`@Operation`注解说明
- 接口参数使用`@Parameter`注解说明
- 接口返回值使用`@ApiResponse`注解说明

## 三、核心业务模块

### 1. 用户认证模块
- **主要功能**: 登录、登出、获取验证码、获取用户信息
- **核心接口**: 
  - `/api/auth/login` - 用户登录
  - `/api/auth/captcha` - 获取图形验证码
  - `/api/auth/info` - 获取用户信息
  - `/api/auth/logout` - 用户登出

### 2. 用户管理模块
- **主要功能**: 用户CRUD，角色分配
- **核心接口**: 
  - `/api/admin/user/**` - 后台系统用户管理
  - `/api/bbx/user/**` - 前台系统用户管理

### 3. 角色权限模块
- **主要功能**: 角色CRUD，权限管理
- **核心接口**: 
  - `/api/admin/role/**` - 角色管理接口

### 4. 文件管理模块
- **主要功能**: 文件上传、访问
- **核心接口**: 
  - `/api/file/upload/**` - 文件上传
  - `/api/file/image/**` - 图片访问
- **配置参数**:
  - 上传路径: ./uploaded-images
  - 允许的扩展名: jpg,jpeg,png,gif

### 5. 内容管理模块
- **主要功能**: 文章、Banner管理
- **核心接口**: 
  - `/api/bbx/article/**` - 文章管理
  - `/api/bbx/banner/**` - Banner管理

## 四、配置详解

### 1. 数据库配置
```properties
spring.datasource.url=jdbc:mysql://[数据库地址]:[端口]/bbx?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
spring.datasource.username=[用户名]
spring.datasource.password=[密码]
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.format_sql=true
```

### 2. 安全配置
```properties
jwt.secret=[签名密钥]
jwt.expiration=86400

admin.security.remember-me-token-validity-seconds=604800
admin.security.ignored-urls=/api/auth/**,/v3/api-docs/**,/swagger-ui/**,/swagger-resources/**,/doc.html,/webjars/**
admin.security.default-password=admin@123
```

### 3. Redis配置
```properties
spring.redis.host=[Redis地址]
spring.redis.port=6379
spring.redis.password=[密码]
spring.redis.database=1
spring.redis.timeout=10000ms
```

### 4. 文件上传配置
```properties
spring.servlet.multipart.max-file-size=20MB
spring.servlet.multipart.max-request-size=20MB
file.upload.path=./uploaded-images
file.access.path=/uploaded-images
file.allowed.extensions=jpg,jpeg,png,gif
file.base.url=${server.servlet.context-path:}/api/file/image
```

### 5. Swagger文档配置
```properties
springdoc.api-docs.enabled=true
springdoc.swagger-ui.enabled=true
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.api-docs.path=/v3/api-docs
knife4j.enable=true
knife4j.setting.language=ZH_CN
```

## 五、开发指南

### 1. 创建新控制器
1. 继承基础控制器或直接创建新控制器
2. 使用`@RestController`和`@RequestMapping("/api/模块名")`注解
3. 添加`@Tag`注解说明控制器功能
4. 每个方法添加`@Operation`和`@ApiResponse`注解

### 2. 异常处理
1. 业务逻辑异常使用`BusinessException`
2. 参数校验使用`@Valid`注解和验证约束

### 3. 权限管理
1. 控制器方法添加`@PreAuthorize`注解
2. 权限格式为`hasAuthority('权限编码')`
3. 可使用Spring EL表达式组合权限

### 4. 数据访问
1. 创建实体类使用`@Entity`注解
2. 创建Repository继承`JpaRepository`
3. 服务层使用`@Service`和`@Transactional`注解

## 六、部署说明

### 1. 环境要求
- JDK 11+
- MySQL 8.0+
- Redis 5.0+
- Maven 3.8+

### 2. 打包命令
```bash
mvn clean package -Dmaven.test.skip=true
```

### 3. 运行命令
```bash
java -jar bbx-admin-open-api-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
``` 