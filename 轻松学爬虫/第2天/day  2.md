# day  2

### 1,urllib.parse

​	处理参数或者url的

```
url中只能使用：字母、数字、下划线、冒号 // ? =等
如果有其他字符，则必须使用编码

urllib.parse.quote()   编码    但是编码的时候包括符号等都编码了

urllib.parse.unquote()   解码    【注】编码的时候只需要编码参数即可

urllib.parse.urlencode(data)    编码 
	data是一个字典，直接将字典的键和值转化成query_string格式，并且实现url编码

```

### 2 构建

```
# 以下代码可能要出现ssl错误 如果出现请添加
import urllib.request
# import ssl
#ssl.create_default_https_context = ssl.create_unverified_context

url = 'http://www.baidu.com/'

# 如何定制UA     UA全称User-Agent ，上网搜索一下就可以了,  在线查看自己的UA，我用的是UC浏览器
# 在这个头部不仅可以定义UA，还可以定制其他的请求头部，一般只需要定制UA
headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.4094.1 Safari/537.36'
}

# 构建请求对象
request = urllib.request.Request(url = url ,headers = headers)

# 发送请求,直接打开这个请求对象即可
request = urllib.request.urlopen(request) 
```

### 3，模拟各种请求

##### 模拟 ’GET‘ 请求

```
# 我们用百度搜索关键字，我们查看地址栏如下

import urllib.request
import urllib.parse

#  让用户输入搜索关键字
keyword = input('请输入要搜索的关键字：')
url = 'http://www.baidu.com/s?'

# get参数
data = {
    'ie' = 'utf8',
    'wd' = keyword,
}
query_string = urllib.parse.urlencode(data)

url += query_string

# 想url发送请求，得到响应
headers = {
    'User-Agent' : User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.4094.1 Safari/537.36'
}

request = urllib.request.Request(url = url , headers = headers)

response = urllib.request.urlopen(request)

# 拼接文件名字
filename = keyword + '.html'

# 写到文件当中
with open(filename,'wb') as fp:
	fp.write(response.read())

```



模拟 "POST" 请求

```
import urllib.request
import urllib.parse

rul = 'fanyi.baidu.com/sug'

# 将表单数据携程一个字典
formdata = {
    'kw' : 'wolf'
}
# formdata单独处理一下 ,转化字节格式
formdata = urllib.parse.urlencode(formdata).encode()

headers = {
    'User-Agent' : User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.4094.1 Safari/537.36'
}

# 构建请求对象
request = urllib.request.Request(url = url, headers = headers)
response = urllib.request.urlopen(request,data = formdata)

print(request.read().decode)
```

urlopen(url,  data = None)

​	如果有data，代表是post请求，如果没有data，代表的是get请求，get的参数需要拼接到url的后面

read才能读取字节类的内容

##### ajax-get跟前面的get相似，主要是发送请求的URL不同

```
import urllib.request
import urllib.parse

url = 'https://movie.douban.com/j/chart/top_list?type=5&interval_id=100%3A90&action=&'
print('每页显示10条数据')
page = int(input('请输入页码:'))
# 根据page计算出来start和limit
start = (page-1) * 10
limit = 10
data = {
	'start': start,
	'limit': limit,
}
url += urllib.parse.urlencode(data)

headers = {
    'User-Agent' : User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.4094.1 Safari/537.36'
}
request = urllib.request.Request(url=url, headers=headers)

response = urllib.request.urlopen(request)


print(response.read().decode('utf8'))
```



##### ajax-post跟前面的post相似，主要是发送请求的URL不同

```
import urllib.request
import urllib.parse

url = 'http://www.kfc.com.cn/kfccda/ashx/GetStoreList.ashx?op=cname'
city = input('请输入要搜索的城市:')
data = {
	'cname': city,
	'pid': '',
	'pageIndex': '1',
	'pageSize': '10'
}
data = urllib.parse.urlencode(data).encode('utf8')
headers = {
    'User-Agent' : User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.4094.1 Safari/537.36'
}
request = urllib.request.Request(url=url, headers=headers)

response = urllib.request.urlopen(request, data=data)

print(response.read().decode('utf8'))
```





### 4，URLError\HTTPError

是异常处理类，属于urllib.error这个模块

URLError：断网或者注意不存在的时候触发

```
# 主机不存在的时候

import urllib.request
import urllib.error

url = 'http://www.maodan.com/'

try:
	response = urllib.request.urlopen(url)
except Exception as e:
	print(e)

print('不影响这一句代码的运行')
```

可以看到主机不存在，仍然可以运行。系统不会崩溃。
Exception是系统的一种异常捕获基类，所有的异常都属于它。

URLError是Exception的子类，父类可以捕获子类异常。

HTTPError是URLError的子类。

代码需要精确捕获的时候，子类放上面，父类放下面



### 5，复杂的get

我们以百度贴吧为例

