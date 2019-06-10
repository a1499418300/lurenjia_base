



QQ卡券开发接口协议



目   录
目录
目录	2
1 引言	4
1.1 文档概述	4
1.2 阅读对象	4
1.3 业务术语	4
2 整体流程图	5
3 数据格式	6
3.1 POST	6
3.2 数据格式	6
3.3 URL公用参数	6
4 签名&验签	7
4.1 签名算法	7
4.2 请求串防篡改签名	7
4.2.1 算法参数	7
4.2.2 签名算法	7
4.2.3 示例	7
4.3 回包串防篡改验签	8
4.4 领券链接-回调验签	8
4.4.1 签名原始串	8
4.4.2 签名算法	8
4.4.3 示例	9
5 用户鉴权凭证	9
6 接口	9
6.1 请求格式说明	9
6.2 领取券接口(gain)	9
6.2.1 业务功能	9
6.2.2 请求参数列表	9
6.2.3 输入参数说明	10
6.2.4 返回参数说明	11
6.2.5 返回结果示例	11
6.3 用户券消费接口（consume）	11
6.3.1 业务功能	12
6.3.2 请求参数列表	12
6.3.3 输入参数说明	12
6.3.4 返回结果示例	13
6.3.5 返回参数说明	13
6.4 用户券列表查询接口（getcardlist）	13
6.4.1 业务功能	14
6.4.2 请求参数列表	14
6.4.3 输入参数说明	14
6.4.4 返回结果示例	15
6.4.5 返回参数说明	15
6.5 用户券回滚核销接口（rollback_consume）	15
6.5.1 业务功能	16
6.5.2 请求参数列表	16
6.5.3 参数说明	16
6.5.4 返回结果示例	17
6.5.5 返回结果说明	17
6.6 查看Code码状态(getcodeinfo)	17
6.6.1 业务功能	18
6.6.2 请求参数列表	18
6.6.3 参数说明	18
6.6.4 返回示例	19
6.6.5 返回参数说明	19
7 常见错误码列表	20



1引言
1.1文档概述
本文档描述QQ卡券对外提供的接口说明，包含过年红包的发货确认接口，同时也可以作为日后接口参数以及参数类型的速查手册。
1.2阅读对象
供第三方开发人员与服务方技术或业务人员参考和查询。

1.3业务术语
术语	示例	说明
appid	10240241	应用的唯一标识
openid	E4A30F865E5486CC212C2A3D814A16C9	QQ用户在应用下对应的身份 
card_id		卡券ID。一个卡券ID对应一类卡券。可以此发放配置库存数量的卡券(一张一个Code码)。
code		卡券code码。一张卡券的唯一标识，核销卡券时使用此串码。
2
整体流程图

3
数据格式
3.1POST
为了保证数据的安全传输，采用HTTPS标准的POST协议。
3.2数据格式
采用标准JSON协议，所有POST的报文的内容格式JSON

3.3URL公用参数
所有请求post结构体第一层必带。详细如下：

参数名	示例值	描述
appid	10000	应用的唯一标识
timestamp	1438319793	请求的当前时间戳，1970-01-01 00:00:00以来流逝的时间(秒)，北京时间。接收到的请求的时间戳在15分钟内方认为有效。
signature	92c382e5b486aeed9f106534dae7cc3f	签名，签名算法见第3节
rand_str	3e5a036cb4bc3a677a38ad9d69eb3feb	随机字符串(a-zA-Z0-9)，不长于32位
示例：
请求确认红包接口(4.1)中，确认红包接口请求url为
https://api.qcoupon.qq.com/card/user/gain?signature=92c382e5b486aeed9f106534dae7cc3f
post内容：


4签名&验签 
4.1签名算法
统一采用MD5算法，接入前由卡券平台先分配对应业务的签名MD5 KEY，该请求签名、返回串签名和跳转链接签名采用同一个MD5 KEY
4.2请求串防篡改签名
4.2.1算法参数
变量名	示例值	描述
post_body	{"errcode":0,"errmsg":"ok","card_id":"p1Pj9jr90_SQRaVqYI239Ka1erkI"}	Post报文body
key	E1%g3a10	appid对应卡券平台的签名key

4.2.2签名算法
Step1. 原始串，拼接参数，这一步无须做url编码:
OriginStr := key=${key}&post_body=${post_body}

Step2. 对处理后的原始串作MD5(32位)加密，结果取小写：
Signature := toLower(MD5(${OriginStr}))

4.2.3示例
给定：
post_body：
{"appid":10000,"timestamp":1438319793,"rand_str":"3e5a036cb4bc3a677a38ad9d69eb3feb","req":{}}
key：
1234567ABCDEFG

