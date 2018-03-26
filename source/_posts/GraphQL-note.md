---
title: GraphQL笔记
date: 2018-01-14 20:53:49
categories: 前端开发
tags: [GraphQL, What do I learn]
---

# What is GraphQL
> new API standard. 
> enables declaratative data fetching
简单来说，GraphQL是用于API的查询语言，与RESTFul一样，是一种查询规范。

# GraphQL vs REST
在漫长的岁月中，REST已经成为了WEB API设计标准了。它提供了一些 `stateless servers`和`structured access to resources`的好的想法，但是也存在着以下这些问题：
1. 接口灵活性差。返回的数据可能是无用或者多余的；或者一份数据需要调用多个接口才能拿到。
2. 操作流程繁琐。先构造HTTP，再进行接受，解析等等
3. 当功能需要拓展的时候，也意味着需要增加新的接口。

GraphQL是声明式数据获取。
1. 更能适应不同的前端框架以及客户端平台
2. 更快的开发速度
3. 可用于任意语言、任意平台

但GraphQL也面临着一些安全问题：
1. 因为一次查询可以包含很多数据操作，当数据操作过多时，也会造成过大的查询压力。
2. 查询格式会暴露API的结构，但可以通过持久化查询操作来解决，具体可见：[持久化查询](https://medium.com/m/global-identity?redirectUrl=https://dev-blog.apollodata.com/persisted-graphql-queries-with-apollo-client-119fd7e6bba5)

...待续