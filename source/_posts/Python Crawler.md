---
title: 记录首次爬虫编写
---
# 记录首次爬虫编写


一直就想写写爬虫，刚好朋友问到帮忙爬数据，刚好练手。
需要从[目标网页](link:http://www.chntkd.org.cn/company/list.asp?provinceid=0&clubtype=0)抓取所有的地址和电话。



## 解析网页

使用 Chrome 打开网页，打开开发者工具查看网页源码。一看结构惊呆，一团麻，各种 table & body 互相嵌套。找到目标文字发现了一个规律，所有需要抓取的文字前面都有 `<br>` 标签，这就容易很多了。接着分析网址中需要的参数，分别是 `provinceid` 和 `page`，`clubtype` 直接传 0，`provinceid` 值固定从网页分析得到一组数字就可以，`page` 相对每个 `provinceid` 不同，分析后发现 
 ```javascript
function slk(){    
    var pageid = document.getElementById("pageid");    
    if(pageid.value > 4) pageid.value = 4
}
```
 里面有个函数来判断页码是否合法，直接通过正则 `.*pageid.value = (.*)}<*` 就可以获取正确的最大页码了，至此页面元素就分析完了。

## 脚本编写

 通过搜索引擎找到了一个可以跑的[爬虫脚本范例](link:https://www.cnblogs.com/xisheng/p/9130156.html)，查了些 Python 的基本语法和 的使用，改改后基本完成。
 
### 脚本代码

```python
import os,sys
import urllib2
from bs4 import BeautifulSoup
import re
import time

reload(sys)
sys.setdefaultencoding( "utf-8" )
fp = open("data.txt","w")

def catchWithPage(provinceid,page,clubtype=0):
    URL = "http://www.chntkd.org.cn/company/list.asp?page=%d&provinceid=%d&clubtype=%d" % (page,provinceid,clubtype)
    page = urllib2.urlopen(URL)
    soup = BeautifulSoup(page,"html.parser")
    page.close()
    for tag in soup.findAll('br'):
        fp.write(tag.next)
        fp.write("\n")


def catch(provinceid,clubtype=0):
    URL = "http://www.chntkd.org.cn/company/list.asp?provinceid=%d&clubtype=%d" % (provinceid,clubtype)
    page = urllib2.urlopen(URL)
    str = page.read()
    result = re.findall(".*pageid.value = (.*)}<*",str)
    print(result)
    num = int(result[0])
    print("province : %d  page count : %d"%(provinceid,num))
    page.close()
    for index in range(1, num+1):
        catchWithPage(provinceid,index)

for pi in (0, 1, 20, 40, 52, 64, 77, 92, 102, 116, 136, 150, 162, 180, 190, 202, 220, 238, 252, 267, 289, 304, 307, 308, 330, 340, 357, 365, 376, 391, 400, 406, 423, 455, 1035, 1091, 1124, 1236, 1237, 2183, 3296, 3568):
    catch(pi)
    time.sleep(1)

fp.close()

```
        
### 脚本缺点
没有做异常处理，当网页错误时脚本会中断，另外网页的访问速度挺慢脚本执行时间也较长。



