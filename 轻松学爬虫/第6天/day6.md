# day06

### 1，selenium +phantomjs

​	selenium是什么？是一个浏览器的自动化测试工具，通过selenium提供的一些方法可以去操作浏览器，让浏览器做一些自动化的工作
	selenium操作谷歌浏览器，核心：而是selenium操作谷歌浏览器驱动，通过驱动再来操作浏览器
	谷歌浏览器驱动下载地址
		http://chromedriver.storage.googleapis.com/index.html
		http://npm.taobao.org/mirrors/chromedriver/
	谷歌驱动和谷歌浏览器版本关系映射表
		http://blog.csdn.net/huilan_same/article/details/51896672
	安装selenium：pip install selenium
	【注】通过selenium操作浏览器的时候，一定要记得停顿，因为是真正的上网过程，要执行其中很多的请求，所以使用selenium非常的慢，效率低
	selenium操作有界面的浏览器做什么？
	phantomjs：是什么？是一款浏览器，它是一款无界面浏览器。就是专门用来写爬虫代码用的。肯定有浏览器的功能，可以将html、css、图片、js给你显示成图文并茂的形式，phantomjs可以执行网页中的js代码。
	网页的呈现形式，很多情况，html中的内容不是直接就有的，而是需要执行js代码，动态的给生成的
	（1）捕获接口，分析接口，然后向接口发送请求，得到数据，得到的数据一般都是json格式，然后再解析json数据即可
	（2）捕获不到接口，或者捕获到接口，看不懂接口参数，使用大招，selenium+phantomjs,因为它的效率低，慢
	selenium如何操作phantomjs？

```
from selenium import webdriver
import time

# 创建一个浏览器对象，通过谷歌驱动,可以直接指定谷歌驱动的路径，也可以将谷歌驱动放到环境变量中
path = r'C:\Users\ZBLi\Desktop\1803\day06\ziliao\chromedriver.exe'
browser = webdriver.Chrome(executable_path=path)

# 中间通过browser进行上网
url = 'http://www.baidu.com/'
browser.get(url)
time.sleep(3)

# 查找得到百度输入框
myinput = browser.find_element_by_id('kw')
# 往框里面写内容
myinput.send_keys('美女')
time.sleep(3)

# 查找得到百度一下按钮
button = browser.find_element_by_id('su')
button.click()
time.sleep(3)

# 通过连接的内容找到美女_百度百科
oa = browser.find_element_by_link_text('美女_百度百科')
oa.click()
time.sleep(3)

# 最后记得将浏览器退出
browser.quit()
```

​	模拟滚动条滚动到底部

```
from selenium import webdriver
import time
path = r'C:\Users\ZBLi\Desktop\1803\day06\ziliao\phantomjs-2.1.1-windows\bin\phantomjs.exe'
browser = webdriver.PhantomJS(path)

browser.get('https://movie.douban.com/typerank?type_name=%E7%88%B1%E6%83%85&type=13&interval_id=100:90&action=')
time.sleep(5)

browser.save_screenshot('./pic/douban1.png')

# 执行下面的js代码，将滚动条滚动到底部
js = 'document.body.scrollTop=10000'
# 在linux下面有时候有问题了,是前端的遗留问题，两个都测试一下即可
# js = 'document.documentElement.scrollTop=10000'
browser.execute_script(js)
time.sleep(3)

browser.save_screenshot('./pic/douban2.png')

# 如果要想要更多地页面数据，那一句js代码要执行多次，但是要注意scrollTop的值，如果页面高度非常大了，这个值也要变大
# 如果网页的下面有 加载更多 的按钮，那么通过browser的方法，找到这个按钮，模拟点击即可

# 得到执行js之后的代码
# browser.page_source
with open('douban.html', 'w', encoding='utf8') as fp:
    fp.write(browser.page_source)

# 再往下的工作就是将文档数据生成bs对象或者tree对象，再去解析即可

browser.quit()
```



​	得到执行js之后的代码

​		browser.page_source

```
from selenium import webdriver
import time
path = r'C:\Users\ZBLi\Desktop\1803\day06\ziliao\phantomjs-2.1.1-windows\bin\phantomjs.exe'
browser = webdriver.PhantomJS(path)

browser.get('http://www.baidu.com/')
time.sleep(3)

# 拍照片的方法
browser.save_screenshot('./pic/baidu1.png')

# 找到输入框
browser.find_element_by_id('kw').send_keys('清新')
# 点击百度一下
browser.find_element_by_id('su').click()
time.sleep(4)

browser.save_screenshot('./pic/baidu2.png')

browser.quit()
```



### 2，headlesschrome

谷歌进军无界面模式，phantomjs退出了，不在维护了。火狐也有，ie也有。美团

如果windows，要求谷歌版本在60+以上
	如果linux，要求谷歌版本在59+以上

