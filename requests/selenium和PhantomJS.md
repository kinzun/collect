## **简介**

selenium最初是一个自动化测试工具,而爬虫中使用它主要是为了解决requests无法直接执行JavaScript代码的问题 selenium本质是通过驱动浏览器，完全模拟浏览器的操作，比如跳转、输入、点击、下拉等，来拿到网页渲染之后的结果，可支持多种浏览器

## **环境安装**

- 下载安装selenium：pip install selenium
- 下载浏览器驱动程序：
  - http://chromedriver.storage.googleapis.com/index.html
- 查看驱动和浏览器版本的映射关系：
  - http://blog.csdn.net/huilan_same/article/details/51896672

## **简单使用/效果展示**

```python
from selenium import webdriver
from time import sleep

# 后面是你的浏览器驱动位置，记得前面加r'','r'是防止字符转义的
driver = webdriver.Chrome(r'驱动程序路径')
# 用get打开百度页面
driver.get("http://www.baidu.com")
# 查找页面的“设置”选项，并进行点击
driver.find_elements_by_link_text('设置')[0].click()
sleep(2)
# # 打开设置后找到“搜索设置”选项，设置为每页显示50条
driver.find_elements_by_link_text('搜索设置')[0].click()
sleep(2)

# 选中每页显示50条
m = driver.find_element_by_id('nr')
sleep(2)
m.find_element_by_xpath('//*[@id="nr"]/option[3]').click()
m.find_element_by_xpath('.//option[3]').click()
sleep(2)

# 点击保存设置
driver.find_elements_by_class_name("prefpanelgo")[0].click()
sleep(2)

# 处理弹出的警告页面   确定accept() 和 取消dismiss()
driver.switch_to_alert().accept()
sleep(2)
# 找到百度的输入框，并输入 美女
driver.find_element_by_id('kw').send_keys('美女')
sleep(2)
# 点击搜索按钮
driver.find_element_by_id('su').click()
sleep(2)
# 在打开的页面中找到“Selenium - 开源中国社区”，并打开这个页面
driver.find_elements_by_link_text('美女_百度图片')[0].click()
sleep(3)

# 关闭浏览器
driver.quit()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **浏览器创建**

Selenium支持非常多的浏览器，如Chrome、Firefox、Edge等，还有Android、BlackBerry等手机端的浏览器。另外，也支持无界面浏览器PhantomJS。

```python
from selenium import webdriver
  
browser = webdriver.Chrome()
browser = webdriver.Firefox()
browser = webdriver.Edge()
browser = webdriver.PhantomJS()
browser = webdriver.Safari()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **元素定位**

webdriver 提供了一系列的元素定位方法，常用的有以下几种：

```python
find_element_by_id()
find_element_by_name()
find_element_by_class_name()
find_element_by_tag_name()
find_element_by_link_text()
find_element_by_partial_link_text()
find_element_by_xpath()
find_element_by_css_selector()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意

1、find_element_by_xxx找的是第一个符合条件的标签，find_elements_by_xxx找的是所有符合条件的标签。

2、根据ID、CSS选择器和XPath获取，它们返回的结果完全一致。

3、另外，Selenium还提供了通用方法`find_element()`，它需要传入两个参数：查找方式`By`和值。实际上，它就是`find_element_by_id()`这种方法的通用函数版本，比如`find_element_by_id(id)`就等价于`find_element(By.ID, id)`，二者得到的结果完全一致。

## **节点交互**

Selenium可以驱动浏览器来执行一些操作，也就是说可以让浏览器模拟执行一些动作。比较常见的用法有：输入文字时用`send_keys()`方法，清空文字时用`clear()`方法，点击按钮时用`click()`方法。示例如下：

```python
from selenium import webdriver
import time
 
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input = browser.find_element_by_id('q')
input.send_keys('MAC')
time.sleep(1)
input.clear()
input.send_keys('IPhone')
button = browser.find_element_by_class_name('btn-search')
button.click()
browser.quit()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **动作链**

