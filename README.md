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
> Content-Type: image/jpeg
> Server: Microsoft-IIS/7.5
> Set-Cookie: ASP.NET_SessionId=qagii0l2jz2elindssl4ptky; path=/; HttpOnly
> X-AspNetMvc-Version: 2.0
> X-AspNet-Version: 4.0.30319
> X-Powered-By: ASP.NET
> Date: Fri, 17 Nov 2017 02:52:51 GMT
> Content-Length: 5085

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
    
    ]
}
```
### 三.请求考勤信息
**二级URL：**/User/GetAttendList
> 示例：http://jwkq.xupt.edu.cn:8080/User/GetAttendList


**请求方式：**POST

**请求头参数：**

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| Cookie|   String|  查询的凭证|seccessfulCookie|
| Content-Type|   String|  是在 HTTP头中一个非常重要的头标示, 他就像一个说明书,说明了服务端/或浏览器应该怎么处理这次请求|application/x-www-form-urlencoded; charset=UTF-8|

**请求body参数（表单格式）：**

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| WaterDate|   String|  查询考勤的时间间距，两个时间间距用‘a’进行分隔；若获取当天数据，提交{a}和{（今天时间）a（今天时间）}达到的效果一样！|2017-11-10a2017-11-17；|
| Status|   String|  对应Web端的时间期限；今天:Status=‘1’；近一周：Status=‘2’；近一月：Status=‘3’|2|
|Flag|String|对应Web页的考勤情况选项框[正常（1），迟到（2），缺勤（3）],当想查询多个考勤情况的时候，需要用‘a’进行间隔|	只查询考勤：3；查询正常和迟到：1a2；|
|page|String|表示web对应的表格显示第几页，默认为 1 即可|	1|
|rows|String| 对应Web页，当前表格显示的条目数量的最大值，不足最大值也就显示多少个，超过最大值就显示最大值，设置这个值的大小就是设置你想得到返回的最大条目数，网页默认值为10|	10|

**请求示例(java):**

```java
{
//请求需求 近一个月的 正常，迟到，缺勤的10条数据
//WaterDate=2017-10-18a2017-11-17&Status=3&Flag=1a2a3&page=1&rows=10
...
RequestBody requestBody = new FormBody.Builder()
          .add("WaterDate", WaterDate)  //WaterDate=2017-10-18a2017-11-17
          .add("Status", Status)       //Status=3
          .add("Flag", Flag)           //Flag=1a2a3
          .add("page", Page)           //page=1
          .add("rows", Rows)           //rows=10
          .build();

Request request = new Request.Builder()
  .url("http://jwkq.xupt.edu.cn:8080/User/GetAttendList")
 .addHeader("Cookie",seccessfulCookie)
 .addHeader("Content-Type","application/x-www-form-urlencoded; charset=UTF-8")
  .post(requestBody)
  .build();
  ...
  }
```
**返回参数：**

返回状态码：200

**部分参数解析：**

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| total|   int|  若查询成功，该值必定大于0，返回的数量值就是查询到的所有数目|		查询成功：58；查询失败：0|
| rows|   Json|  查询得到每一项的详细数据，返回是Json数组的格式，|见示例|

上述请求返回示例;
```json
{
    "total": 58,
    "rows": [
        {
            "BH": 0,
            "Class_No": 256725,
            "JC": 0,
            "Sno": "04151014",
            "Status": 3,
            "OperNo": null,
            "WaterDate": "2017-10-18",
            "Week": 0,
            "Day": 0,
            "RBH": 204,
            "SBH": 98134,
            "EqNo": null,
            "Flag": "1",
            "S_BH": 0,
            "Term_No": "2017-2018-1",
            "JT_No": "1-2",
            "S_Code": "JS130011",
            "S_Name": "计算机网络安全技术B",
            "RoomNum": "FF305",
            "Name": "吴均斌",
            "BName": "长安校区东区逸夫楼",
            "PhotoName": null,
            "DepartmentName": null,
            "ZY": null,
            "Class": null,
            "Pid": null,
            "XY": null,
            "StuSex": null,
            "StuGrade": null,
            "CourseName": null,
            "OperTime": null,
            "Count": null,
            "Order": null,
            "WaterTime": "",
            "CardID": null,
            "WaterType": null
        },
        {
            "BH": 0,
            "Class_No": 256757,
            "JC": 0,
            "Sno": "04151014",
            "Status": 3,
            "OperNo": null,
            "WaterDate": "2017-10-18",
            "Week": 0,
            "Day": 0,
            "RBH": 204,
            "SBH": 93501,
            "EqNo": null,
            "Flag": "1",
            "S_BH": 0,
            "Term_No": "2017-2018-1",
            "JT_No": "3-4",
            "S_Code": "JS110010",
            "S_Name": "基于Verilog的FPGA设计基础",
            "RoomNum": "FF305",
            "Name": "吴均斌",
            "BName": "长安校区东区逸夫楼",
            "PhotoName": null,
            "DepartmentName": null,
            "ZY": null,
            "Class": null,
            "Pid": null,
            "XY": null,
            "StuSex": null,
            "StuGrade": null,
            "CourseName": null,
            "OperTime": null,
            "Count": null,
            "Order": null,
            "WaterTime": "",
            "CardID": null,
            "WaterType": null
        },
        ....
        ]
    }
