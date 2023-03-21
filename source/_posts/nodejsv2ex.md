---
title: 基于V2EX API的nodejs组件.
date: 2019-07-29 15:00:53
tags:
---

今天又学习到了新的知（zi）识（shi），来给自己做个笔录，也算在这酷热的天气里给自己写了一篇降温的‘膏药’，话就讲这么多了 ，start off......

首先 ，依赖选择：

```
/**设置为严格模式*/
'use strict';

//引入依赖
const https = require('https');
const querystring = require('querystring');
const util = require('util');
```

下面代码块，可以理解为封装好的模块，创建命名为：V2exApi.js（此处随意），如何调用下面会讲到，go on......

```
//定义类
class V2exApi {

    /**
    * V2exApi constructor.
    */
    constructor() {
    }

    /**方案一 
    * @param string api
    * @param array  params
    * @param bool   $format
    *
    * @return mixed|string
    *
    * @throws HttpException
    */
    request(api, params = {}, $format = true) {

        let apiDomain = "https://www.v2ex.com/api/";

        //格式化地址
        let apiUrl = util.format(api, apiDomain, Object.keys(params).length > 0 ? querystring.stringify(params) : '');
        const result = "";
        https.get(apiUrl, res => {
            const buffer = [];
            res.on('data', data => {
                buffer.push(data);
            });
            res.on('end', err => {
                let data = Buffer.concat(buffer).toString('utf-8');
                result = $format ? JSON.parse(data) : data;
            });

        }).on('error', err => {
            console.log(err);
        });

        return result;
    }

    /**方案二  异步  
        * @param string api
        * @param array  params
        * @param bool   $format
        *
        * @return mixed|string
        *
        * @throws HttpException
        */
    // request(api, params = {}, $format = true) {

    //     return new Promise(resolve => {
    //         //格式化请求地址
    //         let apiDomain = "https://www.v2ex.com/api/";

    //         let apiUrl = util.format(api, apiDomain, Object.keys(params).length > 0 ? querystring.stringify(params) : '');
    //         const result = "";
    //         https.get(apiUrl, res => {
    //             const buffer = [];
    //             res.on('data', data => {
    //                 buffer.push(data);
    //             });
    //             res.on('end', err => {
    //                 let data = Buffer.concat(buffer).toString('utf-8');
    //                 result = $format ? JSON.parse(data) : data;
    //                 resolve(result);
    //             });

    //         }).on('error', err => {
    //             resolve(err);
    //         });
    //     });
    // }

    /**
     * 获取最热主题.
     *
     * @param bool $format
     *
     * @return mixed|string
     *
     * @throws HttpException
     */
    getHotTopics($format = true) {
        return this.request('%stopics/hot.json', {}, $format);
    }

    /**
     * 获取最新主题.
     *
     * @param bool $format
     *
     * @return mixed|string
     *
     * @throws HttpException
     */
    getLatestTopics($format = true) {
        return this.request('%stopics/latest.json', {}, $format);
    }

    /**
     * 获取节点信息.
     *
     * @param string $name
     * @param bool   $format
     *
     * @return mixed|string
     *
     * @throws HttpException
     */
    getNode(name, $format = true) {
        return this.request('%snodes/show.json?%s', { 'name': name }, $format);
    }

    /**
     * 根据用户名获取用户信息.
     *
     * @param string $username
     * @param bool   $format
     *
     * @return mixed|string
     */
    getMemberByUsername(username, $format = true) {
        return this.request('%smembers/show.json?%s', { 'username': username }, $format);
    }

    /**
     * 根据用户 ID 获取用户信息.
     *
     * @param int  $id
     * @param bool $format
     *
     * @return mixed|string
     */
    getMemberByID(id, $format = true) {
        return this.request('%smembers/show.json?%s', { 'id': id }, $format);
    }
}

//暴露接口
module.exports = V2exApi;
```
## 使用 
在其它JS中引入下列：
```
const V2exApi = require('./V2exApi');

const $v2ex = new V2exApi();
```
Example：

### 获取最热主题

```
方案一：
response = $v2ex.getHotTopics();

方案二 异步方法：
response = $v2ex.getHotTopics().then(res => {
//....
}).catch(err => {
//....
});

或者
async function test () {
response = await $v2ex.getHotTopics();
}
test();
```
 
以上就是基于V2EX API的nodejs组件了，是不是很简单0.0..

git源码地址：[https://github.com/KelvinKey/v2ex-api](https://github.com/KelvinKey/v2ex-api)

V2EX API接口地址：[https://www.v2ex.com/p/7v9TEc53](https://www.v2ex.com/p/7v9TEc53)

 

End，努力做个多思考的人吧.