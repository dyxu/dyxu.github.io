---
layout: post
title: 几种NoSQL的使用误区【转载】
author: reprint
keywords: ["NoSQL"]
description:
categories:
  - 数据库设计
tags: ["NoSQL", "数据库"]

---

{% include JB/setup %}

**本文转载自[博客](http://www.justinablog.com/archives/982)**

最近经常看到关于NoSQL怎么用的讨论，加上自己也已经有四年多在正式项目中使用NoSQL数据库的经验，写下来与大家分享。

## 1 通常不适合Java程序

NoSQL数据库最大的特点就是数据结构自由，不管是键值对还是JSON格式。使用动态语言，Ruby、Python或JavaScript，程序员可以非常流畅直接地书写JSON等数据：

```python
// JSON in Python
action = {
    "_index": "tickets-index",
    "_id": j + 1,
    "_source": {
        "flight":df[1][j],
        "price":float(df[2][j]),
        "timestamp": datetime.now()}
}
```
在JavaScript中的JSON对象操作：

```javascript
// JSON in JavaScript
var myJSONObject = {"bindings": [
        {"ircEvent": "PRIVMSG", "method": "newURI", "regex": "^http://.*"},
        {"ircEvent": "PRIVMSG", "method": "deleteURI", "regex": "^delete.*"},
        {"ircEvent": "PRIVMSG", "method": "randomURI", "regex": "^random.*"}
    ]
};

myJSONObject.bindings[0].method    // "newURI"
```

但在Java中，需要非常复杂的字符串操作或第三方库来完成JSON转换，最简单的写法也是这样：

```java
// JSON in Java
JSONObject myJSONObject  = new JSONObject();
JSONArray bindings  = new JSONArray();
JSONObject event1  = new JSONObject();
JSONObject event2  = new JSONObject();
JSONObject event3 = new JSONObject();

event1.put("ircEvent", "PRIVMSG");
event1.put("method", "newURI");
// event2, event3
bindings.add(event1);
bindings.add(event2);
bindings.add(event3);
myJSONObject.put("bindings", bindings);
然后Java程序员会告诉你：

public static final String DO_NOT_HARD_CODE_THE_KEY = "bindings";
myJSONObject.put(DO_NOT_HARD_CODE_THE_KEY, bindings);
```

## 2 没事别把SQL数据库往NoSQL上迁

既然SQL数据库都能正常工作，那为什么还要向NoSQL数据库迁移呢？这种做法就像要把所有线上Java程序全部用Python重写一样傻。

但有几种场景，相信你曾经使用SQL数据库时会感到非常别扭，可以考虑把这部分数据库迁往NoSQL数据库：

如果你使用SQL数据库来存储序列化对象，请改用NoSQL来存储：MongoDB这样的Document数据库是首选；
如果你使用SQL数据库来存储二进制流（图片、附件等），请改用NoSQL来存储：HDFS、S3甚至MongoDB的GridFS这样的File System很合适，不用再担心你的SQL数据库空间爆增后怎么分表的问题；
超过几十张表链接查询，只为取得一些字典数据及一些外键关联少数字段的数据：请毫不犹豫地使用Column based数据库如Cassandra、HBase，把你需要查询的结果数据直接设计在一个Column Family中，它们专门为解决多列查询问题存在，但你需要逆关系数据库设计范式：根据第一范式设计就对了；
一张不停扩展字段、包含许多Null值的表：月月改表改Mapping改Model不累么？换Document类的NoSQL数据库吧，省时省力又省心；
日志：这个场景就直接使用NoSQL数据库吧，起码不会和你的业务操作抢连接池资源，一不小心还会把你的业务表锁了。

## 3 列族数据库不适用作主数据存储

列族数据库面向特定的查询场景和特定的分析场景，比如你需要查询一个城市的所有用户和一个用户去过的所有城市，你是需要设计将用户作为Row Key设计一个数据存放结构，再将城市作为Row Key设计一个数据存放结构，分别支持两种快速查询的需求。对于数据分析，在有了具体的数据分析模型后，在列族数据库中建立Column Family再从数据源中导入数据，也是比较常见的做法。

因为列族数据库是针对特定需求而作出的设计结构，所以别指望它作为你的主要存储数据库，然后再满足你所有的数据应用需求。我记得FaceBook使用Cassandra就只是为了解决Inbox的搜索请求。

## 4 别把对象序列化到文件系统中

本来以为不会有这样傻的事，但我偏偏就遇到了：在一个系统中，JSON对象被转换为流格式，再存储到Amazon S3中。这个问题的关键在于，理解你要存储的东西到底是Raw Data还是Structured Data.如果你已经有了一个JSON对象，那就原汁原味地往文档数据库里扔吧，别再节外生枝了。

## 5 别问索引服务器该不该当成数据库

数据库就有索引，索引文件也可以开启原数据存储。在关系型数据库中有一个问题就是：“既然索引可以让查询速度变快，为什么我们不在每一列都建立索引呢？”如果我们有了一个每一列都可以索引又不影响写入速度还可以自由格式的存储，它是不是数据库还重要吗？（言外之意是elasticsearch可以当做数据库来使用）