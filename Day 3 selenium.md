## Selenium:

节前准备：安装好selenium库以及ChromeDriver（首先确保安装好chrome），因为只有安装 ChromeDriver ，才能驱动 Chrome 浏览器完成相应的操作。**请记住 Chrome 版本号，因为选择 ChromeDriver 版本时需要用到**安装如下：

下载地址：http://npm.taobao.org/mirrors/chromedriver

版本对照：https://blog.csdn.net/yoyocat915/article/details/80580066

下载好后，在Window下，建议直接将 chromedriver.exe 文件拖到 Python的Scripts 目录下

在程序中，执行如下代码，若有浏览器弹框出来即说明成功辽：

```python
from selenium import webdriver
browser = webdriver.Chrome() 
```

如果闪退 则可能 ChromeDriver 版本和 Chrome 版本不兼容

**节前提示，手敲代码，别光看，光看没用的**
*1.声明浏览器对象*

``` python
from selenium import webdriver

browser = webdriver.Chrome()           //谷歌浏览器
browser = webdriver.Firefox()          //火狐
browser = webdriver.Edge()             //edge
browser = webdriver.PhantomJS()        //无界面
browser = webdriver.Safari()
```

Selenium 支持非常多的浏览器，如 Chrome、Firefox、Edge 等，还有手机端的浏览器 Android、BlackBerry 等，另外无界面浏览器 PhantomJS 也同样支持。

*2.访问页面*

我们可以用 get() 方法来请求一个网页，参数传入链接 URL 即可

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com')
print(browser.page_source)
browser.close()
```

*3.查找*  


1.所有获取单个节点的方法：

```python
find_element_by_id
find_element_by_name
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector



具体格式参下：
input_first = browser.find_element_by_id('lily')
input_second = browser.find_element_by_css_selector('#lily')
input_third = browser.find_element_by_xpath('//*[@id="lily"]')   
```

三种不同的实现方法（其实都是定位）根据id（怎么查找id这种应该你们都会了吧，就不多说了），css选择器，xpath，这样就可以定位到哪一个地方进行操作，比如：自动输入账号密码再点击登录。

 Selenium 还提供了通用的 find_element() 方法,比如 find_element_by_id(id) 就等价于 find_element(By.ID, id)

 ```python
 from selenium.webdriver.common.by import By
 ```

 2.多个节点

 如果要查找所有满足条件的节点，那就需要用 find_elements() 
 将上述单个节点的element加个‘s’就行

   我们用 find_element() 方法，只能获取匹配的第一个节点，结果是 WebElement 类型，如果用 find_elements() 方法，则结果是列表类型，列表的每个节点是 WebElement 类型。  
   也可用上述 by的方法

*4.模拟浏览器操作*

输入文字，用 send_keys() 方法  
清空文字，用 clear() 方法  
按钮点击，用 click() 方法  

```python
input = browser.find_element_by_id('lily')
input.send_keys('sb')
input.clear()

button = browser.find_element_by_class_name('btn-search')
button.click()
```

官方文档的交互动作介绍:  
[http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement)

5.动作链

在上面的实例中，一些交互动作都是针对某个节点执行的 比如，对于输入框，我们就调用它的 输入文字和清空文字方法 ；对于按钮，就调用它的点击方法 其实，还有另外一些操作，它们没有特定的执行对象，比如鼠标拖曳 键盘按键等，这些动作用另一种方式来执行，那就是动作链。

在实现某个节点的拖曳操作，将某个节点从一处拖曳到另外一处，可以这样实现：

```python
from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector('#draggable')  #原点
target = browser.find_element_by_css_selector('#droppable')   #目标点
actions = ActionChains(browser)
actions.drag_and_drop(source, target)
actions.perform()
```

首先，打开网页中的一个拖曳实例 ，然后依次选中要拖曳的节点和拖曳到的目标节点，接着声明 ActionChains 对象并将其赋值为 actions 变量，然后通过调用 actions 变量的 drag_and_drop（）方法， 再调 perform()方法执行动作，此时就完成了拖曳操作。

**小任务1：结合上面讲的定位以及动作链，知道滑动式的验证码怎么解决了吗，写一段小代码试试吧（或者把思路写下来，要求有伪代码，以及试试能不能成功，为什么不能，出现了什么问题？）**



6.执行JavaScript

对于某些操作， Selenium API 并没有提供，比如，下拉进度条

```python
from selenium import webdriver
browser= webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert(”To Bottom”)')
```

这里就利用 execute script ()方法将进度条下拉到最底部，然后弹出alert提示框

所以说有了这个方法，基本上 API没有提供的所有功能都可以用执行 JavaScript 的方式来实现了



7.获取节点信息

Selenium 已经提供了选择节点的方法，返回的是 Web Element 类型，那么它也有相应的方法和属性来直接提取节点信息，如属性、文本等。

- 获取属性,利用get_attribute()方法

  ```python
  logo= browser.find_element_by_id('zh-top-link-logo') //自己找一个节点试一下
  print(logo)
  print(logo.get_attribute('class'))
  ```

- 获取文本，利用text属性

  ```python
  logo= browser.find_element_by_id('zh-top-link-logo') //自己找一个节点试一下
  print(logo)
  print(logo.text)
  ```

- 其他一系列有兴趣自行百度



8. Cookies

   使用 Selenium ，还可以方便地对 Cookies 进行操作，例如获取 添加 删除 Cookies

   ```python
   from selenium import webdriver
   from selenium.webdriver import ActionChains
   
   browser = webdriver.Chrome()
   url = 'https://www.zhihu.com/explore'
   browser.get(url)
   print(brower.get_cookies())  
   brower.add_cookies({'name':'自行输入','domain':'www.zhihu.com'})   //增加一个字典
   print(brower.get_cookies())                                     
   brower.delete_all_cookies()                                       //删除所有cookies
   print(brower.get_cookies())
   ```
**小任务2：认真来说，selenium的玩法很多，给你们例举出来并不是只有这些，很多企业级的数据工作会经常用到selenium，想一想是为什么？**
   