Step1.
OriginStr := 
key=1234567ABCDEFG&post_body={"appid":10000,"timestamp":1438319793,"rand_str":"3e5a036cb4bc3a677a38ad9d69eb3feb","req":{}}

Step2. 
Signature = c795c23913286152adccab183541e3fa

4.3回包串防篡改验签
回包格式统一为：
signature=*******&result=******

result: json结果串
signature: MD5(key=${key}&result=${result})


4.4领券链接-回调验签
4.4.1签名原始串
变量名	示例值	描述
url_param	card_id=XX&appid=00&field=00&code=XXX&attach=hb_token%2623sa0ac%3Dbusiness%2611&rand_str=05d2ab5a1&signature=XX	领券跳转链接Url中所有其他参数
key	E1%g3a10	appid对应卡券平台的签名key (原appkey/encrypt_key)
4.4.2签名算法
Step1. 原始串，拼接参数(无须做url编码)
OriginStr := ${url_params}&key=${key}

Step2. 对原始串按变量名字典序(从小到大)重排，若无对应值则不参与签名:
OriginStr = DictReArrange(${OriginStr})

Step3. 对处理后的原始串作MD5(32位)，结果取小写：
Signature := toLower(MD5(${OriginStr}))
4.4.3示例
Step1. 请求参数附上key参数
card_id=XX&appid=00&field=00&code=XXX&attach=hb_token%2623sa0ac%3Dbusiness%2611&rand_str=05d2ab5a1&key=${key}
Step2. 对原串变量进行字典排序
appid=00&attach=hb_token%2623sa0ac%3Dbusiness%2611&card_id=XX&code=XXX&field=00&key=${key}&rand_str=05d2ab5a1
Step3. MD5(32位小写)运算
Signature := toLower(MD5(${OriginStr}))

5用户鉴权凭证
目前卡券平台根据不同场景提供了两种用户登录凭证鉴权方式：
1.对于日常接入需求，鉴权的凭证是access_token, access_token的获取可以参考文档《09免授权页跳转换用户身份说明.pdf》；
2.对于春节红包活动，考虑到活动时间短且大流量性能问题，卡券在详情页跳转第三方商户页面时，会带上openid参数，登录鉴权凭证加密后附带在attach中，在调用gain接口时，由卡券方验证attach用户的登录鉴权；
注：当从卡券详情页跳转第三方商户领取页面，如果链接上带了openid参数，则可视为采用方式2，否则采用方式1。
6接口
6.1请求格式说明
本文档为了方便开发者阅读，将所有json示例都进行了格式化展示，实际发送请求和数据签名的时候请不要带上空格和换行符，以免签名和服务端不一致被拒绝。
6.2领取券接口(gain)
6.2.1业务功能
确认领取过券，并设置券为已领用状态
6.2.2请求参数列表
请求url: 
https://api.qcoupon.qq.com/card/user/gain?signature=c4d712cee8e4c9bccaa5d844a66bc366

Post数据示例:

6.2.3输入参数说明
各个参数请进行URL 编码，编码时请遵守 RFC 1738

（1）公共参数
发送请求时必须传入公共参数，详见公共参数说明。


（2）私有参数

参数名称	是否必须	类型	描述	示例
code	是	String	卡券code，自领券链接获取。	9ffbb3eef3235c978eb864ac6fb1276f
card_id	是	String	卡券ID，自领券链接获取。	Pdl8UTflqvK7b3BpC_ZQAbvKiO55d5cm
access_token	是	String	Access_token是QQ用户在应用底下的登录票据，更多信息请参考FAQ。	FE0490SD8A02A51F72325AB346CCE2
openid	是	String	QQ用户在应用下对应的身份，可直接透传跳转领券链接中的openid参数	E4A30F865E5486CC212C2A3D814A16C9
attach	是	String	（注意为url参数） 领券链接附参。自领券链接获取，需与code，card_id一起保存。若链接未带此参数，则传空。具体参照流程图	hb_token%3Dabcd0123%26business%3D11%26sig%3Dl324s
gain_time	否	Int64	取人领取的时间戳。用于计算到期时间(不传默认按用户获得券的时间算)	1465120411

调试过程中的私有参数，参考文档《春节活动-外网联调工具.doc》最后一节自助生成。正式环境中的私有参数，由手Q前台跳转到第三方业务时带上。
6.2.4返回参数说明
参数名称	描述
errcode	返回码。
errmsg	如果错误，返回错误信息。
card_id	卡券id


