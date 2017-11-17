# XUPT_ZHJSRequestAPI
西安邮电大学—智慧教室请求数据说明文档

智慧教室请求接口说明
---
### 一级URL：http://jwkq.xupt.edu.cn:8080
### 一.登录接口
#### 1.1 获取验证码
**请求方式：** GET
**二级URL：** /Common/GetValidateCode?time= { 当前时间戳 }
+ 请求URL示例：http://jwkq.xupt.edu.cn:8080/Common/GetValidateCode?time=1510887972
> 由于学校的网页的验证码存在有效期，所以每次获取图片的时候尽量获取当前时间的验证码；
> 并从获取的验证码的返回头中获取相应的验证码 Cookie ，用于登录请求时的验证码验证。

**返回参数：**
+ 返回的 Respond Body 是图片的流式文件，可以用图片框架，如：Glide 等，进行图片的加载显示，
+ 也可以自己开输入流读入，读入的流式文件，根据不同平台进行流数据向图片数据转换；

下面以 Android 为例：
```java
	/*读取返回数据的流文件*/
	 InputStream is = response.body().byteStream();
	 // 利用bitmap 解码方法 把流文件转化为Bitmap 图片像素文件；
	 Bitmap bm = BitmapFactory.decodeStream(is);
```

==注意！==
无论你用什么图片加载的方式，都需要获取请求图片返回头`respond.headers` 中的Set-Cookie ;
这个是登录验证你输入的验证码是否正确的关键 !

示例代码（java）：
```java
//获取返回头
Headers headers = response.headers();

//获取Cookie
// ASP.NET_SessionId=qagii0l2jz2elindssl4ptky; path=/; HttpOnly
List<String> cookies = headers.values("Set-Cookie");
String session = cookies.get(0);

//获取Sessionld,即Cookie中有效的部分：
//ASP.NET_SessionId=qagii0l2jz2elindssl4ptky                     
 auth_Cookie = session.substring(0, session.indexOf(";"));                                          
```

返回头参数示例：
> Cache-Control: private
                                                                                 Content-Type: image/jpeg
                                                                                 Server: Microsoft-IIS/7.5
                                                                                 Set-Cookie: ASP.NET_SessionId=qagii0l2jz2elindssl4ptky; path=/; HttpOnly
                                                                                 X-AspNetMvc-Version: 2.0
                                                                                 X-AspNet-Version: 4.0.30319
                                                                                 X-Powered-By: ASP.NET
                                                                                 Date: Fri, 17 Nov 2017 02:52:51 GMT
                                                                                 Content-Length: 5085

> 其中 Set-Cookie: ASP.NET_SessionId=qagii0l2jz2elindssl4ptky; path=/; HttpOnly
> 就是我们需要的Sessionld！

****
#### 1.2 登录请求
**请求方式：** POST  
**二级URL：**/Account/Login
> 示例：http://jwkq.xupt.edu.cn:8080/Account/Login

**请求参数：**
请求头：

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| Cookie|   String|  验证码Session |	ASP.NET_SessionId=qagii0l2jz2elindssl4ptky|

请求Body（Json格式提交）：
需要把下列Body参数转换为Json格式上传：
==注意：编码方式为： UTF-8==

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| UserName|   String|  学号| 	04151014	|
| UserPassword|   String|  智慧教室密码|151014 |
| ValiCode|   String|  验证码|   00152 |

参数示例：
```json
{
    "UserName": "04151014",
    "UserPassword": "151014",
    "ValiCode": "00152"
}
```

示例代码（java）：
```java
...
JSON = MediaType.parse("application/json; charset=utf-8");
JSONObject json = new JSONObject();
json.put("UserName", accountEdit.getText().toString().trim());
json.put("UserPassword", passwordEdit.getText().toString().trim()); 
json.put("ValiCode", authcodeEdit.getText().toString().trim()); 
...
//请求的Body
RequestBody body = RequestBody.create(JSON, json.toString());             
```
**请求代码示例（java）：**
```java
final Request request = new Request.Builder()
                 .addHeader("Cookie", auth_Cookie)
                 .url("http://jwkq.xupt.edu.cn:8080/Account/Login")
                 .post(body)
                 .build();
```

