## Simulated Login：

为何我们要使用到Simulated Login（模拟登录）这个技术呢？很多情况下，页面的某些信息需要登录才可以查看 对于爬虫来说，需要爬取的信息如果需要登录才可以看到的话，那么我们就需要做一些模拟登录的事情。



库的准备：**requests库和lxml库**（没有的自行下载导入）



举例说明：如果在我们在 Github 上关注了某些人，在登录之后就会看到他 们最近的动态信息，比如他们最近收藏了哪个 Repository ，创建了哪个组织，推送了哪些代码 但是 出登录之后，我们就无法再看到这些信息,如果希望爬取 GitHub 所关注人的最近动态，我们就需要模拟登录 GitHub。

#### 分析登录的过程：

首先我们看一下GitHub的登录过程：

打开GitHub的登录界面，链接：https://github.com/login  输入 GitHub 的用户名和密码，打开开发者工具 （按f12，然后选择network那一项选项栏），将 Preserve Log 选项句选上，这表示显示持续日志。

点击登录按钮， 这时便会看到开发者工具下方显示了各个请求过程，点击名为session的目录（表示会话目录）可以看到请求的 URL https://github.com/session ，请求方式为 POST 再往下看，我们观察到它 Form Data 和Headers 这两部分内容。

Headers 里面包含了 Cookies、Host、Origin、Referer、User-Agent 等信息。Form Data 包含了5个字段， commit 是固定的字符串 Sign in, utf8 是一个勾选字符， authenticity_token 较长，其初步判断是 Base64 加密的字符串， login 是登录的用户名， password 是登录的密码 上所述，**我们现在无法直接构造的内容有 Cookies authenticity_token，所以我们需要单独构造cookies和authenticity_token。**（这一步很重要，所谓模拟登录的实质就是构建登录所需要的参数），下面直接上代码，代码中会有注释以及尾部会有总结。



```python
import requests
from lxml import etree


class Login(object):    ##构建登录的类，初始化
    def __init__(self):
        self.headers = {
            'Referer': 'https://github.com/',
            'User-Agent': '此处填写自己浏览器的User-Agent',     ##这一段是函数请求头
            'Host': 'github.com'
        }
        self.login_url = 'https://github.com/login'              ##登录的网址
        self.post_url = 'https://github.com/session'             ##登录参数传向的地址
        self.logined_url = 'https://github.com/settings/profile'  ##登录后请求的地址
        self.session = requests.Session()        ##用session来维护状态，储存cookies
    
    def token(self):
        response = self.session.get(self.login_url, headers=self.headers)
        selector = etree.HTML(response.text)
        token = selector.xpath('//div//input[2]/@value')     ##获取authenticity_token（检索网页代码可发现这个隐藏块，通过xpath定位，xpath后面会提到，可自行了解）
        return token
    
    def login(self, email, password):
        post_data = {
            'commit': 'Sign in',
            'utf8': '✓',
            'authenticity_token': self.token()[0],
            'login': email,
            'password': password
        }
        print(self.token())
        response = self.session.post(self.post_url, data=post_data, headers=self.headers)
        if response.status_code == 200:    
            print(111)      ##验证登陆成功
        
        response = self.session.get(self.logined_url, headers=self.headers)
        if response.status_code == 200:
            self.profile(response.text)    
 
    
    def profile(self, html):
        selector = etree.HTML(html)
        print(selector)
        name = selector.xpath('//input[@id="user_profile_name"]/@value')
        print(name)
        email = selector.xpath('//select[@id="user_profile_email"]/option[@value!=""]/text()')
        print(name, email)     ##验证登录成功，可以拿到登陆后才能拿到的数据


if __name__ == "__main__":
    login = Login()
    login.login(email='scauhui（这里写自己的账号）', password='这里写自己的密码')
```

实现效果如下图：

```
['a4Q+JWhsJhTinQy7l9bp+l7jIGzvmtyJm5+xi/bmNaOHO/IxqWL9apHZWOwcsukyIFP8RC3n4vBAOWNzwClDsg==']      
111       
<Element html at 0x15e4d3bbcc8>   
['scauhui']
['scauhui'] []
```



总结一下过程以及步骤顺序：

1. **首先分析登录前后参数的变化（哪些是需要构造的，通过开发者工具的network板块看）**
2. 想办法构造参数，上述例子中的cookies和token相对获取的较简单（看下注释）
3. 然后写登陆类以及登录函数
4. 验证登录