6.2.5返回结果示例


6.3用户券消费接口（consume）
6.3.1业务功能
当用户已经使用了优惠卡券之后，第3方通过该接口可以改变用户卡的状态，完成数据同步。
6.3.2请求参数列表
请求url：
https://api.qcoupon.qq.com/card/user/usecard?signature=a84818d7f44e4db12a72d68994fb1202

Post数据示例:

6.3.3输入参数说明
各个参数请进行URL 编码，编码时请遵守 RFC 1738

（1）公共参数
发送请求时必须传入公共参数，详见公共参数说明。
（2）私有参数
参数名称	是否必须	类型	描述	示例
code	是	String	卡券code，自领券链接获取。	9ffbb3eef3235c978eb864ac6fb1276f
card_id	是	String	卡券ID，自领券链接获取。	Pdl8UTflqvK7b3BpC_ZQAbvKiO55d5cm
access_token	是	String	Access_token是QQ用户在应用底下的登录票据，更多信息请参考FAQ。	FE0490SD8A02A51F72325AB346CCE2
openid	是	String	QQ用户在应用下对应的身份，可直接透传跳转领券链接中的openid参数	E4A30F865E5486CC212C2A3D814A16C9
attach	是	String	领券链接附参。自领券链接获取，需与code，card_id一起保存。若链接未带此参数，则传空。具体参照流程图	hb_token%3Dabcd0123%26business%3D11%26sig%3Dl324s
6.3.4返回结果示例


6.3.5返回参数说明
字段名	说明
errcode	错误码，0为正常。
errmsg	错误信息。
card_id	卡券id


6.4用户券列表查询接口（getcardlist）
6.4.1业务功能
第3方通过该接口查询用户在第三方应用下的券列表。
6.4.2请求参数列表
请求url：
https://api.qcoupon.qq.com/card/user/getcardlist?signature=22f0c636b450230bcf24d4140526d342

post参数:

6.4.3输入参数说明
各个参数请进行URL 编码，编码时请遵守 RFC 1738

（1）公共参数
发送请求时必须传入公共参数，详见公共参数说明。

（3）私有参数
参数名称	是否必须	类型	描述	示例
condition	是	Int	筛选类型。
4=即将过期券(七天内)；2=无效券；1=除去4的有效券；
可组合如：5=1+4，即所有有效券；
不填写默认为7即全部。
如填写了card_id,会筛该券。	7
card_id	否	String	卡券ID。不填写时默认查询当前appid下所有的卡券。	Pdl8UTflqvK7b3BpC_ZQAbvKiO55d5cm
access_token	是	String	Access_token是QQ用户在应用底下的登录票据，更多信息请参考FAQ。	FE0490SD8A02A51F72325AB346CCE2
openid	是	String	QQ用户在应用下对应的身份，可直接透传跳转领券链接中的openid参数	E4A30F865E5486CC212C2A3D814A16C9
attach	是	String	（注意为url参数） 领券链接附参。自领券链接获取，需与code，card_id一起保存。若链接未带此参数，则传空。具体参照流程图	hb_token%3Dabcd0123%26business%3D11%26sig%3Dl324s

6.4.4返回结果示例

6.4.5返回参数说明
字段名	说明
errcode	int	错误码，0为正常。
errmsg	string	错误信息。
card_list	json	卡券列表,array成员如下
code	string	Code码
card_id	string	优惠券


6.5用户券回滚核销接口（rollback_consume）
6.5.1业务功能
通过该接口可以将已经核销成功的卡券扭转回未核销（可使用）状态，一般用于订单取消的业务场景。根据第三方特性，回滚有一定规则限制，如果使用需要先和卡券平台技术确认。
6.5.2请求参数列表
请求url：
https://api.qcoupon.qq.com/card/user/rollbackconsume?signature=0071ff9e491bcc06fee5c40f7e17d5a3
post参数:


6.5.3参数说明
各个参数请进行URL 编码，编码时请遵守 RFC 1738

（1）公共参数
发送请求时必须传入公共参数，详见公共参数说明。

（4）私有参数
参数名称	是否必须	类型	描述	示例
code	是	String	卡券code	9ffbb3eef3235c978eb864ac6fb1276f
card_id	否	String	卡券ID	Pdl8UTflqvK7b3BpC_ZQAbvKiO55d5cm
access_token	是	String	Access_token是QQ用户在应用底下的登录票据，更多信息请参考FAQ。	FE0490SD8A02A51F72325AB346CCE2
openid	是	String	QQ用户在应用下对应的身份，可直接透传跳转领券链接中的openid参数	E4A30F865E5486CC212C2A3D814A16C9
attach	是	String	（注意为url参数） 领券链接附参。自领券链接获取，需与code，card_id一起保存。若链接未带此参数，则传空。具体参照流程图	hb_token%3Dabcd0123%26business%3D11%26sig%3Dl324s

