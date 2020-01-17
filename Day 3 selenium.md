

*1.声明浏览器对象*
``` 
from selenium import webdriver

browser = webdriver.Chrome()
browser = webdriver.Firefox()
browser = webdriver.Edge()
browser = webdriver.PhantomJS()
browser = webdriver.Safari()
```
Selenium 支持非常多的浏览器，如 Chrome、Firefox、Edge 等，还有手机端的浏览器 Android、BlackBerry 等，另外无界面浏览器 PhantomJS 也同样支持。

*2.访问页面*

我们可以用 get() 方法来请求一个网页，参数传入链接 URL 即可
```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com')
print(browser.page_source)
browser.close()
```
*3.查找*  


1.所有获取单个节点的方法：
```
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
 Selenium 还提供了通用的 find_element() 方法,比如 find_element_by_id(id) 就等价于 find_element(By.ID, id)
 ```
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

```
input = browser.find_element_by_id('lily')
input.send_keys('sb')
input.clear()

button = browser.find_element_by_class_name('btn-search')
button.click()
```

官方文档的交互动作介绍:  
[http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement)