在上面的实例中，一些交互动作都是针对某个节点执行的。比如，对于输入框，我们就调用它的输入文字和清空文字方法；对于按钮，就调用它的点击方法。其实，还有另外一些操作，它们没有特定的执行对象，比如鼠标拖曳、键盘按键等，这些动作用另一种方式来执行，那就是动作链。

比如，现在实现一个节点的拖曳操作，将某个节点从一处拖曳到另外一处，可以这样实现：

```python
from selenium import webdriver
from selenium.webdriver import ActionChains
import time
browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector('#draggable')
target = browser.find_element_by_css_selector('#droppable')
actions = ActionChains(browser)
# actions.drag_and_drop(source, target)
# actions.perform() #执行动作链
actions.click_and_hold(source)
time.sleep(3)
for i in range(5):
    actions.move_by_offset(xoffset=17,yoffset=0).perform()
    time.sleep(0.5)

actions.release()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **执行JavaScript**

对于某些操作，Selenium API并没有提供。比如，下拉进度条，它可以直接模拟运行JavaScript，此时使用`execute_script()`方法即可实现，代码如下：

```python
from selenium import webdriver
 
browser = webdriver.Chrome()
browser.get('https://www.jd.com/')
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert("123")')
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **获取页面源码数据**

通过`page_source`属性可以获取网页的源代码，接着就可以使用解析库（如正则表达式、Beautiful Soup、pyquery等）来提取信息了。

## **前进和后退** 

```python
#模拟浏览器的前进后退
import time
from selenium import webdriver
 
browser=webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.get('https://www.taobao.com')
browser.get('http://www.sina.com.cn/')
 
browser.back()
time.sleep(10)
browser.forward()
browser.close()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **Cookie处理**

使用Selenium，还可以方便地对Cookies进行操作，例如获取、添加、删除Cookies等。示例如下：

```python
from selenium import webdriver
 
browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com', 'value': 'germey'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **异常处理**

```python
from selenium import webdriver
from selenium.common.exceptions import TimeoutException,NoSuchElementException,NoSuchFrameException

try:
    browser=webdriver.Chrome()
    browser.get('http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable')
    browser.switch_to.frame('iframssseResult')

except TimeoutException as e:
    print(e)
except NoSuchFrameException as e:
    print(e)
finally:
    browser.close()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **phantomJS**

PhantomJS是一款无界面的浏览器，其自动化操作流程和上述操作谷歌浏览器是一致的。由于是无界面的，为了能够展示自动化操作流程，PhantomJS为用户提供了一个截屏的功能，使用save_screenshot函数实现。

```python
from selenium import webdriver
import time

# phantomjs路径
path = r'PhantomJS驱动路径'
browser = webdriver.PhantomJS(path)

# 打开百度
url = 'http://www.baidu.com/'
browser.get(url)

time.sleep(3)

browser.save_screenshot(r'phantomjs\baidu.png')

# 查找input输入框
my_input = browser.find_element_by_id('kw')
# 往框里面写文字
my_input.send_keys('美女')
time.sleep(3)
#截屏
browser.save_screenshot(r'phantomjs\meinv.png')

# 查找搜索按钮
button = browser.find_elements_by_class_name('s_btn')[0]
button.click()

time.sleep(3)

browser.save_screenshot(r'phantomjs\show.png')

time.sleep(3)

browser.quit()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **谷歌无头浏览器**

由于PhantomJs最近已经停止了更新和维护，所以推荐大家可以使用谷歌的无头浏览器，是一款无界面的谷歌浏览器。

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
 
# 创建一个参数对象，用来控制chrome以无界面模式打开
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
# 驱动路径
path = r'C:\Users\ZBLi\Desktop\1801\day05\ziliao\chromedriver.exe'
 
# 创建浏览器对象
browser = webdriver.Chrome(executable_path=path, chrome_options=chrome_options)
 
# 上网
url = 'http://www.baidu.com/'
browser.get(url)
time.sleep(3)
 
