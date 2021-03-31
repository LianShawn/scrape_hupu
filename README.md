# scrape_hupu

#因为第一层级的没有API，所以使用的是selenium来进行抓取
#先import需要的package

```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.keys import Keys
import matplotlib.pylab as plt
import sys
import json
import chardet
import time
from bs4 import BeautifulSoup
import re
import pandas as pd
import csv
import datetime
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager

```
#要找到电脑中的chromedriver在哪，如果没有就安装
```
driver = webdriver.Chrome(ChromeDriverManager().install())
```
```
titles = []
links = []
##获得链接
for link in driver.find_elements_by_xpath("//*[@href]"):
    if len(link.get_attribute('href')) == 34:
        print (link.get_attribute('href'))
        links.append(link.get_attribute('href'))
        
##获得内容
info = driver.find_elements_by_class_name('p_title')
for value in info:
    if len(value.text) > 2:
        print(value.text)
        titles.append(value.text)       
```

```
#完成对第一层数据的抓取
data = {
    "title":titles,
    'links':links
}
```
#第二层数据是具体的评论，hupu有提供API
#import 第二部分需要的package
```
import requests 
import pandas as pd
import re
from fake_useragent import UserAgent
import json
import time
```

#抓取的模型
```
def get_content(url):
    content = requests.get(url)
    content.encoding = 'utf-8'
    content_new = content.text
    result = content_new.replace("\\/", "/").encode().decode('unicode_escape')
    
    return result
```
#建好list，将comment、link和title都一起抓进来
```
a=0
total2= []
url2= []
title = []
```

```
for i in data['links'][a:340]:
    comments = []
    web = re.findall(r'\d.*\d',i)
    url_api = 'https://m.hupu.com/api/v1/bbs-thread-frontend/{}?page=1&lzlist=0'.format(web[0])
    content_new = get_content(url_api)
    page_num = re.findall(r'"r_total_page":"(.*)",',content_new)
    if len(page_num) != 0:
        page_num_new = int(page_num[0])
        print(page_num_new)
        print('='*30)
        for ii in range(0,page_num_new):
            url_api_2 = 'https://m.hupu.com/api/v1/bbs-thread-frontend/{}?page={}&lzlist=0'.format(web[0],ii)
            content_new_2 = get_content(url_api)
            comment = re.findall(r'"content":"<p>(.*?)</p>',content_new_2)
            comment = list(set(comment))
            comments = comments + comment
            print(ii)
    
        total2.append(comments)
        url2.append(i)
        title.append(data['title'][a])
        time.sleep(2)
        print(a)
        a+=1
 ```
#最后保存就好

```
data_final = {
    'title':title,
    'comment':total2,
    'links':url2
}
data_final = pd.DataFrame(data_final)

data_final.to_csv("hupu_result.csv",index=False,sep=',')
```




