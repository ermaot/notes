selenium切换到iframe

## 定位iframe
#### 1. 有id，并且唯一，直接写id
driver.switch_to_frame("x-URS-iframe")
driver.switch_to.frame("x-URS-iframe")

#### 2.有name，并且唯一，直接写name
driver.switch_to_frame("xxxx")
driver.switch_to.frame("xxxx")

#### 3.无id，无name,先定位iframe元素
iframe = driver.find_elements_by_tag_name("iframe")[0]
driver.switch_to_frame(iframe)
driver.switch_to.frame(iframe)

####　4.用indexWebElement来定位：

index从0开始，传入整型参数即判定为用index定位，传入str参数则判定为用id/name定位

#### 5.用indexWebElement来定位
WebElement对象，即用find_element系列方法所取得的对象，我们可以用tag_name、xpath等来定位frame对象
举个栗子：

<iframe src="myframetest.html" />
1
用xpath定位，传入WebElement对象：

driver.switch_to.frame(driver.find_element_by_xpath("//iframe[contains(@src,'myframe')]"))

## 从frame中切回主文档
(switch_to.default_content())

## 嵌套frame的操作
switch_to.parent_frame()
