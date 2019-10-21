## 浏览器操作
包括启动，最大化，设置大小，前进后退，退出

```
//导入webdriver
In [1]: from selenium import webdriver

//启动浏览器（可以使用无头模式headless）
In [2]: bs = webdriver.Chrome('C:/Program Files/chromedriver2.exe')

//最大化窗口
In [3]: bs.maximize_window()

//设置窗口大小（像素）
In [4]: bs.set_window_size(400,500)

//打开网址
In [5]: bs.get("http://www.baidu.com")

In [6]: bs.get("http://www.163.com")

//后退到上一个网址
In [7]: bs.back()

//前进到上一个网址
In [8]: bs.forward()

//浏览器退出
In [9]: bs.quit()
```
## 元素定位

页面元素的属性 | 定位方法| 说明
---|---|---
id |find_element_by_id()|唯一id
name | find_element_by_name()|可能有多个
class name |find_element_by_class_name()|可能有多个
tag name | find_element_by_tag_name()|相同 tag name的元素极其容易出现
link text |find_element_by_link_text()|操作的元素是一个文字链接，那么我们可以通过link text 或 partial link text定位
partial link text  | find_element_by_partial_link_text()|
xpath |find_element_by_xpath()|绝对路径的方式<p>find_element_by_xpath("/html/body/div[2]/form/span/input")<p>firebug或者chrome的F12打开源码页面元素上右击弹出快捷菜单，选择 Copy XPath，
css selector |find_element_by_css_selector()|

#### css定位
CSS 可以比较灵活选择控件的任意属性，一般情况下定位速度要比 XPath 快。

#### CSS选择器的常见语法
定位方法 | 说明
---|---
* |通用元素选择器，匹配任何元素
E |标签选择器，匹配所有使用 E 标签的元素
.info| class 选择器，匹配所有 class属性中包含 info 的元素
#footer |id 选择器，匹配所有 id 属性等于 footer 的元素
E,F |多元素选择器，同时匹配所有 E 元素或 F元素，E 和 F之间用逗号分隔
E F |后代元素选择器，匹配所有属于 E 元素后代的 F 元素，E 和 F 之间用空格分隔
E > F| 子元素选择器，匹配所有 E 元素的子元素 F
E + F| 毗邻元素选择器，匹配紧随 E 元素之后的同级元素 F （只匹配第一个）
E ~ F |同级元素选择器，匹配所有在 E 元素之后的同级 F 元素
E[att='val']| 属性 att 的值为 val 的 E 元素 （区分大小写）
E[att^='val'] |属性 att 的值以 val 开头的 E 元素 （区分大小写）
E[att$='val'] |属性 att 的值以 val 结尾的 E 元素 （区分大小写）
E[att*='val'] |属性 att 的值包含 val 的 E 元素 （区分大小写）
E[att1='v1'][att2*='v2'] |属性 att1 的值为 v1，att2 的值包含 v2 （区分大小写）
E:contains('xxxx') |内容中包含 xxxx 的 E 元素
E:not(s) |匹配不符合当前选择器的任何元素


```
<div class="subdiv">
<ul id="recordlist">
<p>Heading</p>
<li>Cat</li>
<li>Dog</li>
<li>Car</li>
<li>Goat</li>
</ul>
</div>
```


#### CSS选择器的样例
定位器 | 结果
---|---
css=div.subdiv p<p>css=div.subdiv > ul > p|\<p>Heading\</p>
css=form + div |\<div class="subdiv">
css=p + li<p>css=p ~ li|二者定位到的都是 \<li>Cat\</li>但是 storeCssCount 的时候，前者得到 1，后者得到 4
css=form > input[name=username]| <input name="username">
css=input[name$=id][value^=SYS]| <input value="SYS123456" name="vid" type="hidden">
css=input:not([name$=id][value^=SYS])| <input name="username" type="text"></input>
css=li:contains('Goa')|\<li>Goat\</li>
css=li:not(contains('Goa'))|\<li>Cat\</li>

