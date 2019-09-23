# ElasticSearch重建索引

基于scroll+bulk+索引别名实现零停机重建索引

一个field的设置后是不能被修改的，如果要修改，那么需要重新按新的mapping，建立一个index

实例操作  
![](/assets/58.png)

假如现在我要将createTime改为date类型，现在是字符串

![](/assets/60.png)

直接修改是不行的，因为此时createTime字段已经建立，又不能直接删除index重建  
![](/assets/59.png)

不停机修改方式：

1.对索引建立别名

```
PUT /order_index/_alias/bm_order_index/
```

![](/assets/61.png)

java代码里改为对别名的bm\_order\_index继续进行操作

java api里面操作方式:

（1）添加别名

```
client.admin().indices().prepareAliases().addAlias("order_index","bm_order_index");
```

（2）移除别名

```client.admin\(\).indices\(\).prepareAliases\(\).removeAlias\(&quot;order\_index&quot;,&quot;bm\_order\_index&quot;\);

```

（3）删除一个别名后再添加一个

```
client.admin().indices().prepareAliases().removeAlias("order_index","bm_order_index").addAlias("order_index_new","bm_order_index").execute().actionGet();
```

当别名添加完毕后，我们在删除，搜索，更新都可以直接使用：

SearchRequestBuilder search=client.prepareSearch\("my\_index"\);  
需要注意使用别名后，type类型的值不需要在填写，如果你填写了es是会抛异常的，因为它认为你这别名是一个新的索引，所以我们只写index name即可，es服务端知道它的类型。

2.新建新的索引order\_index\_new

![](/assets/62.png)

![](/assets/63.png)

3.使用scroll将数据一批批的查询出来

```
POST /order_index/order_type/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": [
    "_doc"
  ],
  "size": 1
}
```

![](/assets/64.png)

4.使用bulk将查询出来的数据批量导入新索引

```
POST /_bulk
{ "index":  { "_index": "order_index_new", "_type": "order_type", "_id": "2" }}
{ .... }
```

5.将bm\_order\_index 的alias切换到order\_index\_new上去  
POST /\_aliases  
{  
    "actions": \[  
        { "remove": { "index": "order\_index", "alias": "bm\_order\_index" }},  
        { "add":    { "index": "order\_index\_new", "alias": "bm\_order\_index" }}  
    \]  
}

这样就完成了

java api操作：

