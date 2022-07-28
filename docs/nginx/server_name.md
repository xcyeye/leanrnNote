---
date: 2022/1/24 14:35
---

# nginx中的server_name字段

该字段定义服务器的名称，可以是域名，ip，用于选择哪个server处理请求

nginx在选择哪个server处理请求的时候，会根据请求的host和`server_name`进行匹配，他们的优先级如下

1. host和`server_name`准确匹配

2. 以星号`*`开头的最长通配符名称

   > 如`*.example.com`

3. 以星号`*`结尾的最长统配符名称

   > 如`main.*`

4. 正则表达式



