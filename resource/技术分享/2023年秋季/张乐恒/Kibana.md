# 一、简介	

​	我们常说的ELK技术栈，包括Elasticsearch、Logstash和Kibana这三个技术，这三个技术的组合常用于大数据领域。其中，Logstash担任控制层的角色，负责搜集和过滤数据。Elasticsearch担任数据持久层的角色，负责储存数据。今天介绍一下担当视图层角色的Kibana。

​	它拥有各种维度的查询和分析，并使用图形化的界面展示存放在Elasticsearch中的数据。可以使用Kibana来搜索，查看存储在Elasticsearch索引中的数据并与之交互。

# 二、配置与启动

在官网下载解压后，配置监听端口号、es地址、索引名，默认情况下，kibana启动时将生成随机密钥，我们可以自己设置一下

```xml
xpack.reporting.encryptionKey: "zhangleheng"
xpack.security.encryptionKey: "123456"
xpack.encryptedSavedObjects.encryptionKey: "123456"
```

启动

```xml
./bin/kibana
```

这时候我们打开localhost:8080

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316124941303-817937361.png)

# 三、添加自定义索引

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125002324-1761768512.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125008549-1126155590.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125012923-171985027.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125016914-557679240.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125023366-1540814218.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125029588-742105904.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125035741-317517608.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125040327-1265830382.png)

# 四、控制台

控制台插件提供一个用户界面来和 Elasticsearch 的 REST API 交互。控制台的 editor ，用来编写提交给 Elasticsearch 的请求； response 面板，用来展示请求结果的响应。在页面顶部的文本框中输入 Elasticsearch 服务器的地址。默认地址是：“localhost:9200”。

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125117540-1728278009.png)

```xml
# 查看所有节点
GET _cat/nodes

# 查看book索引数据
GET book/_search
{
    "query": {
    "match": {
      "content": "666666"
    }
  }
}

# 添加一条数据
POST book/_doc 
{
  "page":8,
  "content": "1111111"
}

# 更新数据
PUT book/_doc/iSAz4XABrERdg9Ao0QZI
{
  "page":8,
  "content":"11111"
}

# 删除数据
POST book/_delete_by_query
{
  "query": {
    "match": {
      "page": 8
    }
  }
}

# 批量插入数据
POST book/_bulk
{ "index":{} }
{ "page":22 , "content": "Adversity, steeling will strengthen body"}
{ "index":{} }
{ "page":23 , "content": "Reading is to the mind, such as exercise is to the body."}
{ "index":{} }
{ "page":24 , "content": "Years make you old, anti-aging."}
{ "index":{} }
```

# 五、可视化

Kibana可视化控件基于 Elasticsearch 的查询。利用一系列的 Elasticsearch 查询聚合功能来提取和处理数据，再通过创建图表来呈现数据分布和趋势

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125152438-1029425675.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125156751-1132660304.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125207450-1245127647.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125211675-1397580860.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125215674-905614276.png)

我们可以添加一个仪表盘来做成更加丰富的图表

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125226243-447072527.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125232234-303602763.png)

![img](https://img2020.cnblogs.com/blog/662544/202003/662544-20200316125310123-346878926.png)