#### css中的结构性定位
结构性定位就是根据元素的父子、同级中位置来定位
定位器 | 结果|样例|结果样例
---|---|---|---
E:nth(n)<p>E:eq(n)|在其父元素中的E子元素集合中排在第n+1个的E元素 (第一个n=0)|ul > li:nth(0)|\<li>Cat\</li>
E:first|在其父元素中的E子元素集合中排在第1个的E元素|ul > *:nth(0)|\<p>Heading\</p>
E:last|在其父元素中的E子元素集合中排在最后1个的E元素|css=ul > *:last  <p>css=ul > li:last|\<li>Goat\</li>
E:even|在其父元素中的E子元素集合中排在偶数位的E元素 (0,2,4…)|ul > li:even<p>ul > :even | Cat, Car<p>\<p>Heading\</p>
E:odd|在其父元素中的E子元素集合中排在奇数的E元素 (1,3,5…)|ul > li:odd Dog,  <p>ul > p:odd|Goat  <p> [error] not found
E:lt(n)|在其父元素中的E子元素集合中排在n位之前的E元素 (n=2,则匹配0,1)|css=ul > li:lt(2) |\<li>Cat\</li>
E:gt(n)|在其父元素中的E子元素集合中排在n位之后的E元素 (n=2，在匹配3,4)|css=ul > li:gt(2)| \<li>Goat\</li>
E:only-child|父元素的唯一一个子元素且标签为E|css=ul > li:only-child<p>css=ul > :only-child<p>css=ul > *:only-child|[error] not found (ul 没有 only-child)
E:empty|不包含任何子元素的E元素，注意，文本节点也被看作子元素|css=div.subdiv > :only-child| <ul id="recordlist"> … … …</ul>

#### PATH和 CSS的类似定位功能比较
定位方式 | xpath| css
---|---|---
标签 |//div |div
By id |//div[@id='eleid'] |div#eleid
By class |//div[@class='eleclass'] <p>//div[contains(@class,'eleclass')]|div.eleclass
By 属性| //div[@title='Move mouse here'] |div[title=Move mouse here] <p>div[title^=Move]<p>div[title$=here]<p>div[title*=mouse]
定位子元素 |//div[@id='eleid']/*<p>//div/h1|div#eleid >*<p>div#eleid >h1
定位后代元素 |//div[@id='eleid']//h1 |div h1
By index |//li[6] |li:nth(5)
By content| //a[contains(.,'Issue 1164')] |a:contains(Issue 1164)

- CSS定位语法比 XPath 更为简洁，定位方式更多灵活多样
- 不过对 CSS 理解起来要比 XPath较难
- 不管是从性能还是定位更复杂的元素上，CSS 优于XPath，
- css和xpath相比，==更推荐使用 CSS定位页面元素==
- XPath 和 CSS 可以定位到复杂且比较难定位的元素，但相比较用 id和 name来说增加了维护成本和学习成本，==id/name的定位方式更直观和可维护==


## 操作元素
#### 常用方法
```
In [1]: from selenium import webdriver

//启动浏览器（可以使用无头模式headless）
In [2]: bs = webdriver.Chrome('C:/Program Files/chromedriver2.exe')

//最大化窗口
In [3]: bs.maximize_window()

//设置窗口大小（像素）
In [4]: bs.set_window_size(400,500)

//打开网址
In [5]: bs.get("http://www.baidu.com")


In [38]: input_text =  bs.find_element_by_css_selector("#kw")

In [39]: input_text.clear()

In [40]: input_text.send_keys("test")

In [41]: input_text.clear()

In [42]: btn = bs.find_element_by_css_selector("#su")

In [43]: btn.click()

In [44]: btn.submit()
```

#### 常用属性

```
In [46]: btn = bs.find_element_by_css_selector("#su")

In [47]: btn.size
Out[47]: {'height': 36, 'width': 100}

In [48]: btn.text
Out[48]: ''

In [49]: btn.id
Out[49]: '0.8874896518259712-1'

In [50]: btn.get_attribute("type")
Out[50]: 'submit'

In [51]: btn.get_attribute("href")

In [52]: btn.is_displayed()
Out[52]: True

In [53]: btn.is_enabled()
Out[53]: True

In [54]: btn.is_selected()
Out[54]: False

```

## 鼠标操作

```
from selenium.webdriver.common.action_chains import ActionChains

//定位到要右击的元素
right =driver.find_element_by_xpath("xx")

//对定位到的元素执行鼠标右键操作
ActionChains(driver).context_click(right).perform()

//对定位到的元素执行鼠标双击操作
ActionChains(driver).double_click(double).perform()

//对元素拖拽
#定位元素的原位置
element = driver.find_element_by_name("xxx")
#定位元素要移动到的目标位置
target = driver.find_element_by_name("xxx")
#执行元素的移动操作
ActionChains(driver).drag_and_drop(element, target).perform()

//对定位到的元素执行鼠标移动到上面的操作
ActionChains(driver).move_to_element(above).perform()

#对定位到的元素执行鼠标左键按下不动的操作
ActionChains(driver).click_and_hold(left).perform()


```


## 键盘事件

```
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
bs = webdriver.Chrome('C:/Program Files/chromedriver2.exe')
btn = bs.find_element_by_css_selector("#su")

//往文本框输入文字
btn.send_keys("selenium")

//删除文字，即输入backspace
btn..send_keys(Keys.BACK_SPACE)

//输入空格
btn.send_keys(Keys.SPACE)

//ctrl + a， ctrl + x组合键
btn.send_keys(Keys.CONTROL,'a')
btn.send_keys(Keys.CONTROL,'x')

//enter
btn.send_keys(Keys.ENTER)
```


