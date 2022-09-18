---
layout: post
author: shopkeeper
categories: koa nodejs postgis
---

Postgresql库pg并不支持在浏览器环境中使用，因此无法在基于Webpack或Babel的Vue.js框架中使用。

## 基于Node.js的Koa框架

安装依赖

```node
npm install pg
```

使用：提供数据库相关参数并建立连接池；使用连接池进行连接并执行查询语句

```typescript
import pgsql from "pg";

const pgPool = new pgsql.Pool({
  host: "localhost",
  port: 5432,
  user: "postgres",
  password: "postgresql",
  database: "pg_routing_learning",
});

const routing = async () => {
  const connection = await pgPool.connect();
  var res = await connection.query(`SELECT * FROM pgr_dijkstra(   
    'SELECT id, source, target, cost, reverse_cost FROM edge_table',  
     2, 3)`);
  return res.rows;
};
```