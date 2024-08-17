[TOC]

---

# [No spring.config.import property has been defined](https://stackoverflow.com/questions/67507452/no-spring-config-import-property-has-been-defined)

- 问题记录：No spring.config.import property has been defined。这个是因为sping2021后的新问题，默认不导入bootstrap。

- 解决办法：
  
  - 添加pom，使得bootstrap.yml 生效
  
  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bootstrap</artifactId>
  </dependency>
  ```
  
  - 添加 spring.config.import 
    
    ```yml
    #spring:
    #  config:
    #    import: optional:nacos:127.0.0.1:8848:application-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}    
    ```