6.5.4返回结果示例


6.5.5返回结果说明
数据按JSON的格式实时返回
字段名	说明
errcode	错误码，0为正常。
errmsg	错误信息。
card_id	卡券id


6.6查看Code码状态(getcodeinfo)
6.6.1业务功能
查询code码状态, 包括是否领取，是否核销。
6.6.2请求参数列表
请求url：
https://api.qcoupon.qq.com/card/user/getcodeinfo?signature=bb8b2565780b16dc32913c3620d0f95c
post参数:


6.6.3参数说明
各个参数请进行URL 编码，编码时请遵守 RFC 1738

（1）公共参数
发送请求时必须传入公共参数，详见公共参数说明。
（5）私有参数
参数名称	是否必须	类型	描述	示例
code	是	String	卡券code	9ffbb3eef3235c978eb864ac6fb1276f
card_id	否	String	卡券ID	Pdl8UTflqvK7b3BpC_ZQAbvKiO55d5cm
access_token	是	String	Access_token是QQ用户在应用底下的登录票据，更多信息请参考FAQ。	FE0490SD8A02A51F72325AB346CCE2
openid	是	String	QQ用户在应用下对应的身份，可直接透传跳转领券链接中的openid参数	E4A30F865E5486CC212C2A3D814A16C9
check_uin	否	Bool	填写true时会检查传过来的open_id与发券时传的open_id是否相符， 不相符返回149956	true
check_consume	否	Bool	不填默认true,填true时如不能核销返回错误码40127	false
attach	是	String	（注意为url参数） 领券链接附参。自领券链接获取，需与code，card_id一起保存。若链接未带此参数，则传空。具体参照流程图	hb_token%3Dabcd0123%26business%3D11%26sig%3Dl324s

6.6.4返回示例


6.6.5返回参数说明
变量名	类型	描述
errcode	int	错误码，0为正常。
errmsg	string	错误信息。
card_id	string	卡券id
begin_time	int	有效期开始时间
end_time	int	有效期结束时间
user_card_status	string	卡券状态。
NORMAL 正常
CONSUMED 已核销
EXPIRE 已过期
DELETE 已删除
UNAVAILABLE 未领取
can_consume	String	是否可以核销


7 常见错误码列表
40013	不合法的Appid	149992	参数错误	149953	卡券未获取或已删除
40071	不合法的卡券类型	149993	请求id不存在的卡券	149988	code信息不正确
40072	不合法的编码方式	149996	卡券状态错误	149966	卡券已核销
40127	卡券非可核销状态	149946	卡券未通过审核	149954	卡券未核销
41011	缺少必填字段	149952	领取卡券失败	150001	卡券已领取
41012	缺少cardid参数	150002	活动不存在		
45021	字段超过长度限制	150003	用户无此红包		
149956	code码与uin不匹配	150020	attach参数无效		
149957	回滚核销失败	150004	红包已过期		
149958	缺少总量参数	39999	IP访问受限制		
149959	uin不合法	40003	不合法的openid		
149960	回滚核销卡券状态错误	40013	不合法的Appid		
149961	回滚核销超出限制时间	40014	不合法的token		
149962	批量添加卡券失败	40021	不合法的Appid		
149963	批量添加卡券数量过多	40071	不合法的卡券类型		
149965	code不存在	40072	不合法的编码方式		
149955	核销卡券错误	41011	缺少必填字段		
149967	添加卡券code重复	41012	缺少cardid参数		
149968	code为空	43001	非法请求		
149969	code不存在	43002	错误的接口/命令字 		
149971	code已存在	43003	时间戳已失效		
149972	code非自定义	43004	缺少signature参数		
149973	code需自定义	43005	没有访问接口的权限		
149974	date_info参数错误	43007	CGI不存在		
149975	color参数错误	43008	access_token、openid或attach参数错误		
149976	code_type参数错误	43009	用户登录态不正确		
149977	卡券库存发完	43011	缺少req参数		
149981	发券失败	44001	解码出错		
149984	日期格式错误	44002	内部解码错误		
149985	个人领取次数限制	44003	签名不合法		
149986	卡券未生效	45021	字段超过长度限制		
149987	卡券已过期	150009	平台部查询存储失败		
149990	卡券类型独有字段出错				
149991	logo_url域名不合限制				
