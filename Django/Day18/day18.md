---
typora-root-url: ./asset
---

# day18

## 12.管理员操作

## 13.用户登录

http无状态短链接：超文本传输协议 无法区分用户身份和状态的

![image-20241012214615147](/../day18.assets/image-20241012214615147.png)

什么是cookie和session？

- cookie实际是一小段文本，是服务器端下发给客户端的，客户端下一次请求会携带上它，方便服务端识别状态和身份它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态，个性化设置等
- session（会话：用户与服务器之间的一次交互）是服务器端使用一种记录客户端状态的机制，是服务器端为客户端所开辟的**存储空间**

两者有什么不同？

- 作用范围不同，Cookie 保存在客户端（浏览器），Session 保存在服务器端。
- 存取方式的不同，Cookie 只能保存 ASCII，Session 可以存任意数据类型

- 有效期不同，Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能，Session 一般失效时间较短，客户端关闭或者 Session 超时都会失效。
- 安全性不同，Cookie存储在客户端，容易被获取，Session存储在服务器端，安全性较高

两者的关联？

- 浏览器是没有状态的，这意味着浏览器并不知道是张三还是李四在和服务端打交道。这个时候就需要有一个机制来告诉服务端，本次操作用户**是否登录**，是**哪个用户在执行的操作**，那这套机制的实现就需要 Cookie 和 Session 的配合。

### 13.1登录

登录成功后：

- cookie，随机字符串
- session，用户信息

在其他需要登录才能访问的页面中加入验证

```python
def index(request):
    info = request.session.get['info']
    if not info:
        return redirect('/login/')
    ....
```



目标：在视图函数前面统一加入判断。

```Python
info = request.session.get['info']
    if not info:
        return redirect('/login/')
```

### 13.2中间件的体验

![image-20241014173132080](/../day18.assets/image-20241014173132080.png)

- 定义中间件

  ```Python
  from django.utils.deprecation import MiddlewareMixin
  
  class M1(MiddlewareMixin):
      """中间件1"""
      def process_response(selfs,requset):
          print("M1,进来了")
          #没有返回值则默认为None，继续向后走
          #如果有返回值，直接返回了，不再继续
  
      def process_response(self,request,response):
          print("M1,走了")
          return response
  
  class M1(MiddlewareMixin):
      """中间件2"""
      def process_response(selfs,requset):
          print("M2,进来了")
  
      def process_response(self,request,response):
          print("M2,走了")
          return response
  ```

- 应用中间件 setting.py

  ```django
  MIDDLEWARE = [
      'django.middleware.security.SecurityMiddleware',
      'django.contrib.sessions.middleware.SessionMiddleware',
      'django.middleware.common.CommonMiddleware',
      'django.middleware.csrf.CsrfViewMiddleware',
      'django.contrib.auth.middleware.AuthenticationMiddleware',
      'django.contrib.messages.middleware.MessageMiddleware',
      'django.middleware.clickjacking.XFrameOptionsMiddleware',
      'app01.middleware.auth.M1',
      'app01.middleware.auth.M2',
  ]
  ```

### 13.3中间件实现登录

- 编写中间件

  ```django
  class AuthMiddleware(MiddlewareMixin):
      """中间件1"""
      def process_request(selfs,request):
          #0.排除不需要登录就可以访问的页面
          if request.path_info == "/login/":
              return None
          #没有返回值则默认为None，继续向后走
          #如果有返回值，直接返回了
          #1.读取当前访问用户的Session信息，如果有过登录过继续向下走
          info_dict = request.session.get('info')
          if info_dict:
              return
  
          #2 没有登录
          return redirect('/login/')
  ```


### 13.4注销

```Python
def logout(request):
    request.session.clear()
    return redirect('/login')

```

### 13.5当前用户

![image-20241014195123165](/../day18.assets/image-20241014195123165.png)

## 14图片验证码

### 14.1生成图片验证码

```python
pip install pillow
```

```python
def check_code(width=120, height=30, char_length=5, font_file='kumo.ttf', font_size=28):
    code = []
    img = Image.new(mode='RGB', size=(width, height), color=(255, 255, 255))
    draw = ImageDraw.Draw(img, mode='RGB')

    def rndChar():
        """
        生成随机字母
        :return:
        """
        return chr(random.randint(65, 90))

    def rndColor():
        """
        生成随机颜色
        :return:
        """
        return (random.randint(0, 255), random.randint(10, 255), random.randint(64, 255))

    # 写文字
    font = ImageFont.truetype(font_file, font_size)
    for i in range(char_length):
        char = rndChar()
        code.append(char)
        h = random.randint(0, 4)
        draw.text([i * width / char_length, h], char, font=font, fill=rndColor())

    # 写干扰点
    for i in range(40):
        draw.point([random.randint(0, width), random.randint(0, height)], fill=rndColor())

    # 写干扰圆圈
    for i in range(40):
        draw.point([random.randint(0, width), random.randint(0, height)], fill=rndColor())
        x = random.randint(0, width)
        y = random.randint(0, height)
        draw.arc((x, y, x + 4, y + 4), 0, 90, fill=rndColor())

    # 画干扰线
    for i in range(5):
        x1 = random.randint(0, width)
        y1 = random.randint(0, height)
        x2 = random.randint(0, width)
        y2 = random.randint(0, height)

        draw.line((x1, y1, x2, y2), fill=rndColor())

    img = img.filter(ImageFilter.EDGE_ENHANCE_MORE)
    return img, ''.join(code)

```

