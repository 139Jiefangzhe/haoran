![1645088630410-1886df7c-d7d3-4939-961f-7f58009a421e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094846241-50658237.png)

```plain
#知识点：
1、.NET配置调试-信息泄露
2、.NET源码反编译-DLL反编译
3、.NET常见安全问题-未授权访问
```

- #### .NET项目-DLL文件反编译指向-代码特性

```plain
一般都是对dll文件来进行反编译，才能得到相对应的关键代码。用到ILSpy软件进行反编译。
会存在包文件，就是dll文件。还有.config文件。webconfig比较重要，有bin目录，里面的文件是dll文件。
反编译结果：
```

![1645088740268-62882802-e9bf-4e76-8389-9d91ab2ed2e3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094905953-1479441336.png)

```plain
跟踪代码关键：
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Login.aspx.cs" Inherits="Book.Login" %>
```

![1645088764400-70ef5d96-f8f3-4a87-bf63-02d2f7cab899.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094916300-1121318062.png)

- .NET项目-Web.config错误调试-信息泄露

```plain
在web.config中有一个配置<customErrors mode ="off"></customErrors>
mode有三种状态off on 
off状态：会爆出一些源码信息，但是不完全
on状态：会爆出源码，指定设置报错页面等。
```

- .NET项目-身份验证未授权访问-安全漏洞

```plain
判断用户身份：
由于后台本身有多个功能文件页面
1.在每个文件里面添加判断代码。
2.创建一个文件专门用来判断，其他文件包含调用他。

找未授权访问：
1.找那些文件没有包含验证文件的文件
2.验证代码有没有可能绕过。

访问pd.aspx的时候，登录就可以访问，不登录不能访问，找到pd.aspx文件，寻找验证代码文件
发现验证包含：<%@PageTitle=""Language="C#"MasterPageFile="~/purchase/purchase.Master"ClientIDMode="Static"AutoEventWireup="true"CodeBehind="pd.aspx.cs"Inherits="purchase.purchase.pd"%>
找到相对应的dll文件，找到pd代码，不属于验证代码，关注到/purchase/purchase.Master
```

![1645088834636-da017fda-431d-439c-9650-702243a6999e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094933779-413606908.png)

```plain
找到文件/purchase/purchase.Master：
<%@MasterLanguage="C#"AutoEventWireup="true"CodeBehind="purchase.Master.cs"Inherits="Purchase.Purchase.pur"%>
找找pur相对应的dll文件代码：
```

![1645088861110-36cc090e-a9b2-4bc3-9292-2c09f6cd1c30.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094945227-1051673485.png)

```plain
核心验证代码：
if (UserHelper.GetUserId <= 0)
            {
                base.Response.Redirect("../login.aspx");
                return;
            }
跟踪UserHelper.GetUserId
```

![1645088898617-81d35513-c378-4435-805e-1327e7c9d225.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094955739-1897079075.png)

```plain
关键代码：
if (HttpContext.Current.Request.Cookies["userinfo"] != null)
            {
                return int.Parse(HttpContext.Current.Request.Cookies["userinfo"]["userid"]);
            }
            return -1;
}
只要让userinfo下的userid有值，就可以执行，用burpsuite抓包
添加Cookie=userinfo=userid=1，就可以进行绕过，完成。

找没有包含/purchase/purchase.Master的文件。就可以进行登录。
```