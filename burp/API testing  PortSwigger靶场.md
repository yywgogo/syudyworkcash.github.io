# API testing | PortSwigger靶场

# 实验一：Exploiting an API endpoint using documentation

To solve the lab, find the exposed API documentation and delete carlos. You can log in to your own account using the following credentials: wiener:peter.

## 方法一：

使用wiener:peter账号登录后，选择更新email，同时使用burpsuite抓包
在http记录中看到此api接口 /api/user/wiener ；

结合题目需要删除carlos`，将wiener换成carlos``；`

![img](https://i-blog.csdnimg.cn/blog_migrate/6808099e915e295ec3d3d8dd0995444a.png)

但是只能看到用户名和邮箱，因此查找别的接口；访问/api看到此处为api文档；

注：以下三个接口通常为api文档

- `/api`
- `/swagger/index.html`
- `/openapi.json`

![img](https://i-blog.csdnimg.cn/blog_migrate/20a44b015f1cc90dc8bf9787e8f0caa7.png)

选择删除carlos，通过第一关；

![img](https://i-blog.csdnimg.cn/blog_migrate/a255387bc1eaead14ab590268d6d9f2b.png)

## 方法二：

转入重放器，修改为 DELETE api/user/carlos 重放即可

# 实验二：Finding and exploiting an unused API endpoint

为了解决实验室问题，利用隐藏的 API 端点购买**轻质 l33t 皮夹克**。您可以使用以下凭据登录自己的帐户： `wiener:peter`。

#### 所需知识

要解决这个实验，你需要知道：

- 如何使用错误消息来构造有效的请求。

- RESTful API 如何使用 HTTP 方法。

- 如何通过改变 HTTP 方法来揭示附加功能。

  找到 /api/products/1/price 这个api；

  ![img](https://i-blog.csdnimg.cn/blog_migrate/b35dd6d4f179bf285baf385c6f9294b7.png)

  使用POST进行尝试，从回包看出其只支持GET和PATCH;

  ![img](https://i-blog.csdnimg.cn/blog_migrate/7c24e63b1577a028e10d9390f7dfac0b.png)

  尝试使用PATCH，发现Content-type需要是application/json；

  ![img](https://i-blog.csdnimg.cn/blog_migrate/0a241f9269e958643a87d753492c9ebd.png)

  将其修改一下但是回包没什么信息；

  ![img](https://i-blog.csdnimg.cn/blog_migrate/df2063390b226235395f8aea0746200e.png)

  请求包中添加 {"price":0} ，看到回包中显示价格为0；然后在首页中看到商品变为0元；

  ![img](https://i-blog.csdnimg.cn/blog_migrate/fa0a2cc88ff268a1aea164f12d57200d.png)

  ![img](https://i-blog.csdnimg.cn/blog_migrate/65acb903e6a3099d1f8234d7d84e6f1a.png)

  加入购物车并购买后，通过第三关。

  ![img](https://i-blog.csdnimg.cn/blog_migrate/3df43f1a843229b908a1631c5551f8fc.png)

  

# 实验三：Exploiting a mass assignment vulnerability

## 批量分配漏洞

批量分配（也称为自动绑定）可能会无意中创建隐藏参数。当软件框架自动将请求参数绑定到内部对象的字段时，就会发生这种情况。因此，批量分配可能会导致应用程序支持开发人员从未打算处理的参数。

要解决实验室问题，请找到并利用批量分配漏洞来购买**轻量级 l33t 皮夹克**。您可以使用以下凭据登录自己的帐户：`wiener:peter`。

#### 所需知识

要解决这个实验，你需要知道：

- 什么是批量分配。

- 为什么批量分配可能会导致隐藏参数。

- 如何识别隐藏参数。

- 如何利用大规模分配漏洞。按惯例先尝试所有功能，然后查看burpsuite的http history
  发现两个有意思的请求：`POST /api/checkout HTTP/2`和`GET /api/checkout HTTP/2`
  点击付款按钮时会先发送POST请求，数据为购买的商品的列表，包括商品id和数量

  ```prolog
  {"chosen_products":[{"product_id":"1","quantity":1}]}
  ```

  GET请求则会返回该次交易的相关信息，如下图：
  ![l4-s1.png](https://segmentfault.com/img/remote/1460000045059538)
  第一个尝试还是先修改商品的价格，发现没有成功
  ![l4-s2.png](https://segmentfault.com/img/remote/1460000045059539)
  然后修改代表折扣的字段，发送请求后就直接成功购买了
  ![l4-s3.png](https://segmentfault.com/img/remote/1460000045059540)

# 实验四：xploiting server-side parameter pollution in a query string

要解决实验室问题，请以 身份登录`administrator`并删除`carlos`。

#### 所需知识

要解决这个实验，你需要知道：

- 如何使用 URL 查询语法尝试改变服务器端请求。
- 如何使用错误消息来了解服务器端 API 如何处理用户输入。

**参数污染**：某些服务器或应用程序会接受同名的多个参数，但不清楚如何处理它们。这时候，参数的不同处理方式就可能带来问题。例如，服务器可能会选择第一个或最后一个参数，或将它们合并。

**实验目标**：在此实验中，你需要发现并利用这种处理方式来操控服务器的行为。通过发送多个同名参数，你可以改变服务器的操作逻辑，达到执行未授权操作的效果。

**典型步骤**：

- 首先分析应用的处理逻辑，找出可以注入多个同名参数的地方。
- 然后，尝试发送多个相同参数，观察服务器如何处理。
- 最后，利用这种行为绕过某些限制，可能获得敏感数据或执行未经授权的操作。

浏览站点，使用忘记密码功能进行密码重置，输入administrator，使用burp 代理历史查看数据包，查看到忘记密码的数据包，查看739,740,741三个与忘记密码操作相关的数据包：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c22b45e4f089f33eb6e934835fd96525.png)
对username参数进行测试：尝试参数值：administrator，adminstratorx(一个大概率不存在的参数)，查看响应包：
Invalid username,这说明了administrator用户名存在，administratorx用户名不存在，可以用来枚举，但是这里的目的不是枚举用户名。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bca4930cb9e5eb55b9c11407cd4bef86.png)
构造其他参数进行测试，测试服务是否接受其他参数，如添加【&需要使用url编码】：&x=y，username=administrator%26x=y
从响应结果看出服务大概率是接受多个参数，但是参数x是错误不支持的。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a3d3f05a8d0bd63d86156ab275b88245.png)
测试：username=administrator#【#号使用url编码】，username=administrator%23
使用#截断参数测试，发现了参数:Fileld
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7e455595fedf9b5adf8d50f8408deed3.png)
测试：&field=x#【&与#需要url编码】，username=administrator%26field=x%23
这个payload指定一个fileld参数的值
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec8a5a1197e60d8fad811ca7d264c978.png)
接下来，对field参数的值进行爆破：
在“入侵者”>“有效负载”中，单击“从列表添加”。选择内置的服务器端变量名负载列表，然后开始攻击。
查看结果。请注意，带有用户名和电子邮件负载的请求都会返回响应200
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63cc7e439ff899118c2358275f28ee78.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2be92eb0711853d24b15d44f2ed75ea5.png)
爆破成功：得到一个email参数
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4bbf76bbe29fc00066abbf7d3e099f55.png)
在忘记密码的js中发现了，重置密码的url:/forgot-password?reset_token=${resetToken}
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/52bea4263051907a0d2ce3983af74890.png)
后续操作：
在“代理”>“HTTP 历史记录”中，查看/static/js/forgotPassword.jsJavaScript 文件。注意密码重置端点，它引用了 reset_token参数：

```
/forgot-password?reset_token=${resetToken}

在Repeater选项卡中，将参数值field从 更改email为reset_token：

username=administrator%26field=reset_token%23

发送请求。请注意，这会返回一个密码重置令牌，记下重置令牌：6ws18f561lomm78lgqn51skk22qaakjm

在 Burp 的浏览器中，在地址栏中输入密码重置端点。添加您的密码重置令牌作为参数的值 reset_token。例如：

/forgot-password?reset_token=123456789

设置新密码。

使用您的密码以用户身份登录administrator。

转到管理面板并删除carlos以解决实验。

123456789101112131415161718
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42f72209602d9a7860701721e32dfc13.png)
/forgot-password?reset_token=6ws18f561lomm78lgqn51skk22qaakjm
浏览器打开重置密码的url:url中附加管理员administrator的参数，重置密码为123test:
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea960afb3ad426e96c18e0c9cf466b74.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2de44cb0bad1d7c8703c93adf76a43f.png)

在这个实验中，关键是理解服务器如何处理 `field` 参数。当你发现了 `field=email` 参数时，服务器可能会根据它来决定返回哪些数据，比如用户的 email 地址。现在的目标是通过操控这个参数来获取更多有价值的信息，例如 `reset_token`，它可以用于重置管理员密码。

### 原理解释：

1. **字段参数的使用：**  
   `field` 参数告诉服务器你想要请求的字段数据。在这个例子中， `field=email` 表示你想要获得 `administrator` 用户的电子邮件地址。服务器在处理这个请求时，会返回与电子邮件相关的信息。

2. **修改为 `reset_token`：**  
   在 JavaScript 文件 `/static/js/forgotPassword.js` 中，引用了 `reset_token`，这说明 `reset_token` 是用于密码重置的关键参数。如果你能够请求 `reset_token` 而不是 `email`，你可能会得到管理员的密码重置令牌，从而可以控制账户。

   你可以通过如下方式来操控这个请求：
   
   ```bash
   csrf=YIVDEo3J4g6PgHd3APljLZSwhr181CGs&username=administrator%26field=reset_token%23
   ```

3. **为什么需要这样更改：**  
   在服务器端，通常会对参数进行过滤或分类。如果你没有直接的访问权限，但服务器允许你使用 `field` 参数请求特定字段数据（如电子邮件），那么你可以尝试通过修改 `field` 的值来请求其他敏感数据（如 `reset_token`）。这是一种参数污染攻击，通过精心构造的请求，可以绕过服务器的验证逻辑，访问到本不应该暴露的数据。

4. **#符号的作用**：  
   在 URL 编码中，`%26` 代表 `&`，`%23` 代表 `#`。在这个请求中，`#` 通常表示注释或结束符，用来终止后续的参数解释。所以 `username=administrator%26field=reset_token%23` 实际上会被解释为两个参数：
   - `username=administrator`
   - `field=reset_token`

   这样服务器会以为你请求的是 `administrator` 用户的 `reset_token` 字段，而不会继续处理后面的参数，达到了污染参数的效果。

# 实验五：REST URL服务端参数污染：

Lab: Exploiting server-side parameter pollution in a REST URL

https://portswigger.net/web-security/api-testing/server-side-parameter-pollution/lab-exploiting-server-side-parameter-pollution-in-rest-url 未完成

进入靶场：点击功能，密码重置功能，尝试重置administrator用户的密码，进入burp查看代理历史，关注到下面两个流量包：

##### 判断行为：

```
在 Burp 的浏览器中，触发用户密码重置administrator。

在“代理”>“HTTP 历史记录”中，注意POST /forgot-password请求和相关的 /static/js/forgotPassword.jsJavaScript 文件。

右键单击该POST /forgot-password请求并选择“发送到转发器”。

在中继器选项卡中，重新发送请求以确认响应是否一致。

发送各种带有修改后的username参数值的请求，判断输入是否放在服务器端请求的URL路径中而不进行转义：

提交 URL 编码administrator#作为参数的值username。

请注意，这会返回一条Invalid route错误消息。这表明服务器可能已将输入放置在服务器端请求的路径中，并且该片段已截断了一些尾随数据。请注意，该消息还引用了 API 定义。

将 username 参数的值从 更改administrator%23 为 URL-encoded administrator?，然后发送请求。

请注意，这还会返回一条Invalid route错误消息。这表明输入可能被放置在 URL 路径中，因为该?字符指示查询字符串的开头，因此会截断 URL 路径。

将参数值username从 更改administrator%3F为./administrator然后发送请求。

请注意，这将返回原始响应。这表明该请求可能访问了与原始请求相同的 URL 路径。这进一步表明输入可以放置在 URL 路径中。

将 username 参数的值从 更改为./administrator ，../administrator然后发送请求。

请注意，这会返回一条Invalid route错误消息。这表明该请求可能访问了无效的 URL 路径。
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/93e0e6cba7753d70311d5eb0575ba58a.png)
根据响应判断服务端处理的模式：

```
1.提交 URL 编码administrator#作为参数的值username。

请注意，这会返回一条Invalid route错误消息。这表明服务器可能已将输入放置在服务器端请求的路径中，并且该片段已截断了一些尾随数据。请注意，该消息还引用了 API 定义。

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b22266bdddb22b0df68ee4240a084dbb.png)

```
2. 将 username 参数的值从 更改administrator%23 为 URL-encoded administrator?，然后发送请求。
请注意，这还会返回一条Invalid route错误消息。这表明输入可能被放置在 URL 路径中，因为该?字符指示查询字符串的开头，因此会截断 URL 路径
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9b709c8172a232918be059d44b6d2a3.png)

```
3.将参数值username从 更改administrator%3F为./administrator然后发送请求。
请注意，这将返回原始响应。这表明该请求可能访问了与原始请求相同的 URL 路径。这进一步表明输入可以放置在 URL 路径中。
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9747840a06e98a6f530f05184af64345.png)

```
4.将 username 参数的值从 更改为./administrator ，../administrator然后发送请求。

请注意，这会返回一条Invalid route错误消息。这表明该请求可能访问了无效的 URL 路径。
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8161321b483418d1787a6cafba5f6610.png)

##### 利用报错查询到敏感api接口：

```
## 利用：
将用户名参数的值从 更改../administrator为../%23。注意 Invalid route响应。

逐渐添加更多../序列，直到达到../../../../%23请注意，这会返回 Not found响应。这表明您已导航到 API 根目录之外。

在此级别，将一些常见的 API 定义文件名添加到 URL 路径。例如，提交以下内容：

username=../../../../openapi.json%23

请注意，这会返回一条错误消息，其中包含以下用于查找用户的 API 端点：

/api/internal/v1/users/{username}/field/{field}

请注意，此端点指示 URL 路径包含一个名为 的参数field。
```

##### 利用此漏洞

```
username使用已识别端点的结构 更新参数的值。为参数添加无效值 field：

username=administrator/field/foo%23

发送请求。请注意，这会返回一条错误消息，因为该 API 仅支持电子邮件字段。

添加email作为参数的值field：

username=administrator/field/email%23

发送请求。请注意，这将返回原始响应。这可能表明服务器端应用程序识别注入的field参数并且该参数 email是有效的字段类型。

在“代理”>“HTTP 历史记录”中，查看/static/js/forgotPassword.jsJavaScript 文件。识别密码重置端点，参考参数 passwordResetToken：

/forgot-password?passwordResetToken=${resetToken}

在Repeater选项卡中，将参数值field从更改email为passwordResetToken：

username=administrator/field/passwordResetToken%23

发送请求。请注意，这会返回一条错误消息，因为passwordResetToken应用程序设置的 API 版本不支持该参数。

使用/api/您之前确定的端点，更改参数值中的 API 版本username：

username=../../v1/users/administrator/field/passwordResetToken%23

发送请求。请注意，这会返回一个密码重置令牌。记下这一点。

在 Burp 的浏览器中，在地址栏中输入密码重置端点。添加您的密码重置令牌作为参数的值reset_token。例如：

/forgot-password?passwordResetToken=123456789

设置新密码。

使用您的密码登录administrator。

转到管理面板并删除carlos以解决实验。
```

填充无效的字段foo，查看报错：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce7f11fa67d1afe1d0a069251682baae.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54ce5904c0b521f161707d9a5821eb3b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6d2bf75f83aad31c90e1abc68e964488.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad9fb93155800ddc611ef0e1e162b389.png)

##### 浏览器修改漏洞：

使用泄露的token重置管理员密码：
https://0a3600be04d9717c818f8a7d00fc0045.web-security-academy.net/forgot-password?passwordResetToken=y6ig3hyvx0n4p1322r5j7fjqe38r1in8
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ebd57bff9f8fec3bffff2de3bd33a500.png)
修改完成后使用管理员用户登陆，进入管理面板，删除用户carlos,完成靶场：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a35a2d2a56ba53f8b3beef65303d6042.png)