# MybatisPlus 使用小结

## id回填

调save(Entity e)方法时，主键id已经更新到e中了，可以直接拿到。

## service注入其它mapper

- 注入mapper要使用@Resource注解；
- AService只能注入AMapper和BService，不能注入BMapper；

## updateById更新非null值为null

使用mybatis-plus中提供的updateById方法，想将表中某个字段原本不为null的值更新为null，更新没有成功。

### 原因

mybatis-plus FieldStrategy 有三种策略：

- IGNORED：0 忽略；
- NOT_NULL：1 非 NULL，默认策略；
- NOT_EMPTY：2 非空；

默认更新策略是NOT_NULL，即通过接口更新数据时数据为NULL值时将不更新进数据库。

### 解决办法

解决办法有以下3种。

- 配置文件设置；
- 字段单独设置；
- 使用UpdateWrapper；

### 配置文件设置

全局生效。

```properties
#properties文件格式：
mybatis-plus.global-config.db-config.field-strategy=ignored

#yml文件格式：
mybatis-plus:
  global-config:
  	#字段策略 0:"忽略判断",1:"非 NULL 判断",2:"非空判断"
    field-strategy: 0
```

### 字段单独设置

某个字段单独生效。

```java
/**
 * 下架时间
 */
@TableField(strategy = FieldStrategy.IGNORED)
private LocalDateTime offlineTime;

```

### 使用UpdateWrapper

更加灵活。

```java
 /**
  * update更新字段为null
  * @param id
  * @return
  */
 @Override
 public boolean updateArticleById(Integer id) {
     Article article = Optional.ofNullable(articleMapper.selectById(id)).orElseThrow(RuntimeException::new);
     LambdaUpdateWrapper<Article> updateWrapper = new LambdaUpdateWrapper<>();
     updateWrapper.set(Article::getOfflineTime,null);
     updateWrapper.set(Article::getContent,"try mybatis plus update null");
     updateWrapper.set(Article::getPublishTime,LocalDateTime.now().plusHours(8));
     updateWrapper.eq(Article::getId,article.getId());
     int i = articleMapper.update(article, updateWrapper);
     return i==1;
 }
```

### 参考链接

1. https://blog.csdn.net/hu_zhiting/article/details/105812985