**返回参数：**
请求成功：返回Code = 200；

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| IsSucceed|   boolean|  是否登录成功| 	true，是判断是否成功登录的关键|
| Msg|   String|  请求的完成返回请求结果的说明| 	当请求成功的时候为null，错误的话显示相应的错误信息。|
| Obj|   Json|  登录完成后，返回的body| 登录成功后相应的返回，智慧教室登陆成功页面的一些数据信息。|

登录失败示例：
```json
{
    "IsSucceed": false,
    "Msg": "用户名或密码错误！",
    "Obj": 1000
}
```

登录成功返回头示例：
```json
{
    "IsSucceed": true,
    "Msg": null,
    "Obj": [
        {
            "BH": 164,
            "NAME": "雁塔校区18层楼",
            "ADDRESS": "",
            "MANAGERUSER": "",
            "MANAGERTEL": "",
            "AREA": "",
            "Flag": "1000000000",
            "LastUpdateTime": "2017-11-17 11:35:30"
        },
        {
            "BH": 166,
            "NAME": "长安校区西区基础教学B楼",
            "ADDRESS": "",
            "MANAGERUSER": "",
            "MANAGERTEL": "",
            "AREA": "",
            "Flag": "1000000000",
            "LastUpdateTime": "2017-11-17 11:35:29"
        },
        {
            "BH": 167,
            "NAME": "长安校区西区基础教学A楼",
            "ADDRESS": "",
            "MANAGERUSER": "",
            "MANAGERTEL": "",
            "AREA": "",
            "Flag": "1000000000",
            "LastUpdateTime": "2017-11-17 11:35:45"
        },
        {
            "BH": 163,
            "NAME": "长安校区东区逸夫楼",
            "ADDRESS": "",
            "MANAGERUSER": "",
            "MANAGERTEL": "",
            "AREA": "",
            "Flag": "1000000000",
            "LastUpdateTime": "2017-11-17 11:35:29"
        },
        {
            "BH": 165,
            "NAME": "1号实验楼",
            "ADDRESS": "",
            "MANAGERUSER": "",
            "MANAGERTEL": "",
            "AREA": "",
            "Flag": "0000000000",
            "LastUpdateTime": "2017-09-08 08:41:53"
        },
        {
            "BH": 162,
            "NAME": "西区",
            "ADDRESS": "",
            "MANAGERUSER": "",
            "MANAGERTEL": "",
            "AREA": "",
            "Flag": "0000000000",
            "LastUpdateTime": "2017-06-30 16:08:38"
        },
        {
            "BH": 181,
            "NAME": "2号实验楼",
            "ADDRESS": "",
            "MANAGERUSER": "",
            "MANAGERTEL": "",
            "AREA": "",
            "Flag": "0000000000",
            "LastUpdateTime": "2017-04-07 11:38:17"
        }
    ]
}
```
####	必须从返回头中获取的参数：登录成功的 seccessfulCookie 
> 只有获取到登录成功的 Cookie ，才能再后续的请求操作中，获取其他数据.

==该 Cookie 需要包括 验证码 Cookie 的内容==

示例代码（java）：
```java
Headers headers = response.headers();

  //获取登录成功的Cookie
  List<String> cookies = headers.values("Set-Cookie");
   String login_cookie = cookies.get(0);

//最后的 Cookie 为  验证码Cookie + 登录成功返回Coo
 seccessfulCookie = auth_Cookie+"; "
 +login_cookie.substring(0, login_cookie.indexOf(";"));
```

****

### 二 . 请求课程表
**二级URL：** /User/GetStuClass
> 示例：http://jwkq.xupt.edu.cn:8080/User/GetStuClass

**请求参数：**
请求头：
+ URL ： http://jwkq.xupt.edu.cn:8080/User/GetStuClass
+ Cookie :  seccessfulCookie

请求Body（表单形式提交）：


| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| term_no|   String|  查询的时间点为哪一个学期| 2017-2018-1|
| week|   String|  查询的课表为本学期的第几周| 11|
| json|   boolean|  设置返回json数据| 	true|


示例代码（java）：
```java
RequestBody body = new FormBody.Builder()
                  .add("term_no", session)
                  .add("week",String.valueOf(week))
                  .add("json", "true")
                  .build();
                  
  Request request = new Request.Builder()
				 .url("http://jwkq.xupt.edu.cn:8080/User/GetStuClass")
                 .addHeader("Cookie", seccessfulCookie)
                 .post(body)
                 .build();
```

