# 第7天：信息打点-资产泄漏&amp;CMS识别&amp;Git监控&amp;SVN&amp;DS_Store&amp;备份

![1645075939895-2229cdbf-0760-43bc-8718-483c600687a8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172029382-723021632.png)

```plain
#知识点：
1.CMS指纹识别源码获取方式 源码获取方法
2.习惯&配置&特性等获取方式
3.托管资产平台资源搜索监控 GitHub，gitee

#详细点：
参考：https://www.secpulse.com/archives/124398.html
源码泄漏原因：5个方面
1、从源码本身的特性入口
2、从管理员不好的习惯入口
3、从管理员不好的配置入口
4、从管理员不好的意识入口
5、从管理员资源信息搜集入口

源码泄漏集合：
composer.json（比较经典，针对PHP）
git源码泄露（比较经典）
svn源码泄露
hg源码泄漏
网站备份压缩文件
WEB-INF/web.xml 泄露
DS_Store 文件泄露
SWP 文件泄露
CVS泄露
Bzr泄露
GitHub源码泄漏

相关利用项目：
CMS识别：https://www.yunsee.cn/
备份：敏感目录文件扫描-7kbscan-WebPathBrute
CVS：https://github.com/kost/dvcs-ripper
GIT：https://github.com/lijiejie/GitHack
SVN：https://github.com/callmefeifei/SvnHack
DS_Store：https://github.com/lijiejie/ds_store_exp

GITHUB资源搜索：
in:name test #仓库标题搜索含有关键字 
in:descripton test #仓库描述搜索含有关键字 
in:readme test #Readme文件搜素含有关键字 
stars:>3000 test #stars数量大于3000的搜索关键字 
stars:1000..3000 test #stars数量大于1000小于3000的搜索关键字 forks:>1000 test #forks数量大于1000的搜索关键字 
forks:1000..3000 test #forks数量大于1000小于3000的搜索关键字 size:>=5000 test #指定仓库大于5000k(5M)的搜索关键字 pushed:>2019-02-12 test #发布时间大于2019-02-12的搜索关键字 created:>2019-02-12 test #创建时间大于2019-02-12的搜索关键字 user:test #用户名搜素 
license:apache-2.0 test #明确仓库的 LICENSE 搜索关键字 language:java test #在java语言的代码中搜索关键字 
user:test in:name test #组合搜索,用户名test的标题含有test的
关键字配合谷歌搜索：
site:Github.com smtp 
site:Github.com smtp @qq.com 
site:Github.com smtp @126.com 
site:Github.com smtp @163.com 
site:Github.com smtp @sina.com.cn 
site:Github.com smtp password 
site:Github.com String password smt
```

- #### 直接获取-CMS识别-云悉指纹识别平台

```plain
cms平台，需要花钱注册，不一定需要100%:https://www.yunsee.cn/
可以直接识别cms，就可以在网上直接下载，获取到源码，根据特性来搭建识别。
识别有指纹的，像百度，淘宝这些是不可以识别的，是属于内部团队开发的。
识别到web信息，IP信息，域名信息。
如果识别不了，那么就是自己开发等源码
获取源码，获取这套源码在网上有没有出现过漏洞，这就是cms获取的意义。
没有账号，演示不了，不过也是差不多的，只是工具而已。
```

- #### 习惯不好-备份文件-某黑阔博客源码泄漏

```plain
http://www.h0r2yc.com/
扫描到1.zip，访问www.h0r2yc.com/1.zip就可以进行下载。
如果在www.xiaodi8.com同目录备份，就不会出现问题，无法下载到，如果在www.xiaodi8.com下面备份，就会出现安全问题
c:/wwwroot/www.xiaodi8.com/www.xiaodi8.com.rar 可以下载
c:/www.xiaodi8.com.rar 不能下载
可以下载：
```

![1645076193102-6f530d21-fdb8-4271-8511-9a0c2b727734.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172132878-895713229.png)

```plain
不能下载：访问域名不能下载，但是如果用IP访问的话，那么就要看IP指向的是不是www这个目录。具体能不能下载看网站情况。IP指向目录跟域名指向目录是不同的。
```

![1645076231392-d9fae12e-db16-482a-a353-f6c26001fb03.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172144354-66322925.png)

```plain
利用..上一级是不可以的。没有接受进行路径操作。
```

- #### 配置不当-GIT泄漏-某程序员博客源码泄漏

```plain
Git是一个开源的分布式版本控制系统，在执行git init初始化目录的时候，会在当前目录下自动创建一个.git目录，用来记录代码的变更记录等。发布代码的时候，如果没有把.git这个目录删除，就直接发布到了服务器上，攻击者就可以通过它来恢复源代码。
把源码进行Git同步，.git目录是为了云功能同步
如果判断存在 直接在网站后面加上.git
121.36.49.234/.git,出现403发现存在目录
用到GitHub可以获取源码

判断存在：http://121.36.49.234/.git/
```

![1645076276862-c33f4256-e8b3-4021-8bf5-4a6bd9c26ea8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172208684-612971660.png)

```plain
安装GitHack：https://github.com/lijiejie/GitHack，运行
```

![1645076299183-037e2df4-b71d-468f-b985-2059397829ca.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172221583-82845572.png)

```plain
在当前目录下生成源码文件：
```

![1645076321483-56df08b4-d690-49a8-97c7-05007338a339.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172233128-1839276014.png)

