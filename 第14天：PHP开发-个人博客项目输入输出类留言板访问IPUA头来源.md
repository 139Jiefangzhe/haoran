# 第14天：PHP开发-个人博客项目&amp;输入输出类&amp;留言板&amp;访问IP&amp;UA头&amp;来源

![1645081691502-ea9043d6-cb18-46a1-bb54-bcfe0f24f41e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184112898-360845566.png)

```plain
#知识点:
1、PHP-全局变量$_SERVER
2、MYSQL-插入语法INSERT
3、输入输出-XSS&反射&存储
4、安全问题-XSS跨站&CSRF等
```

#### 小迪博客-输入输出&留言板&访问获取

```plain
输入输出演示：
输入代码：
```

![1645081791416-c78d3912-7559-4d1e-93bd-ed2520c461d4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184128245-777889691.png)

```plain
输出代码，经过数据库处理：
```

![1645081816193-a54f72a2-1ae8-4078-9826-3e4ac906a132.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184139309-1068560811.png)

```plain
1. 留言内容在数据库里面
2. 加载前面的前端内容
3. 可以提交留言
4. 提交的留言之后会在加载
留言板演示：
前端演示，发送数据
```

![1645081853282-ec5832cf-98f4-4ac3-b1cc-bad275b218eb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184156420-179513090.png)

```plain
后台输出数据，会在页面中显示：
```

![1645081874923-2aa101cc-673c-40ac-8804-4ef888b9f637.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184208199-1335932935.png)

```plain
访问来源：
IP地址，浏览器信息，设备信息等等。
```

![1645081909903-d6abb1d5-845f-4843-b753-e8a5ccf8ff75.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184229426-1214377100.png)

![1645081921180-677083cc-fc53-46eb-b64f-7f1b529c9cd4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184235573-1012221177.png)

![1645081937195-eb4be164-4aaa-4e7d-8b80-611b772adf2f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184244275-539591717.png)

```plain
可以获取的参数：参考：https://www.cnblogs.com/xishaonian/p/6160893.html
```

![1645081961717-e90bed10-85f2-4499-a5ba-023628699900.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184306697-756562104.png)

```plain
https://ip.chinaz.com/ 获取IP地址，浏览器信息，Windows信息等等，可以通过抓包修改。
```

![1645081988463-5632f2a3-3765-4fdc-8d8a-c6b4afd641f2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184319627-604236697.png)

```plain
抓包修改后：
```

![1645082022196-ba062940-4d07-4caa-b594-975af0db7f3d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184330039-1916396402.png)

墨者靶场-IP地址伪造来源&来源页伪造

```plain
IP地址获取：
```

![1645082059769-706f55f8-4a59-490c-940d-a6f23b509921.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910184341943-951528631.png)

```plain
来源页获取：
https://blog.csdn.net/bnrmaster/article/details/52952095
Csrf:
1.
已知工具这知道的某个管理员的习惯，喜欢看色色
攻击者在色色上面植入了一个js代码能获取管理员管理的网站信息

管理员去访问一个色色会触发代码
由于管理员浏览器可以直接登录网站，管理权限。
管理的网站：检测来源，过滤了攻击
2.
www.alipay.com/pay?name=xiaodi&money=1000&account=1307023024@qq.com
我在我博客注入访问URL的代码：访问这个付款数据包。而这个时候你打开了我的博客并且你在支付宝处于登录状态。这个时候就会触发这个数据包，执行支付。支付宝检测到这个付款URL是从小迪博客过来的，GG。
```