```
import urllib.request
import urliib.parse
import os
import time

def main():
	baming = int(input('请输入要爬去的贴吧的名字：'))
	start_page = int(input('请输入要爬去的起始页码：'))
	end_page = int(input('请输入要爬去的结束页码：'))
	url = 'http://tieba.baidu.com/f?'
	
	for page in range(start_page, end_page + 1):
		print('正在爬去第%s页.....' % page)
		# 根据url和page拼接指定页码的url
		request = handle_request(page, baming, url)
		#根据请求对象发送请求得到响应写入指定的文件当中
		down_load(request)
		print('结束爬去第%s页' % page)
		time.sleep(3)

def down_load(request,baming, page):
	response = urllib.request.urlopen(request)
	# 通过代码创建指定的文件夹
	dirname = baming
	$判断不存在的时候
	if not os.path.exists(dirname)
		os.mkdir(dirname)
	# 文件的名字
	filename = '第%s页.html' % page
	filepath = os.path.join(dirname, filename)
	# 将内容直接写入到filepath中
	with open(filepath, 'wb') as fp:
		fp.write(response.read())
	
		
def handle_request(page, baming, url):
	# 拼接URL
	data = {
        'kw':baming,
        'ie':'utf8',
        'pn':pn
	}
	url += urllib.parse.urlencode(data)
	headers = {
        'User-Agent' : User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.4094.1 Safari/537.36'
	}
	
	# 构建请求对象
	urllib.request.Request(url, headers=headers)
	return request

if __name__ == '__main__'
```



### 6，Handler处理器，自定义opener

​	urlopen（）

​	请求对象

​	为了解决代理和cookie这些更加高级的功能而引入的

​	实现最简单的功能，高级功能的步骤和这个步骤一模一样

```
import urllib.request

url = 'http://www.baidu.com/'

# 创建handler
handler = urllib.request.HTTPHandler()
# 根据handler创建opener
opener = urllib.request.build_opener(handler)

# 发送请求的时候，不要使用urlopen，使用opener.open()
response = opener.open(url)

print(response.read().decode(utf8))
```



### 7 ，代理

​	生活中代理：代练、代孕、代购

​	程序中：

​	代理服务器：快代理、西刺代理、芝麻代理、阿布云代理

​	1）浏览器如何设置代理

​	2）代码中如何设置代理

```
import urllib.request
from bs4 import BeautifulSoup
import re
import time
import random

# --------------------公用方法-----------------------------
class CommanCalss:
    def __init__(self):
        self.header={'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.94 Safari/537.36'}
        self.testurl="www.baidu.com"

    def getresponse(self,url):
        req = urllib.request.Request(url, headers=self.header)
        resp = urllib.request.urlopen(req, timeout=5)
        content = resp.read()
        return content

    def _is_alive(self,proxy):
        try:
            resp=0
            for i in range(3):
                proxy_support = urllib.request.ProxyHandler({"http": proxy})
                opener = urllib.request.build_opener(proxy_support)
                urllib.request.install_opener(opener)
                req = urllib.request.Request(self.url, headers=self.header)
                # 访问
                # try:
                resp = urllib.request.urlopen(req, timeout=5)
                # except Exception as e:
                #     pass
                
            if resp == 200:
                return True
        except:
            return False



 # -----------代理池-----------------------
class ProxyPool:
    def __init__(self,proxy_finder):
        self.pool=[]
        self.proxy_finder=proxy_finder
        self.cominstan=CommanCalss()

    def get_proxies(self):
        self.pool=self.proxy_finder.find()
        for p in self.pool:
            if self.cominstan._is_alive(p):
                continue
            else:
                self.pool.remove(p)

    def get_one_proxy(self):
        return random.choice(self.pool)

    def writeToTxt(self,file_path):
        try:
            fp = open(file_path, "w+")
            for item in self.pool:
                fp.write(str(item) + "\n")
            fp.close()
        except IOError:
            print("fail to open file")


#----------------------获取代理方法---------------------
#定义一个基类
class IProxyFinder:
    def __init__(self):
        self.pool = []

    def find(self):
        return

#西祠代理爬取
class XiciProxyFinder(IProxyFinder):
    def __init__(self, url):
        super(XiciProxyFinder,self).__init__()
        self.url=url
        self.cominstan = CommanCalss()

    def find(self):
        for i in range(1, 10):
            content = self.cominstan.getresponse(self.url + str(i))
            soup = BeautifulSoup(content, 'lxml')
            ips = soup.findAll('tr')
            for x in range(2, len(ips)):
                ip = ips[x]
                tds = ip.findAll("td")
                if tds == []:
                    continue
                ip_temp = tds[1].contents[0] + ":" + tds[2].contents[0]
                self.pool.append(ip_temp)
        time.sleep(1)
        return  self.pool

#---------------------测试--------------------------------
if __name__ == '__main__':
    finder = XiciProxyFinder("http://www.xicidaili.com/wn/")
    ppool_instance = ProxyPool(finder)
    ppool_instance.get_proxies()
    ppool_instance.writeToTxt("proxy.txt")
```

生成一批代理