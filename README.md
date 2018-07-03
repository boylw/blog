# blogs
# Node-blog
## 业务目标
  + 实现登陆注册服务端验证
  + 实现用户访问权限限制
  + 实现与mongodb数据库交互
  + 实现用户发表，查看，删除，评论等操作
  + 实现管理员对普通用户权限控制操作
## 路由设计
| url       | 提交方式 | get参数 | post参数 | 是否登陆  | 备注         |
| --------- | -------- | ------- | -------- | --------- | ------------ |
| /         | get      | null    | null     | no or yes | 渲染主页     |
| /register | get      | null    | null     | no        | 渲染注册页面 |
| /register | post     | null    | null     | no        | 处理注册页面 |
| /login    | get      | null    | null     | no        | 渲染登陆页面 |
| /login    | post     | null    | null     | no        | 处理登陆页面 |
| /loginout | get      | null    | null     | yes       | 退出登陆处理 |

### 1. 登陆注册服务端验证

1. 对`form` 表单进行异步提交

2. 后台接收数据，进行逻辑处理
   - 注册逻辑处理
     + 首先判断是否已经注册
     + 错误状态码进行返回
     + 前台接受状态码，进行相应操作
   - 登陆逻辑处理
     + 判断用户是否存在
     + 账号密码检测

3. session and cookie
    + name=value：键值对，可以设置要保存的 Key/Value，注意这里的 name 不能和其他属性项的名字一样
    + Expires： 过期时间（秒），在设置的某个时间点后该 Cookie 就会失效，如 expires=Wednesday, 09-Nov-99 23:12:40 GMT
    + maxAge： 最大失效时间（毫秒），设置在多少后失效
    + secure： 当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效
    + Path： 表示 cookie 影响到的路，如 path=/。如果路径不能匹配时，浏览器则不发送这个Cookie
    + httpOnly：是微软对COOKIE做的扩展。如果在COOKIE中设置了“httpOnly”属性，则通过程序（JS脚本、applet等）将无法读取到COOKIE信息，防止XSS攻击产生

4. 设置cookie

    + 使用response.writeHead 

    ```javascript
    //设置过期时间为一分钟
    var today = new Date();
    var time = today.getTime() + 60*1000;
    var time2 = new Date(time);
    var timeObj = time2.toGMTString();
    response.writeHead({
       'Set-Cookie':'myCookie="type=ninja", "language=javascript";path="/";
       Expires='+timeObj+';httpOnly=true'
    });
    ```

    + 缺点：使用response.writeHead只能发送一次头部，即只能调用一次，且不能与response.render共存，否则会报错 
    + 使用response.cookie，代码示例如下： 

    ```javascript
    response.cookie('haha', 'name1=value1&name2=value2', {
    maxAge:10*1000, path:'/', httpOnly:true
    });
    ```

    + `语法: response.cookie(‘cookieName’, ‘name=value[name=value…]’,[options]); ` 

    

    ### Cookie 简单实用

    express 在 4.x 版本之后，管理session和cookies等许多模块都不再直接包含在express中， `而是需要单独下载安装相应模块。

    cookieParser安装：

    ```javascript
    $ npm install cookie-parser
    ```

    使用方法: 

    ```javascript
    var express      = require('express');
    var cookieParser = require('cookie-parser');
     
    var app = express();
    app.use(cookieParser());
     
    app.get('/', function (req, res) {
        // 检查 session 中的 isVisit 字段是否存在
        // 如果存在则增加一次，否则为 session 设置 isVisit 字段，并初始化为 1。
            req.cookie.isVisit = 1;
            res.send("欢迎第一次来这里");
            console.log("Cookies: ", req.cookies); //打印cookie
        }
    });
    app.listen(80);
    ```

    ### Session 使用

    + 安装

    ```shell
    npm install express-session
    ```

    + 配置

    ```javascript
    var express = require('express')
    var session = require('express-session')
    
    var app = express()
    app.use(session({
        secret : 'xxx' //为了安全，在前面默认加上私钥字符串
        resave : false,
        saveUninitialized : false //不管用不用session，都会给你发一把钥匙
    }))
    ```

    + 默认Session 数据是内存存储的，服务器一旦重启，就会丢失，后期进行session数据存储
    + 要想清除Session 只需要将req.session 设置为空即可
## 5. 登陆处理
   - 登陆状态码
     + 200 正常登陆
     + 201 存在输入错误
     + 500 服务器出现错误
   - 表单提交异步处理
    ```javacript
    $('#register_form').on('submit', function (e) {
          // 禁用默认状态
	      e.preventDefault()
          // 表单序列化
	      var formData = $(this).serialize()
	      $.ajax({
	        url: '/login',
	        type: 'post',
	        data: formData,
	        dataType: 'json',
	        success: function (data) {
                if (data.err_code == '500') {
                    alert('现在太忙啦，稍后来试试把~')
                }
                if (data.err_code === '201') {
                    alert('请检查登陆信息哦')
                }
                if (data.err_code === '200') {
                    window.location.href = '/'
                }
          }
      })
    })
    ```
## 6. 退出状态处理
   + loginOut 状态处理
   + 清空req.session
   + 重新渲染index.html