- #### 配置不当-SVN泄漏-某国外小伙子源码泄漏

```plain
SVN是一个开放源代码的版本控制系统。在使用SVN管理本地代码过程中，会自动生成一个名为.svn的隐藏文件夹，其中包含重要的源代码信息。网站管理员在发布代码时，没有使用‘导出’功能，而是直接复制代码文件夹到WEB服务器上，这就使.svn隐藏文件夹被暴露于外网环境，可以利用.svn/entries文件，获取到服务器源码。
如git同理
trafficbonus.com/.svn/entries
用到SvnHack-master工具
python SvnHack.py -u http://x.x.x.x/.svn/entries —download

访问http://trafficbonus.com/.svn/entries，如果存在，则证明有这个漏洞：
```

![1645076358138-f88603d8-333c-4cd7-9ef3-9ed08379e491.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172254870-976989166.png)

```plain
执行：python2 SvnHack.py -u https://github.com/callmefeifei/SvnHack --download，但是下载失败，应该是环境的问题。
```

![1645076386933-a7542a79-e1a8-44f6-9f92-2c0083dca315.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172315334-630713835.png)

- #### 配置不当-DS_Store泄漏-某开发Mac源码泄漏

```plain
.DS_Store是Mac下Finder用来保存如何展示 文件/文件夹 的数据文件，每个文件夹下对应一个。如果将.DS_Store上传部署到服务器，可能造成文件目录结构泄漏，特别是备份文件、源代码文件。
zhuchao.yslts.com/.DS_Store 判断存在
工具用法：ds_store_exp.py http://hd.zj.qq.com/themes/galaxyw/.DS_Store
访问zhuchao.yslts.com/.DS_Store，直接提示下载：
```

![1645076425773-727524a1-eebd-4bb0-89d4-08c297dc67bb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172340744-427849049.png)

```plain
下载后打开是乱码：
```

![1645076446030-2e310eff-924c-41f1-b1ba-9575733c3110.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172352121-1357122900.png)

```plain
用到工具执行python2 ds_store_exp.py http://zhuchao.yslts.com/.DS_Store 出现错误：环境问题，问题没有解决
```

![1645076470848-a02b7045-32e1-4e9f-bb8d-5c6b9790c9aa.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172407596-1628352950.png)

- #### PHP特性-composer.json泄漏-某直接搭建源码泄漏

```plain
参考文章：https://blog.csdn.net/qq_35655945/article/details/79694249，可以理解为配置文件，可以通过它来获取源码文件。
配置性文件，可以通过他来获取源码信息，获取cms等
访问：english.cmdesign.com.cn/composer.json
这是PHP特性，只能在PHP源码网站才会发生
访问english.cmdesign.com.cn/composer.json：
```

![1645076517320-e5932daf-bde6-41a9-8048-0ebd73a663dd.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172429868-1501161885.png)

- #### 下载配合-WEB-INF泄露-RoarCTF-2019-EasyJava

```plain
wp:https://blog.csdn.net/silencediors/article/details/102579567 需要配合下载漏洞。
网站:buuoj.cn
WEB-INF是Java的WEB应用的安全目录，如果想在页面中直接访问其中的文件，必须通过web.xml文件对要访问的文件进行相应映射才能访问。

WEB-INF 主要包含一下文件或目录：
WEB-INF/web.xml : Web应用程序配置文件, 描述了servlet和其他的应用组件配置及命名规则.
WEB-INF/database.properties : 数据库配置文件
WEB-INF/classes/ : 一般用来存放Java类文件(.class)
WEB-INF/lib/ : 用来存放打包好的库(.jar)

WEB-INF/src/ : 用来放源代码(.asp和.php等)通过找到 web.xml 文件，推断 class 文件的路径，最后直接 class 文件，再通过反编译 class 文件，得到网站源码。
但是一般来说，就算知道相关路径，没有下载漏洞也不能进行测试。需要配合其他漏洞进行测试。

打开地址，发现：http://2c49355e-4d1b-40df-899d-5d4bc42333b4.node4.buuoj.cn:81/Download?filename=help.docx，存在下载漏洞。
把filename=help.docx用post提交，就会下载这个文件：
```

![1645076577875-de721012-ab2e-43db-a654-69e6bbae1632.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172614042-2038806362.png)

```plain
控制help.docx来下载WEB-INF/web.xml文件。Filename=WEB-INF/web.xml下载打开，观察到代码。
```

![1645076602462-70ba75fb-baa2-460d-a0dd-d751fce6bad5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172510758-440043926.png)

```plain
构造下载地址：filename=WEB-INF/classes/com/wm/ctf/FlagController.class，下载完成后用IDE打开。
```

![1645076623650-23446359-371e-492c-b31d-512afec3146f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172523386-1145490660.png)

- #### 资源监控-GITHUB泄漏-语法搜索&关键字搜索&社工

```plain
网站：blog.aabyss.cn
在GitHub中直接搜索aabyss

社工用户：Norah C.IV 一般会保留个人信息，开发者会在代码中保留，账号密码等。可以进行脚本监控，如果监控到了就会进行发送信息
1.小迪博客资源中邮箱等关键字
```

![1645076659955-ec434994-b584-4400-b8db-d4526e515c93.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910172550983-2072014494.png)

```plain
也可以在bing上搜索。

2.渊龙sec安全团队blog.aabyss.cn

3.群成员 花捞的信息Norah C.IV
```