## 打印信息
title，current_url等信息

```
print(bs.title,bs.current_url)
```

## 设置等待时间
- time.sleep():单位是秒
- implicitly_wait()
```
from selenium.webdriver.support.ui import WebDriverWait
WebDriverWait(driver, timeout, poll_frequency=0.5, ignored_exceptions=None)
//driver - WebDriver 的驱动程序(Ie, Firefox, Chrome 或远程)
//timeout - 最长超时时间，默认以秒为单位
//poll_frequency - 休眠时间的间隔（步长）时间，默认为 0.5秒
//ignored_exceptions - 超时后的异常信息，默认情况下抛 NoSuchElementException 异常

element = WebDriverWait(driver, 10).until(lambda x: x.find_element_by_id(“someId”))
is_disappeared = WebDriverWait(driver, 30, 1, (ElementNotVisibleException)).
until_not(lambda x: x.find_element_by_id(“someId”).is_displayed())


```
until(method, message=’ ’)
调用该方法提供的驱动程序作为一个参数，直到返回值不为 False。
until_not(method, message=’ ’)
调用该方法提供的驱动程序作为一个参数，直到返回值为 False

- implicitly_wait()

```
driver.implicitly_wait(30)
driver.find_element_by_id("su").click()
```

## 定位一组对象
find_elements_by_XXXX

## 层级定位

## 定位 frame 中的对象
- driver.switch_to_frame("f1")
- switch_to_frame 的参数问题。官方说 name 是可以的，但是经过实验发现 id也可以。所以只要 frame中 id 和 name，那么处理起来是比较容易的。如果 frame没有这两个属性的话，你可以直接手动添加。

## 对话框处理
页面上弹出的对话框是自动化测试经常会遇到的一个问题：
- 很多情况下对话框是一个 iframe，如上一节中介绍的例子，处理起来稍微有点麻烦
- 但现在很多前端框架的对话框是 div 形式的，这就让我们的处理变得十分简单，直接使用二次定位


## 浏览器多窗口处理

```
#获得当前窗口
nowhandle=driver.current_window_handle


#获得所有窗口
allhandles=driver.window_handles

#回到原先的窗口
driver.switch_to_window(nowhandle)

## 关闭标签页
driver.close()
```

## alert/confirm/prompt 处理
具体思路是使用 switch_to.alert()方法定位到 alert/confirm/prompt。然后使用 text/accept/dismiss/send_keys 按需进行操作

header 1 | header 2
---|---
text| 返回 alert/confirm/prompt 中的文字信息。
accept |点击确认按钮。
dismiss |点击取消按钮，如果有的话。
send_keys |输入值，这个 alert\confirm 没有对话框就不能用了，不然会报错

## 下拉框处理
先定位到下拉框，然后点击下拉框下的选项
```
from selenium import webdriver
import os,time
driver= webdriver.Firefox()
file_path = 'file:///' + os.path.abspath('drop_down.html')
driver.get(file_path)
time.sleep(2)

#先定位到下拉框
m=driver.find_element_by_id("ShippingMethod")
#再点击下拉框下的选项
m.find_element_by_xpath("//option[@value='10.69']").click()
```
## 分页处理


## 上传文件
定位上传按钮，添加本地文件
driver.find_element_by_name("file").send_keys('D:\\selenium_use_case\upload_file.txt')

## 下载文件（设定下载路径）

```

```


## 设定浏览器参数
```
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
#chrome_options.add_argument('--headless')
prefs = {"plugins.plugins_disabled": ['Chrome PDF Viewer'],  "plugins.plugins_disabled": ['Adobe Flash Player'],  'profile.default_content_setting_values': {'images': 2,'javascript':1}}
chrome_options.add_experimental_option('prefs', prefs)
bs = webdriver.Chrome('C:/Program Files/chromedriver2.exe',chrome_options=chrome_options)
bs.get("https://pan.baidu.com")
```

## 调用js
webdriver 提供了 execute_script() 接口用来调用 js代码

```
#将页面滚动条拖到底部
js="var q=document.documentElement.scrollTop=10000"
driver.execute_script(js)


#将滚动条移动到页面的顶部
js_="var q=document.documentElement.scrollTop=0"
driver.execute_script(js_)
```

## cookie 处理
get_cookies() 获得所有 cookie 信息<p>
get_cookie(name) 返回特定 name 有 cookie信息<p>
add_cookie(cookie_dict) 添加 cookie，必须有 name 和 value 值<p>
delete_cookie(name) 删除特定(部分)的 cookie信息<p>
delete_all_cookies() 删除所有 cookie 信息

