# 第22天：WEB攻防-JS项目&amp;Node.JS框架安全&amp;识别审计&amp;验证绕过

![1645091193635-0a53fd5b-7858-42d5-878e-bb8aa7831650.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113354883-57938762.png)

```plain
#知识点：
1、原生JS&开发框架-安全条件
2、常见安全问题-前端验证&未授权

#详细点：
1、什么是JS渗透测试？
在Javascript中也存在变量和函数，当存在可控变量及函数调用即可参数漏洞
JS开发的WEB应用和PHP，JAVA,NET等区别在于即没有源代码，也可以通过浏览器的查看源代码获取真实的点。所以相当于JS开发的WEB应用属于白盒测试（默认有源码参考）

2、流行的Js框架有那些？
3、如何判定JS开发应用？
 插件wappalyzer
 源代码简短
 引入多个js文件
 一般有/static/js/app.js 等顺序的js文件
 cookie中有connect.sid
4、如何获取更多的JS文件？
 JsFinder
 Packer-Fuzzer
 扫描器后缀替换字典
5、如何快速获取价值代码？
 method:"get"
 http.get("
 method:"post"
 http.post("
 $.ajax
 service.httppost
 service.httpget
```

- #### 安全条件-可控变量&特定函数

```plain
本地代码演示
```

- #### 开发框架-Vulhub-Node.JS安全

```plain
来到相对应的目录，
docker-compose build 
docker-compose up -d 
config
```

- #### 真实应用-APP应用直接重置密码

```plain
用APP抓包，发现登录地址：mx2.simuwz.com/user/login
判断js:
```

![1645091341304-4fb4e2c4-ae10-4267-8d88-40f0a0d3d828.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113432269-412256914.png)

```plain
需要看js有没有嵌套js代码。
审查元素：关键性代码：
```

![1645091365376-983ab68f-9d42-49ef-843b-7f198ff1287d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113440505-1597192166.png)

```plain
未授权访问：
"/user/reset_password/" + e.phone
http://mx2.simuwz.com/user/reset_password/13554365566 ，就可以直接进入重置页面，但是会出现流程错误
关键判断：
```

![1645091365376-983ab68f-9d42-49ef-843b-7f198ff1287d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113449351-894631560.png)

- 真实应用-违法彩彩文件上传安全

```plain
Ctrl+f 可以进行关键字搜索：
```

![1645091425421-532904d9-376a-4e49-bb48-b1dcd7e522aa.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113458920-1347175320.png)

```plain
发现是前端验证代码：
```

![1645091454753-d5ca98aa-caad-48cf-a0b4-27cb4be8ca5c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113513009-1569458880.png)

```plain
但是这个网站是js开发的，所以不能禁用js
用网站下载器可以去下载到源码，然后在进行禁用，但是后面下载不下来。
然后看上传视频可不可以。发现上传视频没有验证。
```

![1645091478773-58b044ff-a041-4073-a27d-e04f04b24f15.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113522122-1182716594.png)

```plain
抓包发现上传视频地址：但是访问无效
```

![1645091513443-b12f4f08-2ad9-4a4e-9a67-64d81aa8e911.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912113632001-1733551259.png)

```plain
但是地址不解析PHP，不能连接。
```