```

**rows中请求结果主要参数说明：**

| 参数名      |  说明| 示例|
| :-------- |:------ |:------ |
| Class_No| 表示考勤库里的课程号| 	256757|
| Status| 表示此次刷卡的状态：1，正常；2，迟到；3，缺勤| 	3 |
| Sno| 表示刷卡者的学号| 	04151014|
| Name| 表示刷卡者的姓名| 吴均斌|
| S_Name| 课程名称| 	基于Verilog的FPGA设计基础|
| WaterDate| 表示上课的时间| 	2017-10-18|
| WaterTime| 表示刷卡的时间| 	2017-10-18 14:29:03|
| Term_No| 表示上课的学期| 	2017-2018-1|
| RoomNum| 上课的教室| FF305|
| BName| 上课的教学楼 | 	长安校区东区逸夫楼|
| JT_No| 表示上课的时间，一天之中的第几节课|	3—4|
| S_Code| 表示该课程的代号| JS110010|

###  四.请求所有科目的考勤情况
> 该请求是请求每门科目的正常签到，迟到和缺勤的次数

**二级URL：** User/GetAttendRepList"
> 示例：http://jwkq.xupt.edu.cn:8080/User/GetAttendRepList

**请求方式：** POST

**请求头参数：**
+ Cookie = seccessfulCookie

**请求Body（表单形式）：**
+ json = true

示例代码（java）：
```java
{
...
RequestBody body = new FormBody.Builder()
  .add("json", "true")
    .build();
    
final Request request = new Request.Builder()
		.url("http://jwkq.xupt.edu.cn:8080/User/GetAttendRepList")
        .header("Cookie", seccessfulCookie)
        .build();
        ....
}