```
from selenium import webdriver
bs = webdriver.Chrome('C:/Program Files/chromedriver2.exe')
bs.get("http://www.baidu.com")

 print(cookie)
[{'domain': '.baidu.com', 'httpOnly': False, 'name': 'H_PS_PSSID', 'path': '/', 'secure': False, 'value': '26525_1449_21114_29522_29518_29098_29567_28831_29220_29071_2964
1'}, {'domain': '.baidu.com', 'expiry': 3713169529.666848, 'httpOnly': False, 'name': 'BIDUPSID', 'path': '/', 'secure': False, 'value': 'D9C68D8CEDDD72FA184A3CCCE4609DD2
'}, {'domain': '.baidu.com', 'httpOnly': False, 'name': 'delPer', 'path': '/', 'secure': False, 'value': '0'}, {'domain': '.baidu.com', 'expiry': 3713169529.6669, 'httpOn
ly': False, 'name': 'PSTM', 'path': '/', 'secure': False, 'value': '1565685882'}, {'domain': '.baidu.com', 'expiry': 1565772283.856541, 'httpOnly': False, 'name': 'BDORZ'
, 'path': '/', 'secure': False, 'value': 'B490B5EBF6F3CD402E515D22BCDA1598'}, {'domain': 'www.baidu.com', 'expiry': 1566549883, 'httpOnly': False, 'name': 'BD_UPN', 'path
': '/', 'secure': False, 'value': '12314553'}, {'domain': 'www.baidu.com', 'httpOnly': False, 'name': 'BD_HOME', 'path': '/', 'secure': False, 'value': '0'}, {'domain': '
.baidu.com', 'expiry': 3713169529.666683, 'httpOnly': False, 'name': 'BAIDUID', 'path': '/', 'secure': False, 'value': 'D9C68D8CEDDD72FA184A3CCCE4609DD2:FG=1'}]
 
cookie = bs.get_cookie('H_PS_PSSID')
 
bs.add_cookie(cookie)
 
bs.delete_cookie("CookieName")
 
bs.delete_cookie('H_PS_PSSID')

bs.delete_all_cookies()
```

## 获取对象的属性
get_attribute('data-node')

```
bs.find_element_by_css_selector("#su").get_attribute("href")
```

## 验证码问题
1. 去掉验证码：对于开发人员来说，只是把验证码的相关代码注释掉即可
2. 设置万能码：程序中留一个“后门”---设置一个“万能验证码”
3. 验证码识别技术：Python-tesseract 来识别图片验证码
4. 在线验证码识别：使用在线的网络API接口，将验证码图片发送过去，然后返回得到结果
5. 记录 cookie


## expected_conditions
expected_condtions提供了16种判断页面元素的方法：
方法|解释
---|---
title_is|判断当前页面的title是否完全等于预期字符串，返回布尔值
title_contains|判断当前页面的title是否包含预期字符串，返回布尔值
presence_of_element_located|判断某个元素是否被加到dom树下,不代表该元素一定可见
visibility_of_element_located|判断某个元素是否可见,可见代表元素非隐藏,并且元素的宽和高都不为0
visibility_of|跟上面的方法是一样的,只是上面需要传入locator,这个方法直接传定位到的element就好
presence_of_all_elements_located|判断是否至少一个元素存在于dom树中,举个例子,如果页面上有n个元素的class都是'coumn-md-3',name只要有一个元素存在,这个方法就返回True
text_to_be_present_in_element|判断某个元素中的text文本是否包含预期字符串
text_to_be_present_in_element_value|判断某个元素中的value属性值是否包含了预期字符串
frame_to_be_availabe_and_switch_to_it|判断该frame是否可以switch进去,如果可以,则返回True并且switch进去,否则返回False
invisibility_of_element_located|判断某个元素是否不存在于dom树或不可见
element_to_be_clickable|判断某个元素是见并且是enable(有效)的,这样的话才叫clickable
staleness_of|等某个元素从dom树下移除,返回True或False
element_to_be_selected|判断某个元素是否被选中,一般用于select下拉表
element_selection_state_to_be|判断某个元素的选中状态是否符合预期
element_located_selection_state_to_be|跟上面的方法一样,只是上面的方法传入定位到的element,这个方法传入locator
alert_is_present|判断页面上是会否存在alert


```
text_should_present = EC.text_to_be_present_in_element((By.NAME, ‘tj_trhao123‘), ‘hao123‘)
self.assertTrue(text_should_present(dr))

或者
expected_conditions.title_contains("百度")(bs)  //这种写法看起来有点奇怪expected_conditions.title_contains("百度")返回的是对象<selenium.webdriver.support.expected_conditions.title_is object at 0x00000014B3C4DF28>
```
