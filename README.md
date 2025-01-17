
## 1\. 简介


map是我们日常开发中常会的集合类之一, 但是我们除了常用的`get`和`put`之外,其他的方法好像很少会用到,接下来我们就介绍一下几个经常被忽略但又很好用的方法.


## 2\. Quick Start


### 2\.1 数据准备


创建一个map对象, 并声明几个用于测试的user对象



```
Map hashMap = Maps.newHashMap();
User zhangsan = new User(1, "张三");
User lisi = new User(2, "李四");
User zhangtieniu = new User(3, "张铁牛");

```

### 2\.2 重温put



```
// hashmap put (添加/更新元素)
@Test
public void put() {
    User test = hashMap.put(null, null);
    User user = hashMap.put(1, null);
    User user1 = hashMap.put(1, zhangsan);
    User user2 = hashMap.put(1, lisi);
    User user3 = hashMap.put(null, zhangsan);
    User user4 = hashMap.put(null, lisi);
    User user5 = hashMap.get(null);
    log.info("map: {}", hashMap);
    log.info("user: {}, user1: {}, user2: {}, user3: {}, user4: {}, user5: {}", user, user1, user2, user3, user4, user5);
    //map: {null=User(id=2, name=李四), 1=User(id=2, name=李四)}
    //user: null, user1: null, user2: User(id=1, name=张三), user3: null, user4: User(id=1, name=张三), user5: User(id=2, name=李四)
}

```

1. key和value可以为null (hashmap 和 linkedhashmap)
2. 使用null可以正常的覆盖和获取元素
3. put可以直接新增or覆盖已有的元素
4. put方法返回对应key的oldValue，如果没有oldValue则返回null


### 2\.3 getOrDefault



```
// getOrDefault(Object key, V defaultValue) (获取/返回默认值)
@Test
public void getOrDefault() {
    hashMap.put(1, zhangsan);
    hashMap.put(2, null);
    final User user1 = hashMap.get(1);
    final User user2 = hashMap.getOrDefault(2, lisi);
    final User user3 = hashMap.getOrDefault(3, zhangtieniu);
    log.info("map: {}", hashMap);
    //map: {1=User(id=1, name=张三), 2=null}
    log.info("user1: {}, user2: {}, user3: {}", user1, user2, user3);
    //user1: User(id=1, name=张三), user2: null, user3: User(id=3, name=张铁牛)
}

```

1. 当map中没有对应的key时, 返回对应的defaultValue


注意: 如果map中存在对应的key, 但是对应的`value == null`时, 返回的是null, 而不是defaultValue


源码如下:



```
default V getOrDefault(Object key, V defaultValue) {
    V v;
    return (((v = get(key)) != null) || containsKey(key))
        ? v
        : defaultValue;
}

```

### 2\.4 putIfAbsent



```
// putIfAbsent(K key, V value) (如果不存在则添加)
@Test
public void putIfAbsent() {
    hashMap.put(1, null);
    User user1 = hashMap.putIfAbsent(1, zhangsan);
    User user2 = hashMap.putIfAbsent(2, lisi);
    User user3 = hashMap.putIfAbsent(2, zhangtieniu);
    log.info("map: {}", hashMap);
    log.info("user1: {}, user2: {}, user3: {}", user1, user2, user3);
    //map: {1=User(id=1, name=张三), 2=User(id=2, name=李四)}
    //user1: null, user2: null, user3: User(id=2, name=李四)
}

```

1. 如果指定的key对应的value不为null时(`oldValue != null`) : 不覆盖 \& 返回oldValue
2. 当指定key的value不存在时(`oldValue == null`) : 添加元素 \& 返回oldValue



> 可以理解为 当指定key的value不存在时, 才去put, 否则不添加


源码如下:



```
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }

    return v;
}

```

### 2\.5 compute



```
// compute(K key, BiFunction super K, ? super V, ? extends V remappingFunction) (计算)
@Test
public void compute() {
    hashMap.put(1, zhangsan);
    User user1 = hashMap.compute(1, (k, oldValue) -> lisi);
    log.info("map: {}, user1: {}", hashMap, user1);
    //map: {1=User(id=2, name=李四)}, user1: User(id=2, name=李四)
    User user2 = hashMap.compute(1, (k, oldValue) -> null);
    log.info("map: {}, user2: {}", hashMap, user2);
    //map: {}, user2: null
}

```

