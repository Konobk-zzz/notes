# Mybatis

## 多个包下面的mapper只扫描了一个包

```yml
mybatis-plus:
  mapper-locations: classpath*:mapper/*Mapper.xml
```

解决办法是使用`classpath*`

**原理 （待填坑）**