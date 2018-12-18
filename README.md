# 针对gensee(展视互动) 防盗链机制的破解

**仅用于学习、请勿用于商业用途，如若侵权、请告知**

### 破解缘由

​	这是我最近在教授爬虫内容的学生在学习如何使用selenium进行动态资源爬取、模拟点击课后练习时候突发奇想，希望能够帮助他的妈妈，针对妈妈正在上的夜校网课的视频内容进行录屏下载，可以方便妈妈的学习。我非常认可学生的行为，并希望能够给予帮助。本着进一步加深对流媒体机制的学习以及贡献自己的一部分力量，开始对http://www.xuetian.cn/进行尝试性分析。

​	（先吐槽后端疯狂轮训CheckLoginTerminal检查）

​	乍一看众多请求，无从下手。

​	![image-20181216224228915](https://ws4.sinaimg.cn/large/006tNbRwly1fy8yv3gculj31580ri48f.jpg)

​	首先尝试看看感兴趣的swf小文件，发现是网站学习时候的ppt内容，前后十一页，害死强迫症。

​	尝试无果后目标转向其他的请求，针对xml等，想着应该是各种配置文件。

​	在第一个`crossdomain.xml`请求的`Request Headers`中看到了上次请求的地址，这个不应该暴露出来，泄露隐私信息，

```json
[
	{"key":"nickName","value":"%e5%b8%ad%e9%a3%9e%e9%a3%9e"},								{"key":"token","value":"888888"},
	{"key":"uid","value":"1000000000"},														{"key":"k","value":"FDE21C3799BC35FBD50910979F2772F8                                         C3809BD5C3E769713C56948336DFE7309DEE6DE9DB689E95E7614F49CD05D93E9E74BB11BE7498F84F6AFA7080343F3599D37CB09643640F"}
]
```

这条请求里可以看到nickname、token、uid、k，看到这边的k感到很奇怪，应该是一个校验码之类的，此时也看到了视频左上方的logo，发现时gensee的接口，遂查看了他们的接口文档

![image-20181216225857251](https://ws1.sinaimg.cn/large/006tNbRwly1fy8zc5hdl9j312y0mytmc.jpg)

K值的验证为什么需要第三方验证，这个又是从何而来？带着这样的问题，尝试去掉K值，可以得到正常观看地址；后再大胆一点，去掉所有的参数，竟然依旧可以得到观看地址，也就是说真正起作用的只有play后的那串字符串标识，这个虚晃一枪晃到了用户。。。。

​	正经回来，换个请求继续看，`config?code=` ，访问后得到一个xml文件，里面有着`record path`，很感兴趣

```xml
<record path="http://cache.gensee.com/gsgetrecord/record11.gensee.net/gsrecord/2672/upload/2018_05_09/9713432_1525856650/record83.xml"/>
```

访问该地址，得到另一份配置文件`record83.xml` ，在这里我看到之前发现的十一张ppt内容，验证了这份xml的价值。继续搜索关键字段，

```xml
<conf name="CastMaker_Edit" id="CastMaker" starttime="2018-05-09 08:05:08" duration="902.860" endtime="2018-05-09 08:20:10" storage="10943234" kbps="96.965" novideo="false" videoheight="240" videowidth="320" audiocodec="11" ver="2" annofile="anno.xml" hls="hls/record.m3u8" jsanno="anno.js" hlsaudioonly="hls/recordaudioonly.m3u8" continue="true" js="record83.xml.js">
```

看到`conf`标签中的属性`hls`有`record`字样，后缀为`m3u8`，以及`hlsreadonly`属性，还有文件基础信息`starttime、duration、endtime、kbps`等,让我认为这个应该就是最终文件的地址，但是并不知道该如何得到该地址，尝试拼接URI，并最终得到

```html
http://cache.gensee.com/gsgetrecord/record11.gensee.net/gsrecord/2672/upload/2018_05_09/9713432_1525856650/hls/record.m3u8
```

该地址得到了一个`record.m3u8`的文件，查看文件内容如下：

![image-20181216233502034](https://ws4.sinaimg.cn/large/006tNbRwly1fy90ncs4y9j30m410q453.jpg)

看到这里就基本完成了这次的破解，ts视频流就是我们要的内容，可以想到应该是没有声音的，配合audio可以合并成一个完整的视频。

![image-20181216234705896](https://ws1.sinaimg.cn/large/006tNbRwly1fy90q5waibj313c07c0ul.jpg)

### 总结

- 整体来说防盗链的想法是好的，对于视频流都隐藏在配置文件中
- 但是对于视频流的实际地址可以在做一层保护，起码不能让用户猜到实际位置
- 最后还是在吐槽K值校验的航母造的大