browser.save_screenshot('baidu.png')
 
browser.quit()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 

## **登录qq空间，爬取数据**

```python
import requests
from selenium import webdriver
from lxml import etree
import time

driver = webdriver.Chrome(executable_path='/Users/bobo/Desktop/chromedriver')
driver.get('https://qzone.qq.com/')
#在web 应用中经常会遇到frame 嵌套页面的应用，使用WebDriver 每次只能在一个页面上识别元素，对于frame 嵌套内的页面上的元素，直接定位是定位是定位不到的。这个时候就需要通过switch_to_frame()方法将当前定位的主体切换了frame 里。
driver.switch_to.frame('login_frame')
driver.find_element_by_id('switcher_plogin').click()

#driver.find_element_by_id('u').clear()
driver.find_element_by_id('u').send_keys('328410948')  #这里填写你的QQ号
#driver.find_element_by_id('p').clear()
driver.find_element_by_id('p').send_keys('xxxxxx')  #这里填写你的QQ密码
    
driver.find_element_by_id('login_button').click()
time.sleep(2)
driver.execute_script('window.scrollTo(0,document.body.scrollHeight)')
time.sleep(2)
driver.execute_script('window.scrollTo(0,document.body.scrollHeight)')
time.sleep(2)
driver.execute_script('window.scrollTo(0,document.body.scrollHeight)')
time.sleep(2)
page_text = driver.page_source

tree = etree.HTML(page_text)
#执行解析操作
li_list = tree.xpath('//ul[@id="feed_friend_list"]/li')
for li in li_list:
    text_list = li.xpath('.//div[@class="f-info"]//text()|.//div[@class="f-info qz_info_cut"]//text()')
    text = ''.join(text_list)
    print(text+'\n\n\n')
    
driver.close()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **尽可能多的爬取豆瓣网中的电影信息**

```python
from selenium import webdriver
from time import sleep
import time

if __name__ == '__main__':
    url = 'https://movie.douban.com/typerank?type_name=%E6%81%90%E6%80%96&type=20&interval_id=100:90&action='
    # 发起请求前，可以让url表示的页面动态加载出更多的数据
    path = r'C:\Users\Administrator\Desktop\爬虫授课\day05\ziliao\phantomjs-2.1.1-windows\bin\phantomjs.exe'
    # 创建无界面的浏览器对象
    bro = webdriver.PhantomJS(path)
    # 发起url请求
    bro.get(url)
    time.sleep(3)
    # 截图
    bro.save_screenshot('1.png')

    # 执行js代码（让滚动条向下偏移n个像素（作用：动态加载了更多的电影信息））
    js = 'window.scrollTo(0,document.body.scrollHeight)'
    bro.execute_script(js)  # 该函数可以执行一组字符串形式的js代码
    time.sleep(2)

    bro.execute_script(js)  # 该函数可以执行一组字符串形式的js代码
    time.sleep(2)
    bro.save_screenshot('2.png') 
    time.sleep(2) 
    # 使用爬虫程序爬去当前url中的内容 
    html_source = bro.page_source # 该属性可以获取当前浏览器的当前页的源码（html） 
    with open('./source.html', 'w', encoding='utf-8') as fp: 
        fp.write(html_source) 
    bro.quit()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## **selenium规避被检测识别**

现在不少大网站有对selenium采取了监测机制。比如正常情况下我们用浏览器访问淘宝等网站的 window.navigator.webdriver的值为 
undefined。而使用selenium访问则该值为true。那么如何解决这个问题呢？

只需要设置Chromedriver的启动参数即可解决问题。在启动Chromedriver之前，为Chrome开启实验性功能参数`excludeSwitches`，它的值为`['enable-automation']`，完整代码如下：

```python
from selenium.webdriver import Chrome
from selenium.webdriver import ChromeOptions

option = ChromeOptions()
option.add_experimental_option('excludeSwitches', ['enable-automation'])
driver = Chrome(options=option)
```