---
title: node.js + mssql 简易封装操作
date: 2019-07-05 17:53:43
tags:
feature: true
---

时间吧，总是这么凑巧，在我学习【node.js】还没几天，我的 Microsoft SQL Server Management Studio 18 就歇菜了，至于怎么歇菜的吧....它可能的意思就是想让我换电脑了... 所以为了解决问题，就写了这个小东西满足需求；
....咳咳咳....

回归正题，开始最简易的封装数据操作。

首先老样子,先安装：

**安装方法**

```
npm install mssql
```

**引入依赖**
```
///引入依赖
const mssql = require('mssql');
```

**配置Config**

{% asset_img 1412426-20190705172223422-1396451175.png This is an example image %}

其实这里的config，和后端程序配置的web.config基本是一个意思。（可忽略 0.0）

{% asset_img 1412426-20190705173120708-749956324.png This is an example image %}

code：
```
///引入依赖
const mssql = require('mssql');

//方法对象
const units = {
  sql: function (sql, callback) {
    ///连接池
    new mssql.ConnectionPool(units.config())
      .connect()
      .then(pool => {
        let ps = new mssql.PreparedStatement(pool);
        ps.prepare(sql, err => {
          if (err) {
            console.log(err);
            return;
          }
          ps.execute('', (err, result) => {
            if (err) {
              console.log(err);
              return;
            }
            ps.unprepare(err => {
              if (err) {
                console.log(err);
                callback(err, null);
                return;
              }
              callback(err, result);
            });
          });
        });
      }).catch(err => {
        console.log("Database Connection Failed! Bad Config:", err);
      });
  },
  /*
 * 默认config对象
 * @type {{user: string, password: string, server: string, database: string, pool: {min: number, idleTimeoutMillis: number}}}
 */
  config: function () {
    return {
      user: 'sa',                       //SQL Server 的登录名
      password: '123456',               //SQL Server 的登录密码
      server: 'localhost',              //SQL Server 的地址
      database: 'sale',                 //数据库名称
      port: 1433,                       //端口号，默认为1433
      pool: {
        min: 0,                         //连接池最小连接数，默认0
        max: 10,                        //连接池最大连接数，默认10
        idleTimeoutMillis: 3000         //设置关闭未使用连接的时间，单位ms默认30000
      },
      /*--其他属性--*/
      // connectionTimeout：             //连接timeout，单位ms 默认 15000
      // requestTimeout：                //请求timeout，单位ms默认15000
      // parseJSON：                     //将json数据集转化成json obj 
    }
  }
}

module.exports = units;
```

此上面这段代码就可以封装为一个命名为：helper.js(名字随意)。

然后就可以在其他的js里面来调用这个封装好的‘方法’：

```
const helper = require('./helper');
```

接下来就是写最基本的 参数化  批量：insert丶select丶update 丶delete ：

```
const helper = require('./helper');
/*
 * 查询所有
 * @param tableName
 * @param result
 */
helper.sql('select * from dbo.tableName where 1 = 1', function (err, result) {
    if (err) {
        console.log(err);
        return;
    }
    console.log('data :', result);
});

/*
 * 修改
 * @param updateObj     修改内容（必填）
 * @param whereObj      修改对象（必填）
 * @param tableName     表名
 * @param callBack(err,recordset)
 */
helper.sql("update dbo.tableName set name = @updateObj where id = @whereObj", err => {
    if (err) {
        console.log("error:" + err);
        return;
    } else {
        console.log('Ok!');
    }
});

/*
 * 添加
 * @param addObj    添加对象（必填）
 * @param tableName 表名
 * @param callBack(err,recordset)
 */
helper.sql("insert into dbo.tableName(obj)values(@addObj)", err => {
    if (err) {
        console.log("error:" + err);
    } else {
        console.log("Ok!");
    }
})

/*
 * 删除
 * @param whereObj    删除对象（必填）
 * @param tableName 表名
 * @param callBack(err,recordset)
 */
helper.sql("delete dbo.tableName where 1 = 1 and id = @whereObj", err => {
    if (err) {
        console.log("error:" + err);
    } else {
        console.log("Ok!");
    }
})
```
以上就实现了 最简易的node.js + mssql的使用。

越努力，越幸运。
