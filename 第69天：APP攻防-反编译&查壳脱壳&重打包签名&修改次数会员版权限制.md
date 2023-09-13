![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135733773-155750592.png)

```plain
#知识点：
0、自动化查壳脱壳
1、图标&版权&信息等
2、功能修改&逆向破解
3、证书签名&重打包等

#章节点：
1、信息收集-应用&资产提取&权限等
2、漏洞发现-反编译&脱壳&代码审计
3、安全评估-组件&敏感密匙&恶意分析

#核心点：
0、内在点-资产提取&版本&信息等
1、抓包点-反代理&反证书&协议等
2、逆向点-反编译&脱壳&重打包等
3、安全点-资产&接口&漏洞&审计等
```

- 反编译加固-自动查壳脱壳

```plain
对apk进行处理的过程：
1.脱壳-后续操作
2.反编译-看源代码
3.修改后-限制突破
4.重打包-签名


查壳：ApkScan-PKID，把文件拖进去就可以查到壳
项目地址：https://github.com/CodingGay/BlackDex
安装，打开就可以使用

查壳探探：可以发现探探apk是网易易盾加固
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135803813-284815809.png)

```plain
如果有壳的话，对其进行反编译，用AppInfoScanner-master这个工具进行反编译。
执行：G:\python38\python.exe app.py android -i 探探.apk
但是发生了错误，是因为这个apk加壳了，反编译不了。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804494-1954409898.png)

```plain
脱壳：BlackDex，运行在模拟器上，安装进去就可以用了。不一定全部能脱壳

比如探探的壳，还有其他大型产生的壳是没办法脱壳成功的。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135802625-565867666.png)

```plain
如果不能进行脱壳就没办法进行反编译，那就只能放弃，搞其他方面的资产收集，对apk反编译就没办法进行进去。
有一些逆向大牛，对一些爱奇艺等软件进行反编译，把里面的广告啊，vip等进行清爽华，比如在https://www.52pojie.cn/上就有很多
```

- 功能修改-反编译&次数&会员

```plain
安卓修改大师破解操作：
1、下载并安装原程序，先不要运行软件；
2、接着把crack文件夹里的“ApkHelper.exe”拷贝到软件安装目录下替换；
【默认路径C:\Program Files (x86)\ApkEditor】
3、再用crack文件夹里的注册机进行注册；
4、至此就全部破解完成啦，用户可随意使用里面的功能了
```

次数

```plain
打开apk，发现没有进行登录，就有了观看次数，还有账号id等等
思考：为什么记住观看次数，还有账号id登信息？是通过手机型号来判定的吗？可以对手机型号进行修改
记一下观看次数9/10 账号id：54801087
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135802999-112208873.png)

```plain
然后对模拟器的属性进行修改，比如设备型号，手机号码，IMEI等进行修改，重启设备
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135803165-1268061058.png)

```plain
然后再次打开贵妃，次数跟账号id那就变成了初始化了。所以是通过手机设备来给一点账号，给一个次数来进行判定的。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135805111-752859985.png)

```plain
用到工具安装修改大师，把apk放进去就可以进行反编译。里面有很多功能，比如代码/布局修改，代码定位，搜索定位等。

对贵妃.apk进行反编译，进行搜索：今日剩余观影次数
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135803310-87312975.png)

```plain
打开/res/values/strings.xml文件，然后找到代码：<string name="today_has_view_count">今日剩余观影次数</string>
发现关键值today_has_view_count
搜索today_has_view_count
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804043-242613707.png)

```plain
然后打开/res/values/public.xml，这个文件只是定义了一点而已，可能是次数的大小等。
代码：<public type="string" name="today_has_view_count" id="0x7f0f010d" />

然后再打开/res/layout/fragment_mine.xml，也没有发现利用价值的东西。

如果他把观影完后，就是出现其他关键字，搜一下：次数
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135803957-1042364912.png)

```plain
定位到/smali/com/ufozfuxzqm/dvbphwfo/ni/activity/MoviesEDetailActivity.smali文件
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135805082-566891846.png)

```plain
关键代码：const-string v1, "今日观影次数不足,注册可增加观影次数!"
这个代码是smail代码，不是很懂，可以转化为java代码，但是这个java代码只能看不能修改，需要修改smail文件才能修改。

那么这个次数修改到底在哪里呢？刚刚搜索的关键字是：today_view
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135805049-1286237793.png)

```plain
其中有一个/smail/com/ufozfnxzqm/dvbphwfo/entity/UserInfo.smali文件，然后看他的java代码
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804899-1320567494.png)

