# 断点续传文件

## 原理知识



问：什么是断点续传？断点续传的原理是什么？

- 　答：断点续传就是信号中断后(掉线或关机等)，下次能够从上次的地方接着传送(一般指下载或上传)，不支持断点续传就意味着下次下载或上传必须从零开始。http协议中的断点续传是基于Http头Range以及Content-Range。HTTP头中一般断点下载时才用到Range和Content-Range实体头，Range用户请求头中，指定第一个字节的位置和最后一个字节的位置，如（ Range:200-300或者Range:200- ）；Content-Range用于响应头。通俗的来讲就是文件大小为10，这次下载了3，被中断了，下次继续下载时则将指针移到3位置，从3开始下载，最终将整个文件下载下来。



## Header的格式

请求下载整个文件: 

```html
GET /test.rar HTTP/1.1 
Connection: close 
Host: 192.168.95.11
Range: bytes=0-801 //一般请求下载整个文件是bytes=0- 或不用这个头
```

一般正常回应 ：

```
HTTP/1.1 200 OK 
Content-Length: 801      
Content-Type: application/octet-stream 
Content-Range: bytes 0-800/801 //801:文件总大小
```

Content-type：

Content-type 告诉浏览器文件的MIME 类型，这是非常重要的一个响应头了，MIME种类繁多。很可能会在程序中漏掉一些MIME类型，表示全部为 

content-type:application/octet-stream（字节流）

Content-Disposition：是 MIME 协议的扩展，MIME 协议指示 MIME 用户代理如何显示附加的文件。当 Internet Explorer 接收到头时，**它会激活文件下载对话框**，它的文件名框自动填充了头中指定的文件名。 嗯,就是这个头哟,激活弹出提示下载框，一般这样写content-disposition:attachment; filename=name

Content-Length："Content-Length: 321" 就是告诉浏览器这个文件的大小是321字节,其实我发现好像不设置这个头,浏览器也能自己识别

Pragma Cache-control：把这2个头都设置成public 告诉浏览器缓存，我一般设置cache-control:public

Content-Range：字段说明服务器返回了文件的某个范围及文件的总长度。这时Content-Length字段就不是整个文件的大小了，而是对应文件这个范围的字节数，这一点一定要注意。一般格式，

```
Content-Range: bytes 500-999/1000
```

## Range：

可以请求实体的一个或者多个子范围。

例如：
　　表示头500个字节：bytes=0-499
　　表示第二个500字节：bytes=500-999
　　表示最后500个字节：bytes=-500
　　表示500字节以后的范围：bytes=500-　　【下载断点续传（一般range格式为500-）】
　　第一个和最后一个字节：bytes=0-0,-1
　　同时指定几个范围：bytes=500-600,601-999
　　但是服务器可以忽略此请求头，如果无条件GET包含Range请求头，响应会以状态码206（PartialContent）返回而不是以200（OK）。【206表示服务器已经完成get的部分请求，即表示断点续传】

# TODO:

1. JAVA服务端如何控制下载速度(php可以在代码中设置?)
2. 在返回下载时, sleep(代码中制造延迟), java是否可以设置;
3. 使用Wireshark抓包工具进行抓包分析入门;

工具学习:

1. Chrome Dev;
2. Chrome extension 开发;
3. 

