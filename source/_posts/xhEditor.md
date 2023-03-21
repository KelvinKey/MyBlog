---
title: xhEditor编辑器上传图片到 OSS
date: 2018-09-27 14:28:54
tags:
---

 前段时间，公司在项目上用到了xhEditor编辑器来给用户做一个上传图片的功能当时做的时候觉得很有意思，想想 基本的用户图片上传到自己服务器，还有点小占地方；

后来....然后直接上传到阿里云 。接下来就是基本操作：
首先，引入官方提供的js库
{% asset_img 1412426-20180927110039107-1436532960.png This is an example image %}

注：xhEditor官网插件下载;OSS~库引入直接复制以下即可：

```
  <!-- oss 上传文件 JavaScript 库 -->  
    <script src="http://gosspublic.alicdn.com/aliyun-oss-sdk-4.4.4.min.js"></script>
```

其次，进入xhEditor官方提供的js库里面
{% asset_img 1412426-20180927112340706-1722091288.png This is an example image %}

注：因我当时看的是xhEditor压缩过后，所以看起来很不方便，所以在此表明黄色标注为：全局可搜索 。红色标注为：自定义属性 。
 
```
var i = e('<span class="xheUpload">
    i.append('<input type="file"' + (1 < p ? ' multiple=""' : "") + ' class="xheFile" size="13" id="UpOss"  name="filedata" tabindex="-1" />');
var h = e(".xheFile", i); h.change(function () {
```
接下来，在上面段落的Change事件中开始表演

第一步：声明自己的accessKeyId 和 accessKeySecret  这需要到阿里云里面去设置

{% asset_img 1412426-20180927115335795-2032634143.png This is an example image %}

```
var _Client = new OSS.Wrapper({
        region: 'oss-cn-shanghai',                              
        accessKeyId: 'xxxxxxxxxxxxxx',                          
        accessKeySecret: 'xxxxxxxxxxxxxxxxxxxxxxxxx',           
        bucket: 'xxxxxxx'
       });
```
第二步：紧接上面，文件上传

{% asset_img 1412426-20180927140612512-1537219366.png This is an example image %}

```
var _File = document.getElementById("UpOss").files[0];   // 获取文件流
   var _Val = document.getElementById("UpOss").value;       
   var suffix = _Val.substr(_Val.indexOf(".")),             // 文件名后缀名
                                   
     obj = timestamp(),     //文件名 也就是时间戳
     ymd = timesymd();      //自定义文件夹

   var stAs = ymd + "/" + obj + suffix;     //上传到阿里云的文件地址       
       _Client.multipartUpload(stAs, _File).then(function (result) {
           console.log(result);       //返回对象
           console.log(result.url);   //返回链接                    

        a.val(result.url);   //赋值
                     
       }).catch(function (err) {
          console.log(err);  // 返回异常
      });
```

生成文件夹 文件名

{% asset_img 1412426-20180927141104846-2119285573.png This is an example image %}

```
//文件夹 时间戳 
function timesymd() { 
    var time = new Date();
    var y = time.getFullYear(); 
    var m = time.getMonth() + 1;
    var d = time.getDate(); 
    return "" + y + _Add(m) + _Add(d)
     };

 //文件名  时间戳 
 function timestamp() {
    var time = new Date();
    var y = time.getFullYear();
    var m = time.getMonth() + 1;
    var d = time.getDate(); 
    var h = time.getHours();
    var mm = time.getMinutes();
    var s = time.getSeconds(); 
    return "" + y + _Add(m) + _Add(d) + _Add(h) + _Add(mm) + _Add(s);    }；

  function _Add(m) { return m < 10 ? '0' + m : m; }
```

最后，备注

 注：   此方法上传图片需要去阿里云配置上传权限

 {% asset_img 1412426-20180927141744334-1313129648.png This is an example image %}