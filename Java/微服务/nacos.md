##  

--- #################### 注册中心 + 配置中心相关配置 ####################
```yml
spring:
  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        ## system-server 服务
        - id: system-admin-api # 路由的编号
          uri: grayLb://xsf-system
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/admin-api/system/**
          filters:
            - RewritePath=/admin-api/system/v3/api-docs, /v3/api-docs # 配置，保证转发到 /v3/api-docs
        - id: system-app-api # 路由的编号
          uri: grayLb://xsf-system
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/app-api/system/**
          filters:
            - RewritePath=/app-api/system/v3/api-docs, /v3/api-docs
        ## infra-server 服务
        - id: infra-admin-api # 路由的编号
          uri: grayLb://xsf-infra
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/admin-api/infra/**
          filters:
            - RewritePath=/admin-api/infra/v3/api-docs, /v3/api-docs
        - id: infra-app-api # 路由的编号
          uri: grayLb://xsf-infra
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/app-api/infra/**
          filters:
            - RewritePath=/app-api/infra/v3/api-docs, /v3/api-docs
        - id: infra-spring-boot-admin # 路由的编号（Spring Boot Admin）
          uri: grayLb://xsf-infra
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/admin/**
        - id: infra-websocket # 路由的编号（WebSocket）
          uri: grayLb://xsf-infra
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/infra/ws/**
        ## basic-server 服务
        - id: basic-admin-api # 路由的编号
          uri: lb://xsf-basic
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/admin-api/basic/**
          filters:
            # - RewritePath=/admin-api/basic/(?<remaining>.*)$, /$\{remaining} # 转发其他请求到后端服务
            # - RewritePath=/admin-api/basic/v3/api-docs, /v3/api-docs # 特别处理Swagger文档请求
        #     - RewritePath=/admin-api/basic/(.*), /$1  # 将 /admin-api/basic/ 后面的部分直接转发给后端服务
          # filters:
            - RewritePath=/admin-api/basic/v3/api-docs, /v3/api-docs # 配置，保证转发到 /v3/api-docs
        - id: basic-app-api # 路由的编号
          uri: lb://xsf-basic
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/app-api/basic/**
          filters:
            # - RewritePath=/app-api/basic/(?<remaining>.*)$, /$\{remaining} # 转发其他请求到后端服务
            # - RewritePath=/app-api/basic/v3/api-docs, /v3/api-docs # 特别处理Swagger文档请求
        #     - RewritePath=/app-api/basic/(.*), /$1  # 将 /app-api/basic/ 后面的部分直接转发给后端服务
          # filters:
            - RewritePath=/app-api/basic/v3/api-docs, /v3/api-docs # 配置，保证转发到 /v3/api-docs
        - id: basic-websocket # 路由的编号（WebSocket）
          uri: lb://xsf-basic
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/admin-api/basic/**
      x-forwarded:
        prefix-enabled: false # 避免 Swagger 重复带上额外的 /admin-api/system 前缀


logging:
  file:
    name: ${user.home}/logs/${spring.application.name}.log # 日志文件名，全路径

knife4j:
  # 聚合 Swagger 文档，参考 https://doc.xiaominfo.com/docs/action/springcloud-gateway 文档
  gateway:
    enabled: true
    routes:
      - name: xsf-system
        service-name: xsf-system
        url: /admin-api/system/v3/api-docs
      - name: xsf-infra
        service-name: xsf-infra
        url: /admin-api/infra/v3/api-docs
      - name: xsf-basic
        service-name: xsf-basic
        url: /admin-api/basic/v3/api-docs

```