1. `remappingFunction返回值 != null` : 覆盖oldValue \& 返回newValue
2. `remappingFunction返回值 == null` : 删除对应元素 \& 返回null



> 可以理解为 使用remappingFunction的返回值覆盖对应key的旧值, 当remappingFunction返回值为null时, 会直接将当前元素移除掉


源码如下:



```
default V compute(K key,
        BiFunction <span class="hljs-built_in"super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue = get(key);

    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue == null) {
        // delete mapping
        if (oldValue != null || containsKey(key)) {
            // something to remove
            remove(key);
            return null;
        } else {
            // nothing to do. Leave things as they were.
            return null;
        }
    } else {
        // add or replace old mapping
        put(key, newValue);
        return newValue;
    }
}

```

### 2\.6 computeIfAbsent



```
// computeIfAbsent(K key, Function super K, ? extends V mappingFunction) (不存在则计算)
@Test
public void computeIfAbsent() {
    User user = hashMap.computeIfAbsent(1, k -> zhangsan);
    User user1 = hashMap.computeIfAbsent(1, k -> lisi);
    User user2 = hashMap.computeIfAbsent(2, k -> null);
    log.info("map: {}, user：{}, user1: {}, user2:{}", hashMap, user, user1, user2);
    //map: {1=User(id=1, name=张三)}, user：User(id=1, name=张三), user1: User(id=1, name=张三), user2:null
}

```

1. `oldValue != null` : 不覆盖 \& 返回oldValue
2. `oldValue == null && mappingFunction返回值 != null`: 添加元素 \& 返回newValue
3. `oldValue == null && mappingFunction返回值 == null`: 不覆盖 \& 返回null



> 可以理解为 当指定key的value不存在时, 才使用mappingFunction的返回值覆盖对应key的旧值, 如果key对应value存在或者mappingFunction的返回值为null时, 则不覆盖


源码如下:



```
default V computeIfAbsent(K key,
        Function <span class="hljs-built_in"super K, ? extends V> mappingFunction) {
    Objects.requireNonNull(mappingFunction);
    V v;
    if ((v = get(key)) == null) {
        V newValue;
        if ((newValue = mappingFunction.apply(key)) != null) {
            put(key, newValue);
            return newValue;
        }
    }

    return v;
}

```

### 2\.7 computeIfPresent



```
// computeIfPresent(K key, BiFunction super K, ? super V, ? extends V remappingFunction) (存在则计算)
@Test
public void computeIfPresent() {
    hashMap.put(1, zhangsan);
    User user1 = hashMap.computeIfPresent(1, (k,oldValue) -> lisi);
    User user2 = hashMap.computeIfPresent(3, (k,oldValue) -> zhangtieniu);
    log.info("map: {}, user1: {}， user2: {}", hashMap, user1, user2);
    //map: {1=User(id=2, name=李四)}, user1: User(id=2, name=李四)， user2: null
    User user3 = hashMap.computeIfPresent(1, (k,oldValue) -> null);
    log.info("map: {}, user3:{}", hashMap, user3);
    //map: {}, user3:null
}

```

1. `oldValue == null` : 不覆盖\&返回null
2. `oldValue != null && remappingFunction返回值 == null` : 移除元素\&返回null
3. `oldValue != null && remappingFunction返回值 != null` : 覆盖元素\&返回newValue



> 可以理解为 当key对应的value存在时, 才使用remappingFunction的返回值覆盖对应key的旧值, 如果key对应的value不存在或者remappingFunction的返回值为null时, 则不覆盖


源码如下:



```
default V computeIfPresent(K key,
        BiFunction <span class="hljs-built_in"super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue;
    if ((oldValue = get(key)) != null) {
        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue != null) {
            put(key, newValue);
            return newValue;
        } else {
            remove(key);
            return null;
        }
    } else {
        return null;
    }
}

```

### 2\.8 replace



