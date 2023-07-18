---
title: 在C++下借助hiredis使用redis数据库
author: daz
pubDatetime: 2023-04-16T12:48:04Z
featured: false
draft: false
tags:
  - database
ogImage: ""
description: "redis && hiredis"
---

![版权头](https://img-blog.csdnimg.cn/img_convert/54e60afdf2764a07539da3136f3ce3e4.png)
本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。

---

[迁移博客]

# 在 C++下借助 hiredis 使用 redis 数据库

## 简介

- 本文为我在编写 C++ 聊天室项目时使用 redis 的经验之谈,主要讲解如何使用`C++`去调用 redis 数据库,并将其封装为一个类,方便程序随时调用
- 因为项目并没有使用到 redis 的订阅发布模式,所以本文所提及的均为 redis 的键值命令,大概如下:
  - key
  - string
  - hash
  - list
  - set
- 调用 redis 的方法是借助 hiredis 接口使用 redis 命令进操作

## 关于 hiredis

- `hiredis`是 redis 数据库的简约`C客户端库`，是 redis 官方的 C 语言客户端，支持所有命令(command set)，管道(pipelining)，时间驱动编程(event driven programming)
- `hiredis项目地址`: github 地址：https://github.com/redis/hiredis

## 编写为 C++类

### 头文件

```cpp
#include <hiredis/hiredis.h>

#include <iostream>
#include <list>
#include <set>
#include <sstream>
#include <string>
#include <vector>
#include <cstring>
```

### 类的主要结构

```cpp
class Redis
{
public:
  // redis的构造函数
  Redis() {
    m_redis = NULL;
    init();
  }
  // redis的析构函数
  ~Redis() {
    if (m_redis != NULL) {
      redisFree(m_redis);
      std::cout << "redis存储完毕" << std::endl;
    }
  }

  bool setString(std::string key, std::string value)
  {
    redisReply *reply;
    bool result = false;
    reply = (redisReply *)redisCommand(m_redis, "SET %s %s", key.c_str(),
                                       value.c_str());
    if (reply == NULL) {
      redisFree(m_redis);
      m_redis = NULL;
      result = false;
      std::cout << "set string faild" << __LINE__ << std::endl;
      return result;
    } else if (strcmp(reply->str, "OK") == 0) {
      result = true;
    }

    freeReplyObject(reply);

    return result;
  }

  std::string getString(std::string key)
  {
    // GET命令
  }

  bool delKey(std::string key)
  {
    // DEL命令
  }

  bool setVector(std::string key, std::vector<int> value)
  {
    // RPUSH命令
  }

  std::vector<int> getVector(std::string key)
  {
    // LRANGE命令
  }

  bool setHash(std::string key, std::string field, std::string value)
  {
    // HSET命令
  }

  int hashExist(std::string key, std::string field)
  {
    // HEXIST命令
  }

  std::string getHash(std::string key, std::string field)
  {
    // HGET命令
  }

  std::vector<std::string> getHashKey(std::string key)
  {
    // HKEYS命令
  }

  bool hashDel(std::string key, std::string field)
  {
    // HDEL命令
  }

  bool saddValue(std::string key, std::string value)
  {
    // SADD命令
  }

  int sismember(std::string key, std::string value)
  {
    // SISMEMBER命令
  }

  bool srmmember(std::string key, std::string value)
  {
    // SREM命令
  }

  int hlen(std::string key)
  {
    // HLEN命令
  }

  int slen(std::string key)
  {
    // SLEN命令
  }

  std::vector<std::string> smembers(std::string key)
  {
    // SMEMBERS命令
  }

  int lpush(std::string key, std::string value)
  {
    // LPUSH命令
  }

  int llen(std::string key)
  {
    // LLEN命令
  }

  redisReply **lrange(std::string key)
  {
    // LRANGE命令
  }

  redisReply **lrange(std::string key, int a, int b)
  {
    // LRANGE命令:指定范围
  }

  bool ltrim(std::string key)
  {
    // LTRIM命令
  }

  bool lrem(std::string key, std::string value) {
    // LREM命令
  }

  std::string lpop(std::string key) {
    // LPOP命令
  }

private:
  void init() {
    // 调用hiredis接口完成初始化
    struct timeval timeout = {1, 50000};  // 连接等待时间为1.5秒
    m_redis = redisConnectWithTimeout("127.0.0.1", 6379, timeout);
    if (m_redis->err) {
      // 自己编写的错误处理函数
      my_error("RedisTool : Connection error", __FILE__, __LINE__);
    } else {
      std::cout << "init redis success" << std::endl;
    }
  }

  redisContext *m_redis;
};
```

### 调用 demo

- 在我的聊天室项目中,`只有服务器连接了redis`,主要用来存储和获取用户数据

```cpp
#include <myredis.hpp>

int main()
{
  /*
  // 初始化自定义的redis类
  Redis myredis;

  // 查询当前用户名是否已经注册
  int flag = myredis.hashExist(UserMap, json["name"]);
  if(flag) {
    std::cout << "用户名已注册" << std::endl;
  }else {
    std::cout << "用户名未注册" << std::endl;
  }
  */

  int hashExist(std::string key, std::string field) {  // HEXISTS
    redisReply *reply;
    reply = (redisReply *)redisCommand(m_redis, "HEXISTS %s %s", key.c_str(),
                                       field.c_str());
    if (reply == NULL) {
      redisFree(m_redis);
      m_redis = NULL;
      return -1;
    }
    return reply->integer;
  }
}
```

## 完整代码

- 完整的代码在我的 Github 仓库,附带有简单注释
- 地址: https://github.com/Daz-3ux/tasks/blob/master/chat/include/REDIS.hpp