```plain
这里有很多函数，比如
这里应该是获取vip的意思
    public int getIs_vip()
    {
        return is_vip;
    }

这里应该是获取次数的意思
    public String getRe_today_cache_times()
    {
        return re_today_cache_times;
    }
    public String getRe_today_view_times()
    {
        return re_today_view_times;
    }

返回的值是：re_today_view_times
类似初始化的意思，但是这里不能直接修改，只能修改smali文件

找到函数点：
.method public getRe_today_view_times()Ljava/lang/String;
    .locals 1

    .line 307
    iget-object v0, p0, Lcom/ufozfnxzqm/dvbphwfo/entity/UserInfo;->re_today_view_times:Ljava/lang/String;

    return-object v0
.end method

返回一个v0，在smail语法中就是返回一个对应的地方
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804133-1741236178.png)

```plain
把这个代码进行修改，让v0变成99999

.method public getRe_today_view_times()Ljava/lang/String;
    .locals 1
    .line 307
    iget-object v0, p0, Lcom/ufozfnxzqm/dvbphwfo/entity/UserInfo;->re_today_view_times:Ljava/lang/String;
    const-string v0, "9999999"
    return-object v0
.end method
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135805920-407824132.png)

```plain
修改完后，点击保存。然后进行重打包，点击项目打包，这样就可以完成打包，但是我这里打包失败了。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135803793-290021440.png)

会员

```plain
vip是怎么展示的，因为这里没有关键词，所以只能搜VIP的关键词，但是匹配了很多，在/smail/com/ufozfnxzqm/dvbphwfo/entity/UserInfo.smali文件中也有，优先看他
在java中间中：
    public int getIs_vip()
    {
        return is_vip;
    }
看一下smali代码，
.method public getIs_vip()I
    .locals 1

    .line 347
    iget v0, p0, Lcom/ufozfnxzqm/dvbphwfo/entity/UserInfo;->is_vip:I

    return v0
.end method
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804962-421520023.png)

```plain
如果从代码层面看的话，还要找到getIs_vip声明地方，看看逻辑
搜一下逻辑：is_vip
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135805001-97497132.png)

```plain
在/smali/com/ufozfnxzqm/dvbphwfo/ui/activity/PlayerEActivity.smali文件中看java代码，有关键判断：if(PlayerApplication.appContext.getmUserInfo() != null && PlayerApplication.appContext.getmUserInfo().getIs_open_time_limit() == 1 && PlayerApplication.appContext.getmUserInfo().getIs_vip() != 1)

layerApplication.appContext.getmUserInfo().getIs_vip() != 1
后面提示：
(new android.support.v7.app.AlertDialog.Builder(mContext)).setMessage("非vip试看时间已到,请开通VIP会员或免费领取").setPositiveButton(mContext.getString(0x7f0f0038), new android.content.DialogInterface.OnClickListener()
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804998-1566081020.png)

```plain
从这里可以看出，不等于1就不是会员，等于1就是会员，所以这里可以直接跳到/smail/com/ufozfnxzqm/dvbphwfo/entity/UserInfo.smali这个文件里来，然后把getIs_vip函数进行修改，让他的值为1

.method public getIs_vip()I
    .locals 1
    .line 347
    iget v0, p0, Lcom/ufozfnxzqm/dvbphwfo/entity/UserInfo;->is_vip:I
    const/4 v0, 0x1
    return v0
.end method
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804210-490349595.png)

```plain
保存后重打包，但是还是失败，应该是环境问题。
```

- 功能修改-反编译&图标&信息

```plain
也可以对贵妃的图标，信息，功能进行修改。

在安卓修改大师中，小插件功能也是可以加的，比如弹窗，广告
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804577-1675060767.png)

```plain
也可以进行图标替换，应用的名称
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135804137-1334798134.png)

```plain
可以进行图片修改，比如进来的图片等等
```

- 打包编译-证书签名&重打包等

```plain
证书签名可以用到逆向这方面的工具包。

在Android中的AndroidKiller
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135805070-1416775521.png)

```plain
app tools工具有app签名等
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135807408-2069729588.png)

```plain
这些工具功能比较单一，也没有安卓修改大师好用。不是很能成功。
```