## Ajax dynamically requests web content（Ajax动态获取网页内容）：

节前准备：**安装requests，urllib，pyquery库**



首先我们要知道Ajax是什么？为什么要用这个来做动态网页的爬取，你们现在爬取的多是固定的网页内容，请求后通过BeautifulSoup等库或者正则表达式进行内容提取，但是你们是否想过，如果像微博这种动态网页（比如你下拉式内容刷新但是网页不刷新，应该怎么提取呢？很显然，常规的这种直接Reqeusts到HTML内容再解析是没有用的，因为内容根本不在HTML上）

我们在用 requests 抓取页面的时候，得到的结果可能和在浏览器中看到的不一样：在浏览器中可以看到正常显示的页面数据，但是使用 requests 得到的结果并没有 这是因为 requests 获取的都是原始的 HTML 文档，而浏览器中的页面则是经过 JavaScript 处理数据后生成的结果，这些数据的来源有多种，可能是通过 Ajax 加载的， 可能是包含在 HTML 文档中的，也可能是经过 JavaScript 和特 定算法计算后生成的 对于第一种情况，数据加载是一种异步加载方式，原始的页面最初不会包含某些数据，原始页面 加载完后，会再向服务器请求某个接口获取数据，然后数据才被处理从而呈现到网页上，这其实就是 发送了一个 Ajax 请求 Web 发展的趋势来看，这种形式的页面越来越多 网页的原始 HTML 文档不会包含任何数据， 数据都是通过统一加载后再呈现出来的，这样在 Web开发上可以做到前后端分离，而且降低服务器直接渲染页面带来的压力 所以如果遇到这样的页面，直接 requests 等库来抓取原始页面，是无法获取到有效数据的， 这时需要分析网页后台 接口发送的 Ajax 请求，如果可以用 requests 来模拟 Ajax 请求，那么就可以成功抓取了.



1. 分析方法：首先我们打开微博：https://m.weibo.cn/u/1776448504，同样的熟练打开开发者工具Network，看一下类型为xhr的链接（Ajax请求类型为xhr），可以发现有getlndex 开头的请求，其 Type为xhr ，这就是 Ajax 请求。点开可以看见Request，Headers，URL和Response Headers 等信息 其中 Request Headers 中有 个信息为 X-Requested-With:XMLHttpRequest ，这就标记了此请求是 Ajax 请求。

2. 随后点击一下 Preview ，即可看到响应的内容，它是 JSON 格式的。这里的返回结果是个人信息，如昵称、简介、头像等，这也是用来渲染个人主页所使用的数据 ，JavaScript 接收到这些数据之后，再执行相应的渲染方法，整个页面就渲染出来了。也可以切换到 Response 选项卡，从中观察到真实的返回数据。

3. 我们来看一下现在不断往下滑动，开发者工具里面是不是出来了许多getIndex开头的链接说明是一样的都是通过Ajax请求才加载出来的数据，我们点开看详细信息（Headers）：可以看到里面的url为：https://m.weibo.cn/api/container/getIndex?display=0&retcode=6102&type=uid&value=1776448504&containerid=1076031776448504&since_id=4446229901328232

   分析多几个可以看出来固定的几个都是type，value，containerid，变的只有since id（老版微博中变得是page，表示当前是第几页，原理一样），这里就可以构造请求头了，请求这个链接，然后接受返回的数据。

4. 可以看下Preview选项卡里的响应内容，可以看到里面data的cards里面有10条内容，再点开其中一条的mblog可以看见里面就是微博内容的信息了，正文，点赞数，评论数，转发数等等各种内容。然后就可以开始写代码了。

   



```python
import requests
from urllib.parse import urlencode
from pyquery import PyQuery as pq
#from pymongo import MongoClient



base_url = 'https://m.weibo.cn/api/container/getIndex?'#基本url
headers = {
    'Host': 'm.weibo.cn',
    'Referer': 'https://m.weibo.cn/u/1776448504',
    'User-Agent': '写自己的',
    'X-Requested-With': 'XMLHttpRequest',
}  #构建参数头，在network中查看
#client = MongoClient()  ##连接数据库
#db = client['pydazuoye']  #指定数据库
#collection = db['py'] #声明对象
max_page = 3  #最大翻页


def get_page(page):  #构造每一页的请求
    params = {
        'type': 'uid',
        'value': '1776448504',
        'containerid': '1076031776448504',
        'page': page
    }
    url = base_url + urlencode(params) #完整的url，,将参数传递过去，根据network发现
    try:                                 #异常处理，若有问题则捕捉报错
        response = requests.get(url, headers=headers)
        if response.status_code == 200:  #响应码
            return response.json(), page     #返回内容json内容
    except requests.ConnectionError as e:
        print('Error', e.args)


def parse_page(json, page: int):  #解析每一页的内容
    if json:
        items = json.get('data').get('cards')
        #print(items)
        for index, item in enumerate(items): #遍历生成器里的每一个数据
            #print(index, item)
            if page == 1 and index == 1:
                continue
            else:
                item = item.get( 'mblog')  #利用network分析，提取mblog里面的内容
                #print(item)
                if item == None:
                    break
                else:
                    tgb = {}              #对照内容，构造想要的数据列表
                    tgb['id'] = item.get('id')
                    tgb['text'] = pq(item.get('text')).text()
                    tgb['attitudes'] = item.get('attitudes_count')
                    tgb['comments'] = item.get('comments_count')
                    tgb['reposts'] = item.get('reposts_count')
                    yield tgb
                



if __name__ == '__main__':
    for page in range(1, max_page + 1): #设置翻页
        json = get_page(page)   #下面是参数传递
        results = parse_page(*json)
        for result in results:       #输出
            print(result)
            
```

**提示：注意理解代码顺序和思路**
**小任务：上面我有提到老版的微博使用的参数是page，那么新版中的参数是什么，应该怎么处理翻页呢那？**

