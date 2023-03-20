# 1、@SpringBootApplication包含那几个子类
- @SpringBootConfiguration：进入注解中看到了@Configuration注解，也就是配置类注解，主要是将实体注入到spring容器中。
- @EnableAutoConfiguration:实现自动导入的注解
- @ComponentScan:具体的扫描路径

#2、如果自定义