```
	from selenium.webdriver.chrome.options import Options

	chrome_options = Options()

	chrome_options.add_argument('--headless')

	chrome_options.add_argument('--disable-gpu')

	chome_options.add_argument(('--proxy-server=http://' + ip))

```



案例如下

```
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time

chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')

path = r'C:\Users\ZBLi\Desktop\1803\day06\ziliao\chromedriver.exe'
browser = webdriver.Chrome(path, chrome_options=chrome_options)
browser.get('https://movie.douban.com/typerank?type_name=%E7%88%B1%E6%83%85&type=13&interval_id=100:90&action=')
time.sleep(3)

browser.save_screenshot('./pic/douban3.png')

browser.quit()
```



### 3，requests

​	是什么？是python的第三方库，这个库和urllib库是一个功能，模拟发送http请求，由于urllib库非常的不方便，所以写了这么一个库

林纳斯脱袜子-linux内核-bitkeep-git

```
pip install requests
```

http://docs.python-requests.org/zh_CN/latest/index.html

get请求

带参数get请求

data是一个原生字典

```
r = requests.get(url=url, headers=headers, params=data)
```

响应对象

```
	r.text  		字符串内容
	r.content	字节内容
	r.status_code	状态码
	r.url		请求url
	r.headers 	响应头
	r.encoding = 'utf8'	指定字符集
	r.encoding 		查看字符集	
	r.json == json.loads(r.text)	将json转化成python
```



案例

```
import requests

'''
url = 'https://www.baidu.com/s?'
data = {
    'ie': 'utf8',
    'wd': '王力宏'
}
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
}

r = requests.get(url, params=data, headers=headers)
'''
# print(r)
# 字符串格式内容
# print(r.text)
# 字节格式内容
# print(r.content)
# 获取发送请求的url
# print(r.url)
# 获取响应状态码
# print(r.status_code)
# 获取响应头
# print(r.headers)
# r.encoding = 'gbk'
# print(r.encoding)

url = 'https://movie.douban.com/j/chart/top_list?type=5&interval_id=100%3A90&action=&start=0&limit=20'
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
}
r = requests.get(url=url, headers=headers)
# print(r.text)
print(type(r.json()))
```





定制头部

##### 发送post请求

```
data是一个原生字典
r = request.post(url=url, headers=headers, data=data)
```

代码案例

```
import requests

post_url = 'https://cn.bing.com/ttranslationlookup?&IG=ECFF4CD98C524F66B5D55062F0E9D9E4&IID=translator.5036.14'
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
}
data = {
    'from': 'zh-CHS',
    'to': 'en',
    'text': '饺子',
}
r = requests.post(url=post_url, headers=headers, data=data)
print(r.text)
```



##### 代理机制

```
import requests

post_url = 'https://cn.bing.com/ttranslationlookup?&IG=ECFF4CD98C524F66B5D55062F0E9D9E4&IID=translator.5036.14'
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
}
data = {
    'from': 'zh-CHS',
    'to': 'en',
    'text': '饺子',
}
r = requests.post(url=post_url, headers=headers, data=data)
print(r.text)
```





##### 会话机制

​	和urllib里面一样

```
s = requests.Session()    创建一个会话，发送请求都使用s来进行发送
s.get()		s.post()
```

代码

```
import requests
import time

# 在代码中有没有一个东西和浏览器是一样的，能够保存cookie呢？下次发送的时候自动携带cookie

# 首先创建一个会话
s = requests.Session()

# 往下所有的请求，都是s里面的方法方法发送，那么就会自动保存cookie和携带cookie

post_url = 'http://www.renren.com/ajaxLogin/login?1=1&uniqueTimestamp=2018741029602'
headers = {
	'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
}
formdata = {
	'email': '17701256561',
	'password': 'lizhibin666',
	'icode': '',
	'origURL': 'http://www.renren.com/home',
	'domain': 'renren.com',
	'key_id': '1',
	'captcha_type': 'web_login',
	'f': 'https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3DVwDXbx3oN5RBHzVxzj2jwbsO3z8VmHcZ1HZQTdC3enq%26wd%3D%26eqid%3D834642bf0000b410000000055b6bac87',
}

r = s.post(post_url, data=formdata, headers=headers)

time.sleep(3)

get_url = 'http://www.renren.com/960481378/profile'

r = s.get(url=get_url, headers=headers)

with open('renren.html', 'wb') as fp:
	fp.write(r.content)
```





##### 异常处理

```
requests.exceptions
```

Timeout : 请求超时
ConnectionError ：主机不存在或者拒绝链接
HTTPError ：找不到文件资源



##### 取消ssl证书验证

```
verify=False   
from requests.packages import urllib3
urllib3.disable_warnings()
```

12306网站案例

```
import requests
# 忽略警告
from requests.packages import urllib3
urllib3.disable_warnings()

url = 'https://www.12306.cn/'
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
}
# 取消证书验证
r = requests.get(url=url, headers=headers, verify=False)
```



