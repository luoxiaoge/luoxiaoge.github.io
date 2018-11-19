1，mybatis中进行批量插入 

  在mybatis中时常进行批量插入，批量插入需要用到mybatis 的foreach 标签

首先来看一下foreach 标签几个元素的含义

item：表示集合中每一个元素进行迭代时的别名

index(下标从0开始):指 定一个名字，用于表示在迭代过程中，每次迭代到的位置

collection:

1,如果传入的是单参数且参数类型是一个List的时候，collection属性值为list

批量插入list：

```
@Test
public void testBatchInsert() throws  Exception{
   //Map<String,Object> map  = new HashMap<String, Object>();
   //map.put("specId","my heart will go on");
   List<Map<String,Object>> list  = new ArrayList<Map<String,Object>>();
   Map<String,Object> map1  = new HashMap<String, Object>();
   map1.put("specId","aaaa");
   map1.put("specValue",10);
   Map<String,Object> map2  = new HashMap<String, Object>();
   map2.put("specId","aaaa");
   map2.put("specValue",20);
   Map<String,Object> map3  = new HashMap<String, Object>();
   map3.put("specId","aaaa");
   map3.put("specValue",30);

   list.add(map1);
   list.add( map2);
   list.add(map3 );

   //map.put("specValue",list);
   // 批量插入 list
   bookDao.batchInsert(list);
   
}
```

sql:

```
<insert id="batchInsert" parameterType="Map"  useGeneratedKeys="true" keyProperty="id">
   INSERT INTO book(`name`,`number`) VALUES
   <foreach collection="list" item ="temp" separator=",">
      (
      #{temp.specId},
      #{temp.specValue}
      )
   </foreach>
</insert>
```

2,如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array

批量插入数组：

```
@Test
public void  testBatchInsertArrays(){
   Integer[]  arrays = new Integer[]{1,2,3};
   bookDao.batchInsertArray(arrays);
}
```

sql：

```
<insert id="batchInsertArray"  >
   INSERT INTO book(`name`,`number`) VALUES
   <foreach collection="array" item ="temp" index="index" separator=",">
      (
      "aaa",
      #{index} 
      )
   </foreach>
</insert>
```

3,如果传入的参数是多个的时候，我们就需要把它们封装成一个Map了  如果此时Map 中有一个list参数 这个时候collection 为 list 参数的key值

map中key 为 where 条件 ， value 为修改值时：<red>需要注意的事jdbcUrl 必须加上allowMultiQueries=true</red>

```
@Test
public void  testBatchUpdateBook(){
    Map<String,Object>  map = new HashMap<String, Object>();
    map.put("aaa",10);
    map.put("zhang",200);
    bookDao.batchUpdateBook(map);
}
```

sql:

```
<update id="batchUpdateBook" parameterType="Map">
   <foreach item="value" index="key" collection="Book.entrySet()" separator=";">
           UPDATE book SET
           number= ${value}
           WHERE name = #{key}
   </foreach>
</update>
```

open:表示该语句以什么开始

separator:表示在每次进行迭代之间以什么符号作为分隔 符

close:表示以什么结束



```
<!-- Gitalk 评论 start  -->
{% if site.gitalk.enable %}
<!-- Link Gitalk 的支持文件  -->
<link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
<script src="https://unpkg.com/gitalk@latest/dist/gitalk.min.js"></script>

<div id="gitalk-container"></div>
    <script type="text/javascript">
    var gitalk = new Gitalk({

    // gitalk的主要参数
		clientID: `868b7888c6dc35f2b67d`,
		clientSecret: `bc6b74517d62efd93370dc344ee7dfe05fae3d82`,
		repo: `luoxiaoge.github.io`,
		owner: 'luoxiaoge',
		admin: ['luoxiaoge'],
		id: 'window.location.pathname',
    
    });
    gitalk.render('gitalk-container');
</script>
{% endif %}
<!-- Gitalk end -->
```





