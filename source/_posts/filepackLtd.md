---
title: C# 文件打包下载
date: 2021-12-30 10:16:13
tags:
---

今天偶然在实践中使用到了多文件需要打包下载的功能，这么说吧，可能内心还有一些美滋滋，又要到了每日的动脑动手环节了，直接石更（ying）核开干......
 
## 首先
不得不提到一个需要使用到的类库，那就是`SharpZipLib`，可能猛然一看还是有点陌生的，但是在项目中打开NuGet包管理器窗口，一点即搜，分分钟下载到位；

简单介绍一下`SharpZipLib`的功能：其实主要用来解压缩**Zip、GZip、BZip2、Tar** 等格式，是以托管程序集的方式实现，可以非常方便的应用于项目之中。
 
好了好了 不要聊了不要聊了  先上Code 先上Code..
 
你懂我的意思，需要先引用一下类库：
```
 using ICSharpCode.SharpZipLib.Zip;
```
 
接下来将虚拟路径映射到本地/服务器上的物理路径，如果文件已存在就删除，不存在就创建
```
#region 删除目录中现有文件及子目录
string filePath = System.Web.Hosting.HostingEnvironment.MapPath("~/Files/Temp/{0}/Test".Ft(DateTime.Now.ToString("yyyyMMdd")));
Directory.CreateDirectory(filePath);
DirectoryInfo dir = new DirectoryInfo(filePath);
FileSystemInfo[] fileinfo = dir.GetFileSystemInfos();//返回目录中所有文件和子目录
foreach (FileSystemInfo i in fileinfo)
{
    if (i is DirectoryInfo)//判断是否文件夹
    {
        DirectoryInfo subdir = new DirectoryInfo(i.FullName);
        subdir.Delete(true);//删除子目录和文件
    }
    else
    {
        System.IO.File.Delete(i.FullName); //删除指定文件
    }
}
#endregion
```

接下来流程就很简单了，就是首先获取我们需要下载的文件   --->  然后请求页面  --->  生成自己需要的文件路径及文件名称 ---> DownLoad到本地，如果有若干个文件可以考虑使用ForEach来进行遍历。

```
List<string> file = new List<string>();
var url = "https://baidu.com/Download/Files?access={0})".Ft(文件.pdf);
REST.GET(url);
 var urlfile = filePath + "\\" + $"{定义成自己需要的文件名称}";
file.Add(urlfile);
HttpDownloadFile(url, urlfile);

FilesZip(file,9,System.Web.Hosting.HostingEnvironment.MapPath("~/Files/Temp/{0}/{1}".Ft(DateTime.Now.ToString("yyyyMMdd"), DateTime.Now.ToString("yyyyMMddhhmmss") + ".zip")));
```
这是一个提供的下载文件的方法 ↓↓↓
```
public void HttpDownloadFile(string url, string path){
HttpWebRequest request = WebRequest.Create(url) as HttpWebRequest;
HttpWebResponse response = request.GetResponse() as HttpWebResponse;
Stream responseStream = response.GetResponseStream();
Stream stream = new FileStream(path, FileMode.Create);
byte[] bArr = new byte[1024];
int size = responseStream.Read(bArr, 0, (int)bArr.Length);
while (size > 0)
{
    stream.Write(bArr, 0, size);
    size = responseStream.Read(bArr, 0, (int)bArr.Length);
}
stream.Close();
responseStream.Close();
}
```
这是主要负责打包压缩，包括可以设置压缩级别、Zip包加密、Zip包注释
```
public void FilesZip(List<string> fileNames, int? compresssionLevel, string saveFullPath, string password = "", string comment = ""){
    using (ZipOutputStream zos = new ZipOutputStream(System.IO.File.Open(saveFullPath, FileMode.OpenOrCreate)))
    {
if (compresssionLevel.HasValue)
{
    zos.SetLevel(compresssionLevel.Value);//设置压缩级别
}

if (!string.IsNullOrEmpty(password))
{
    zos.Password = password;//设置zip包加密密码
}

if (!string.IsNullOrEmpty(comment))
{
    zos.SetComment(comment);//设置zip包的注释
}

foreach (string file in fileNames)
{
    if (System.IO.File.Exists(file))
    {
FileInfo item = new FileInfo(file);
FileStream fs = System.IO.File.OpenRead(item.FullName);
byte[] buffer = new byte[fs.Length];
fs.Read(buffer, 0, buffer.Length);

ZipEntry entry = new ZipEntry(item.Name);
zos.PutNextEntry(entry);
zos.Write(buffer, 0, buffer.Length);
fs.Close();
    }
}
zos.Close();
    }
}
```

## 结语
在普通的星期五，结束这普通的2021。
{% asset_img 1412426-20220613101021320-1014728214.png This is an example image %}
