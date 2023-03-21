---
title: Node.js实现简易的获取access_token
date: 2019-07-23 15:37:18
tags:
---

还是老样子，在自学node.js的道路上走得坑坑洼洼，按住了躁动的自己，调整好心情 ，ready........Go....! 

首先在项目里新建config.json，其中 appid 与 appscrect 两个参数 位于 [微信公众平台](https://mp.weixin.qq.com) 左侧菜单的基本配置中：

{% asset_img 1412426-20190723150011993-1655343884.png This is an example image %}
代码块如下：
```
{
    "token": "wechat",
    "appID": "wx376********7243e",
    "appScrect": "8962157f61*************1e04d244a",
    "apiDomain": "https://api.weixin.qq.com/",
    "apiURL": {
        "accessTokenApi": "%scgi-bin/token?grant_type=client_credential&appid=%s&secret=%s"
    }
}
```
接下来我新建accessToken.js，主要用于封装获取access_tocken的方法（名字随意）

一成不变,依赖选择：
{% asset_img 1412426-20190723151735308-334892714.png This is an example image %}

代码块如下，这里引入的配置config就是上面基本配置：
```
'use strict';
// ↑ 设置为严格模式

//引入依赖
const util = require("util")
const https = require("https");
//引入配置文件
const config = require("../config");
```
且，我这里定义的是全局变量用来存token，这里的存取的方法有很多种；比如:存为cookie，session，更新本地存储等......

{% asset_img 1412426-20190723152200176-1070479054.png This is an example image %}

代码块如下:
```
//全局
const accessTokenJson = {
    access_token: "",
    expires_time: ""
};
```

接下来就是获取access_tocken的方法：

{% asset_img 1412426-20190723152423268-1528732605.png This is an example image %}

代码块如下：

```
/** 
 * 获取access_tocken的方法
*/
const auth = {
    getAccessToken: err => {
        return new Promise(resolve => {
            //获取当前时间
            const currentTime = new Date().getDate();
            //格式化请求地址
            let url = util.format(config.apiURL.accessTokenApi, config.apiDomain, config.appID, config.appScrect);

            if (accessTokenJson.access_token == "" || accessTokenJson.expires_time < currentTime) {
                https.get(url, res => {
                    const buffer = [];
                    //监听
                    res.on('data', data => {
                        buffer.push(data);
                    });
                    res.on('end', err => {
                        let data = Buffer.concat(buffer).toString('utf-8');
                        let result = JSON.parse(data);

                        console.log(result);
                        //放入全局
                        accessTokenJson.access_token = result.access_token;
                        accessTokenJson.expires_time = new Date().getTime() + (parseInt(result.xpires_in) - 200) * 1000;

                        resolve(accessTokenJson.access_token);
                    });
                }).on("error", err => {
                    console.log(err)
                });
            } else {
                resolve(accessTokenJson.access_token);
            }
        });
    }
};

//test 主函数
auth.getAccessToken();

/** 暴露可供外部访问的接口
 * module.exports = auth;
  */
```

最后，这里直接 node accessToken.js 就Ok了。

PS：最后注释：暴露可供外部访问的接口 可以理解为这个可以封装为一个单一的方法，供其调用。

例如,在其他js中可以直接引入：

{% asset_img 1412426-20190723152834414-1442668152.png This is an example image %}

代码块如下:

```
'use strict';

const token = require("./accessToken");

token.getAccessToken().then(res => {
    //console.log(res)
}).catch(err => {
    //console.log(err);
})

```
获取access_token效果图:
{% asset_img 1412426-20190723153250221-942882036.png This is an example image %}