```python
def create_valimg(request):

    img,code_string = check_code()
    #写入内存
    stream = BytesIO()
    img.save(stream, 'png')
    return HttpResponse(stream.getvalue())
```

## 15ajax请求

浏览器向网站发送请求时：URL和表单的形式提交。

- GET
- POST

特点：页面会刷新。

除此之外，也可以基于Ajax向后台发送请求（偷偷的发送）



- 依赖jQuery

- 编辑ajax代码

  ```javascript
  $.ajax({
  	url:"发送的地址",
     	type:"post",
      data:{
          n1:123,
          n2:456
      },
      success:function(res){
          console.log(res);
      }
  })
  ```

### 15.1 发送GET请求

```javascript
$.ajax({
	url:"发送的地址",
   	type:"get",
    data:{
        n1:123,
        n2:456
    },
    success:function(res){
        console.log(res);
    }
})
```

```Python
def task_ajax(request):
	print(request.GET)
	return HttpResponse("成功了")
```

### 15.2 发送POST请求

```javascript
function clickMe(){
    $.ajax({
        url:"/task_ajax",
        type:"post",
        data:{
            n1:123,
            n2:456
        },
        success:function (res){
            console.log(11)
        }
    })
}
```

```Python
from django.views.decorators.csrf import csrf_exempt
@csrf_exempt
def task_ajax(request):
    print(request.GET)
    print(request.POST)
    return HttpResponse("成功了")
```

### 15.3 关闭绑定事件

```html
{% extends 'layout.html' %}>

{% block content %}
    <div class="container">
        <h1>任务管理</h1>

        <h3>示例1</h3>
        <input id="btn1" type="button" class="btn btn-primary" value="点击">
    </div>
{% endblock %}

{% block js %}
    <script type="text/javascript">
        $(function () {
            //页面加载后执行
            bindBtn1Event()

        })
        function bindBtn1Event() {
            $('#btn1').click(function () {
                $.ajax({
                    url: "/task_ajax/",
                    type: "post",
                    data: {
                        n1: 123,
                        n2: 456
                    },
                    success: function (res) {
                        console.log(11)
                    }
                })
            })
        }
    </script>
{% endblock %}
```

### 15.4 ajax请求的返回值

```html
{% extends 'layout.html' %}>
{% block content %}
    <div class="container">
        <h1>任务管理</h1>
        <h3>示例1</h3>
        <input id="btn1" type="button" class="btn btn-primary" value="点击">
    </div>
{% endblock %}
{% block js %}
    <script type="text/javascript">
        $(function () {
            //页面加载后执行
            bindBtn1Event()

        })
        function bindBtn1Event() {
            $('#btn1').click(function () {
                $.ajax({
                    url: "/task_ajax/",
                    type: "post",
                    data: {
                        n1: 123,
                        n2: 456
                    },
                    dataType:"JSON", #原来是字符串，自动转为json格式的
                    success: function (res) {
                        console.log(res)
                        console.log(111)
                        console.log(res.data)
                        console.log(res.status)
                    }
                })
            })
        }
    </script>
{% endblock %}
```

```Python
@csrf_exempt
def task_ajax(request):
    print(request.GET)
    print(request.POST)
    print(1111)
    data_dict = {"status":True,'data':[11,22,33,44]}
    return HttpResponse(json.dumps(data_dict))
```

```html
{% extends 'layout.html' %}>
{% block content %}
    <div class="container">
        <h1>任务管理</h1>
        <h3>示例1</h3>
        <input id="btn1" type="button" class="btn btn-primary" value="点击1">

        <h3>示例2</h3>
        <input type="text" id="textUser" placeholder="姓名">
        <input type="text" id="txtAge" placeholder="年龄">
        <input id="btn2" type="button" class="btn btn-primary" value="点击2">

        <h3>示例3</h3>
        <form id="form3">
            <input type="text" name="user" placeholder="姓名">
            <input type="text" name="age" placeholder="年龄">
            <input type="text" name="email" placeholder="邮箱">
            <input type="text" name="more" placeholder="个人简介">
            <input id="btn3" type="button" class="btn btn-primary" value="点击3">
        </form>
    </div>
{% endblock %}
{% block js %}
    <script type="text/javascript">
        $(function () {
            //页面加载后执行
            bindBtn1Event();
            bindBtn2Event();
            bindBtn3Event();

        })

        function bindBtn1Event() {
            $('#btn1').click(function () {
                $.ajax({
                    url: "/task_ajax/",
                    type: "post",
                    data: {
                        n1: 123,
                        n2: 456
                    },
                    dataType: "JSON",
                    success: function (res) {
                        console.log(res)
                        console.log(111)
                        console.log(res.data)
                        console.log(res.status)
                    }
                })
            })
        }

        function bindBtn2Event() {
            $('#btn2').click(function () {
                $.ajax({
                    url: "/task_ajax/",
                    type: "post",
                    data: {
                        user: $('#textUser').val(),
                        age: $('#txtAge').val()
                    },
                    dataType: "JSON",
                    success: function (res) {
                        console.log(res)
                        console.log(111)
                        console.log(res.data)
                        console.log(res.status)
                    }
                })
            })
        }

        function bindBtn3Event() {  #直接表单读取
            $('#btn3').click(function () {
                $.ajax({
                    url: "/task_ajax/",
                    type: "post",
                    data: $("#form3").serialize(),
                    dataType: "JSON",
                    success: function (res) {
                        console.log(res)
                        console.log(111)
                        console.log(res.data)
                        console.log(res.status)
                    }
                })
            })
        }
    </script>
{% endblock %}
```

