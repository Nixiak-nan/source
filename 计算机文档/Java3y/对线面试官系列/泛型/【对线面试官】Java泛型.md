![](https://tva1.sinaimg.cn/large/0081Kckwly1glugj4r2nqj30ku112qab.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glugmvnbkfj30ku1127a8.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glugrtdxruj30ku112gra.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glugrz5ucij30ku112q9k.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glugs3xfcyj30ku112n0t.jpg)

```java
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {

	List<T> findAll();

	List<T> findAll(Sort sort);

	List<T> findAllById(Iterable<ID> ids);

	<S extends T> List<S> saveAll(Iterable<S> entities);

	void flush();

	<S extends T> S saveAndFlush(S entity);

	void deleteInBatch(Iterable<T> entities);

	void deleteAllInBatch();

	T getOne(ID id);

	@Override
	<S extends T> List<S> findAll(Example<S> example);

	@Override
	<S extends T> List<S> findAll(Example<S> example, Sort sort);
}
```

![](https://tva1.sinaimg.cn/large/0081Kckwly1glugtz4oo2j30ku112dmd.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glugvt3g8vj30ku112jx7.jpg)

```java
// 传入 需要group by 和 sum 的字段名
public cacheMap(List<String> groupByKeys, List<String> sumValues) {
  this.groupByKeys = groupByKeys;
  this.sumValues = sumValues;
}

private void excute(T e) {
  
  // 从pojo 取出需要group by 的字段 list
  List<Object> key = buildPrimaryKey(e);
  
  // primaryMap 是存储结果的Map
  T value = primaryMap.get(key);
  
  // 如果从存储结果找到有相应记录
  if (value != null) {
    for (String elem : sumValues) {
      // 反射获取对应的字段，做累加处理
      Field field = getDeclaredField(elem, e);
      if (field.get(e) instanceof Integer) {
        field.set(value, (Integer) field.get(e) + (Integer) field.get(value));
      } else if (field.get(e) instanceof Long) {
        field.set(value, (Long) field.get(e) + (Long) field.get(value));
      } else {
        throw new RuntimeException("类型异常,请处理异常");
      }
    }
    
    // 处理时间记录
    Field field = getDeclaredField("updated", value);
    if (null != field) {
      field.set(value, DateTimeUtils.getCurrentTime());
    }
  } else {
    // group by 字段 第一次进来
    try {
      primaryMap.put(key, Tclone(e));
      createdMap.put(key, DateTimeUtils.getCurrentTime());
    }catch (Exception ex) {
      log.info("first put value error {}" , e);
    }
  }
}
```

![](https://tva1.sinaimg.cn/large/0081Kckwly1glugzi6kicj30ku11210a.jpg)



![](https://tva1.sinaimg.cn/large/0081Kckwly1gluh34tqcnj30ku112jwm.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gluhfiaea5j30ku112grv.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm3bbpx8dej30ku112wl0.jpg)

**文章以纯面试的角度去讲解，所以有很多的细节是未铺垫的。**

比如说反射和泛型基础，这些在【**Java3y**】都有过详细的基本教程甚至电子书，我就不再详述了。

欢迎关注我的微信公众号【**面试造火箭**】来聊聊Java面试

![](https://tva1.sinaimg.cn/large/0081Kckwly1gluioeg3f9j3076076glt.jpg)