```

**返回参数：**

部分参数解析：

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| IsSucceed|   boolean|  作为查询成功与否的标准，true 为成功，false 为失败|true|
| Msg|   String |  查询返回的提示消息，当请求失败的时候，会有相应的报错信息；若请求成功，则为空值 |null|
| Obj| json数组|	查询得到每一项的详细数据，返回是Json数组的格式，|见示例|

示例：
```json
{
    "IsSucceed": true,
    "Msg": null,
    "Obj": [
        {
            "Total": 16,
            "ShouldAttend": 11,
            "Attend": 7,
            "Late": 0,
            "Leave": 0,
            "Absence": 4,
            "Attendance": 0,
            "Secion": null,
            "SecionId": null,
            "CourseNo": "JS100701",
            "CourseName": "计算机专业英语",
            "RankNo": null,
            "WDate": null,
            "Class": null,
            "Name": null,
            "Sex": null,
            "Sno": "04151014",
            "Status": null,
            "ClassRoom": null,
            "ClassNo": null,
            "Nj": null,
            "Build": null,
            "BuildName": null,
            "RoomNo": null,
            "RoomNum": null,
            "Teacher": null,
            "TeacherName": null,
            "DepartmentName": null,
            "DepartmentCode": null,
            "Zy": null,
            "PhotoName": null,
            "ZyMc": null,
            "Week": 0,
            "WeekNum": 0,
            "Term_No": null
        },
        {
            "Total": 15,
            "ShouldAttend": 10,
            "Attend": 7,
            "Late": 0,
            "Leave": 0,
            "Absence": 3,
            "Attendance": 0,
            "Secion": null,
            "SecionId": null,
            "CourseNo": "JS102110",
            "CourseName": "软件工程B",
            "RankNo": null,
            "WDate": null,
            "Class": null,
            "Name": null,
            "Sex": null,
            "Sno": "04151014",
            "Status": null,
            "ClassRoom": null,
            "ClassNo": null,
            "Nj": null,
            "Build": null,
            "BuildName": null,
            "RoomNo": null,
            "RoomNum": null,
            "Teacher": null,
            "TeacherName": null,
            "DepartmentName": null,
            "DepartmentCode": null,
            "Zy": null,
            "PhotoName": null,
            "ZyMc": null,
            "Week": 0,
            "WeekNum": 0,
            "Term_No": null
        },
          ...
       ]
}
```

**Obj中的Json数组请求结果主要参数说明：**

| 参数名      |  说明| 示例|
| :-------- |:------ |:------ |
| Sno| 表示刷卡者的学号| 	04151014|
| CourseName| 课程的名称| 软件工程B|
| CourseNo| 课程的编号| JS102110|
| Total| 表示这门课程在本学期中的上课次数| 	15|
| ShouldAttend| 表示该门课程目前为止应该出席的次数，也就是到目前为止上了几次课| 	10 |
| Attend| 表示该门课程正常刷卡了几次，也就是出席了几次| 7| 
| Late| 表示该门课程上课迟到了几次| 0|
| Absence| 表示该门课程缺勤了几次| 3|

### 五.提交申诉
**二级URL：** /Apply/ApplyData
> 示例：http://jwkq.xupt.edu.cn:8080/Apply/ApplyData

**请求方式：**POST

**请求头参数**

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| Cookie|   String|  查询的凭证|seccessfulCookie|
| Host| String| 一台网络服务器上面可以放成千上万个网站（虚拟主机），当对这些网站的请求到来时，服务器能够根据Host这一行中的值来确定本次请求的是哪个具体的网站| jwkq.xupt.edu.cn:8080|
| Accept| String| 表示浏览器支持的 MIME 类型| application/json, text/javascript, */*; q=0.01|
| Origin| String| 表明该请求来源自于哪里 | http://jwkq.xupt.edu.cn:8080|
| Referer| String| Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的，服务器基此可以获得一些信息用于处理| http://jwkq.xupt.edu.cn:8080/User/Query|
| Content-Type|   String|  是在 HTTP头中一个非常重要的头标示, 他就像一个说明书,说明了服务端/或浏览器应该怎么处理这次请求|application/x-www-form-urlencoded; charset=UTF-8|

**请求的Body（Json形式）：**

示例：
```json
{
    "Remark": "没带卡，点名的时候到了",
    "A_Status": "1",
    "Class_no": "256686",
    "Jc": "1-2",
    "R_BH": "203",
    "S_Code": "96789",
    "S_Date": "2017-11-10",
    "S_Status": "3",
    "Term": "2017-2018-1"
}
```

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| Remark| String | 申诉者输入的申诉理由，必须要不少于5个字| 由于今天忘记带一卡通，导致打卡失败|
| A_Status| String| 申诉成什么状态，1.正常，2.迟到，3.缺席| 默认为：1|
| Class_no| String| 表示考勤库里的课程号| 从需要申诉的课程中获取：256686|
| Jc| String| 表示上课的时间，一天之中的第几节课 | 从需要申诉的课程中获取：1-2 |
| R_BH| String| 教室容纳量 | 从需要申诉的课程中获取：203 |
| S_Code| String| 教室编号 |  从需要申诉的课程中获取：96789|
| S_Date| String| 该课程上课的时间| 从需要申诉的课程中获取：2017-11-10|
| S_Status| String| 未申诉前课程状态，1.正常，2.迟到，3.缺席| 从需要申诉的课程中获取：3|
| Term| String| 申诉课程所处的学期| 从需要申诉的课程中获取：2017-2018-1|

请求示例（java）：
```java
{
...
JSON = MediaType.parse("application/json; charset=utf-8");
                    JSONObject json = new JSONObject();
                    json.put("Remark", Msg);
                    json.put("A_Status", "1");
                    json.put("Class_no", data.getClass_No());
                    json.put("Jc", data.getJT_No());
                    json.put("R_BH",data.getRBH());
                    json.put("S_Code",data.getSBH());
                    json.put("S_Date", data.getWaterDate());
                    json.put("S_Status",data.getStatus());
                    json.put("Term", data.getTerm_No());
         RequestBody body = RequestBody.create(JSON, json.toString());
                

    Request request = new Request.Builder()
       .url("http://jwkq.xupt.edu.cn:8080/Apply/ApplyData")
      .addHeader("Cookie", seccessfulCookie)
    .addHeader("Host", "jwkq.xupt.edu.cn:8080")
   .addHeader("Accept","application/json, text/javascript, */*; q=0.01")
  .addHeader("Origin","http://jwkq.xupt.edu.cn:8080")
 .addHeader("Content-Type","application/x-www-form-urlencoded; charset=UTF-8")                    
 .addHeader("Referer","http://jwkq.xupt.edu.cn:8080/User/Query")
         .post(body)
         .build();
...
}
```

**返回参数：**

部分参数解析：

| 参数名      |     类型|   说明| 示例|
| :-------- | :--------| :------ |:------ |
| IsSucceed|   boolean|  作为查询成功与否的标准，true 为成功，false 为失败|true|
| Msg|   String |  查询返回的提示消息，当请求失败的时候，会有相应的报错信息；若请求成功，则为空值 |申诉成功：null；已经申诉过：该记录已被申请过|
| Obj| json数组|	查询得到每一项的详细数据，返回是Json数组的格式，|见示例|

**Obj中的主要参数解析和请求Body类似**
示例：
```json
{
    "IsSucceed": true,
    "Msg": null,
    "Obj": {
        "Class_no": 256686,
        "S_Date": "2017-11-10",
        "Jc": "1-2",
        "S_Code": "96789",
        "R_BH": 203,
        "Term": "2017-2018-1",
        "Remark": "没带卡，点名的时候到了",
        "S_Status": 3,
        "A_Status": 1,
        "BH": 0,
        "Sno": "04151014",
        "Apply_Date": null,
        "Audit_Date": null,
        "Audit_User": null,
        "Status": 0,
        "A_Remark": null,
        "Name": null,
        "Sex": 0,
        "Department": null,
        "S_Name": null,
        "SBH": null,
        "RName": null,
        "BName": null,
        "Flag": 0
    }
}
```