```
// replace(K key, V value)
// replace(K key, V oldValue, V newValue) (替换)
@Test
public void replace() {
    hashMap.put(1, zhangsan);
    hashMap.put(2, lisi);
    hashMap.replace(1, zhangtieniu);
    hashMap.replace(2, null);
    hashMap.replace(3, zhangtieniu);
    hashMap.replace(2, null, zhangtieniu);
    log.info("map: {}", hashMap);
    //map: {1=User(id=3, name=张铁牛), 2=User(id=3, name=张铁牛)}
}

```

1. 替换指定key的value值
2. 可以将对应的value设置为null
3. 对应的key不存在时不会添加新元素
4. `replace(K key, V oldValue, V newValue)`方法多了一层判断, 当key对应的value与oldValue相等时, 才会替换newValue



> 可以理解为 替换指定key的value值


源码如下:



```
default V replace(K key, V value) {
    V curValue;
    if (((curValue = get(key)) != null) || containsKey(key)) {
        curValue = put(key, value);
    }
    return curValue;
}

default boolean replace(K key, V oldValue, V newValue) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, oldValue) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    put(key, newValue);
    return true;
}

```

### 2\.9 merge



```
// merge(K key, V value, BiFunction super V, ? super V, ? extends V remappingFunction) (融合)
@Test
public void merge() {
    hashMap.put(1, zhangsan);
    User user1 = hashMap.merge(1, lisi, (oldValue, defaultValue) -> zhangtieniu);
    User user2 = hashMap.merge(2, lisi, (oldValue, defaultValue) -> zhangtieniu);
    log.info("map: {}, user1: {}， user2: {}", hashMap, user1, user2);
    //map: {1=User(id=3, name=张铁牛), 2=User(id=2, name=李四)}, user1: User(id=3, name=张铁牛)， user2: User(id=2, name=李四)
}

```

1. `oldValue == null` : 使用传进来的 value 作为newValue, `oldValue != null` : 使用remappingFunction的返回值作为newValue


	* `newValue == null` : 移除元素 \& 返回newValue
	* `newValue != null` : 覆盖元素 \& 返回newValue



> 可以理解为 融合三个值 分别为:
> 
> 
> 1. key对应的value(oldValue)
> 2. merge方法的第二个参数value (可以理解为oldValue的defaultValue)
> 3. merge方法的第三个参数remappingFunction方法的返回值
> 
> 
> 融合逻辑为: 如果key对应的value不存在时, 使用merge方法的第二个参数value作为newValue, 如果key对应的value存在时,使用remappingFunction的返回值作为newValue, 如果newValue不为null则覆盖元素, 为null则移除元素


源码如下:



```
    default V merge(K key, V value,
            BiFunction <span class="hljs-built_in"super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
}

```

### 2\.10 foreach



```
// foreach java8 新增
@Test
public void foreach() {
    hashMap.put(1, zhangsan);
    hashMap.put(2, lisi);
    hashMap.forEach((key, value) -> log.info("key: {}, value: {}", key, value));
    //key: 1, value: User(id=1, name=张三)
    //key: 2, value: User(id=2, name=李四)
}

```

## 3\. 总结




| 方法名称 | 方法参数 | 方法描述 | 方法特点 |
| --- | --- | --- | --- |
| put | key, value | 添加元素 | hashmap/linkedhashmap: key, value 都可以为null |
| getOrDefault | key, defaultValue | 获取元素 | 当map中没有对应的key时, 返回defaultValue |
| putIfAbsent | key, value | 当不存在时添加元素 | 这里不存在指的是: key对应的旧值为null |
| compute | key, BiFunction remappingFunction | 重新计算key对应的value | 使用remappingFunction的返回值替换key的旧值 |
| computeIfAbsent | key, Function mappingFunction | 当不存在时计算 | 这里不存在指的是: key对应的旧值为null, 与putIfAbsent方法逻辑类似 |
| computeIfPresent | key, BiFunction remappingFunction | 当存在时计算 | 这里存在指的是: key对应的旧值!\=null |
| replace | key, value | 替换元素 | 替换指定key的value, 不会添加元素 |
| merge | key, value, BiFunction remappingFunction | 融合 | 融合key的旧值, 默认值, remappingFunction的返回值作为新值 |
| foreach | BiConsumer | 遍历 | java8新加的遍历方式 |


 本博客参考[milou云加速器cloud](https://www.huabeikeji.com)。转载请注明出处！
