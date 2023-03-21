---
title: 基于高德开放平台的 NODE 天气信息组件
date: 2019-07-30 18:16:37
tags:
---

看看了画在手上的JQuery手表，马上就快到了下班的时间了，心里总觉的空唠唠的， 好像空缺了什么一样，仔细的想了一想，微微叹了一口气，觉得是时候在这里和大家分享一下原因了........

首先：

## 安装
```
npm install util
npm install https
npm install querystring
```
## 配置

在使用本扩展之前，你需要去 [高德开放平台](https://lbs.amap.com/dev/id/choose) 注册账号，然后创建应用，获取应用的 API Key。

就是它↓↓↓：

{% asset_img 1412426-20190730175815441-1317321424.png This is an example image %}

万成不变，不离其中 引入依赖，顺便设定为严格模式：
```
/**设置为严格模式*/
'use strict';

/**
 * dependence
 */
const https = require('https');
const util = require('util');
const querystring = require('querystring');
```
下面就是重要内容代码块，命名：weather.js（random）.
```
/**
 *  Weather.
 */
const Weather = {
    /**
     * url
     */
    url: "https://restapi.amap.com/v3/weather/weatherInfo?%s",
    /**
     * API Key
     */
    key: "xxxxxxxxxxxxxxxxxxxxxx",
    /**
     * @param string city
     * @param string format
     *
     * @return mixed|string
     */
    getLiveWeather: (city, format = 'json') => {
        return Weather.getWeather(city, 'base', format);
    },
    /**
     * @param string city
     * @param string format
     *
     * @return mixed|string
     */
    getForecastsWeather: (city, format = 'json') => {
        return Weather.getWeather(city, 'all', format);
    },
    /**
     * @param string city
     * @param string type
     * @param string format
     *
     * @return mixed|string
     *
     * @throws HttpException
     * @throws InvalidArgumentException
     */
    getWeather: (city, type, format = 'json') => {
        if (!['base', 'all'].includes(type.toLowerCase())) {
            console.error('Invalid type value(base/all):', type);
            return;
        }
        if (!['json', 'xml'].includes(format.toLowerCase())) {
            console.error('Invalid response format(json/xml):', format);
            return;
        }

        let query = querystring.stringify(
            {
                key: Weather.key,
                city: city,
                output: format.toLowerCase(),
                extensions: type.toLowerCase()
            }
        );

        let getUrl = util.format(Weather.url, query);
        
        https.get(getUrl, res => {
            const buffer = [];
            res.on('data', data => {
                buffer.push(data);
            });
            res.on('end', () => {
                let data = Buffer.concat(buffer).toString('utf-8');
                console.log("json" === format ? JSON.parse(data) : data);
            });
        }).on('error', err => {
            console.log(err);
        });

    }
};

//暴露接口
module.exports = Weather;
```

## 使用
在其它JS中引入下列代码：
```
const weather = require('./weather');
```
然后就可以

### 获取实时天气 
```
response = weather.getLiveWeather('上海');
```
 
### 获取天气预报
```
response = weather.getForecastsWeather('上海', 'json');
```
 
### 获取 XML 格式返回值
第三个参数为返回值类型，可选 `json` 与 `xml`，默认 `json` ：
```
response = weather.getLiveWeather('上海', 'xml');
```
 
## 参考
[高德开放平台](https://lbs.amap.com/dev/id/choose)
 
## End
Git源码地址：[https://github.com/KelvinKey/weather](https://github.com/KelvinKey/weather)
 
好了，到这里我想说的就差不多结束了，明天丶后天的天气也知道的差不多了；心中的空缺也弥补上了.... Off work
 

越努力 ，越幸运。