**返回参数:**


| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| IsSucceed|   boolean|  是否查询成功| 	true|
| Msg|   String|  请求的完成返回请求结果的说明| 	当请求成功的时候为null，错误的话显示相应的错误信息。|
| Obj|   Json|  登录完成后，返回的body| 登录成功后相应的返回，课表的Json格式|


返回的Json格式课表信息参数解析：
```json
{
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 3,
            "JT_NO": "3-4",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "基于Verilog的FPGA设计基础",
            "S_Code": "JS110010",
            "Teach_Name": "董梁",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
             }
```
主要参数：


| 参数名      |  说明| 示例|
| :-------- |:------ |:------ |
| WEEKNUM| 表示星期几| 	3|
| JT_NO| 表示第几节上课| 	3-4|
| R_BH| 表示教室的可容纳人数| 	204|
| Sno| 查询者的学号| 	04151014|
| S_Name| 课程名称| 	基于Verilog的FPGA设计基础|
| S_Code| 课程的编号| 	JS110010|
| Teach_Name| 课任老师的名字| 	董梁|
| RoomNum| 上课的教室| FF305|
| B_Name| 上课的教学楼 | 	长安校区东区逸夫楼|


返回数据示例：
```json
{
    "IsSucceed": true,
    "Msg": null,
    "Obj": [
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 3,
            "JT_NO": "3-4",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "基于Verilog的FPGA设计基础",
            "S_Code": "JS110010",
            "Teach_Name": "董梁",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 1,
            "JT_NO": "5-6",
            "R_BH": 203,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "软件工程B",
            "S_Code": "JS102110",
            "Teach_Name": "曹小鹏",
            "RoomNum": "FF303",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 2,
            "JT_NO": "5-6",
            "R_BH": 203,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "马克思主义基本原理概论",
            "S_Code": "RW100040",
            "Teach_Name": "贺红霞",
            "RoomNum": "FF303",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 1,
            "JT_NO": "3-4",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "基于Verilog的FPGA设计基础",
            "S_Code": "JS110010",
            "Teach_Name": "董梁",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 3,
            "JT_NO": "1-2",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "计算机网络安全技术B",
            "S_Code": "JS130011",
            "Teach_Name": "王小耿",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 3,
            "JT_NO": "5-6",
            "R_BH": 203,
            "W_FLAG": 1,
            "Sno": "04151014",
            "S_Name": "Java语言程序设计B",
            "S_Code": "JS100042",
            "Teach_Name": "朱晓龙",
            "RoomNum": "FF303",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 5,
            "JT_NO": "5-6",
            "R_BH": 203,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "Java语言程序设计B",
            "S_Code": "JS100042",
            "Teach_Name": "朱晓龙",
            "RoomNum": "FF303",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 4,
            "JT_NO": "1-2",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "嵌入式系统原理与应用A",
            "S_Code": "JS100383",
            "Teach_Name": "李有谋",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 4,
            "JT_NO": "5-6",
            "R_BH": 186,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "Windows编程",
            "S_Code": "JS100130",
            "Teach_Name": "王文浪",
            "RoomNum": "FF203",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 5,
            "JT_NO": "1-2",
            "R_BH": 203,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "计算机专业英语",
            "S_Code": "JS100701",
            "Teach_Name": "贾晖",
            "RoomNum": "FF303",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 2,
            "JT_NO": "1-2",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "数据库原理及应用A",
            "S_Code": "JS100491",
            "Teach_Name": "尚小林",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 4,
            "JT_NO": "3-4",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "数据库原理及应用A",
            "S_Code": "JS100491",
            "Teach_Name": "尚小林",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        },
        {
            "Statmp": null,
            "BH": 0,
            "TERM_NO": null,
            "CLASS_NO": null,
            "WEEK": null,
            "WEEKNUM": 2,
            "JT_NO": "3-4",
            "R_BH": 204,
            "W_FLAG": 0,
            "Sno": "04151014",
            "S_Name": "嵌入式系统原理与应用A",
            "S_Code": "JS100383",
            "Teach_Name": "李有谋",
            "RoomNum": "FF305",
            "B_Name": "长安校区东区逸夫楼",
            "CalerdarId": null
        }
    ]
}
```

