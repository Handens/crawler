# day03

### 1，代理

在代理网站上，复制一点ip和端口号，存入txt文件当中，我创建的名字是pool.txt

```
import urllib.request
import random
import time

url = 'http://www.baidu.com/#ie=UTF-8&wd=ip'
headers = {
	'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36'
}
request = urllib.request.Request(url=url, headers=headers)

# 读取文件
fp = open('pool.txt', 'r')
string = fp.read()
fp.close()

# print(string)
# 将字符串按照换行符切割，得到一个列表，列表里面就是一个一个的代理服务器
lt = string.splitlines()

# print(lt)
while 1:
	# 从列表中随机抽取一个代理
	daili = random.choice(lt)
	# 发送请求
	proxy = {'http': daili}
	# 创建handler
	handler = urllib.request.ProxyHandler(proxies=proxy)
	# 创建opener
	opener = urllib.request.build_opener(handler)

	try:
		response = opener.open(request)
		print('使用代理%s成功' % daili)

		with open('ip.html', 'wb') as fp:
			fp.write(response.read())
		break
	except Exception as e:
		# 将这个代理从你的列表移除
		lt.remove(daili)
		print('使用代理%s失败' % daili)
	time.sleep(2)

```

### 2 cookies使用

##### cookies是什么

​	客户端     服务器

​	每一次请求都是单独的请求，请求之间没有任何关系，登录时候是一个请求

访问登录的页面又是一个请求，这两个请求必须有关系，所以引入了cookies

​	登录的时候，服务端给你响应，在响应里面就有cookies，浏览器就会将cookies保存起来，下次再请求的时候，就会带着cookie来访问



​	需求：通过代码访问登录后的页面

​	（1）通过抓包

​	首先让浏览器登录成功，然后浏览器再访问登录后页面的时候，你来抓包，抓取到请求头里面的cookie信息，然后写到代码中即可

```
import urllib.request

url='*****'    #改成自己的url
headers = {
	'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
	'Cookies':'浏览器赋值的cookies'
}
request = urllib.request.Request(url=url, headers=headers)

r = urllib.request.urlopen(request)

with open('文件名.后缀名','wb') as fp:
	fp.write(r.read())
```

​	

（2）模拟登录

​	首先抓包抓取post请求，通过代码模拟发送post请求，然后创建ck对象，用来保存和携带cookies即可

```
import http.cookiejar

# 创建一个cookiejar,用来储存cookie
ck = http.cookiejar.CookieJar()
# 根据ck创建一个handler
handler = urllib.request.HTTPCookieProcessor(ck)
opener = urllib.request.build_opener(handler)
```





### 3,正则表达式

##### 	为什么引入正则表达式？

​	字符串的函数  	可以查找所有的邮箱，可以验证邮箱格式是否正确，可以匹配一类东西

​	因为有正则，学习正则就是学习规则

​	四大难懂的东西：女人的心、医生的处方、道士的符、程序猿的正则

##### 	

##### 	单字符规则：

```
		\d：所有的数字字符

		\D ：非\d

		\w：数字、字母、下划线、中文

		\W：非\w

		\s：匹配所有的空白字符    \n  \t  空格

		\S：非\s

		[ ]：[abcd]中的一个（仅限一个）

		[^abcd]：里面加一个‘^’除了里面写的都能匹配

		. ：除了\n以外任意字符
```

##### 	数量修饰：

```
		{m} : 修饰前面的字符出现m次

		{m,}：修饰前面的字符最少m次

		{m,n}：最少m次，最多n次

		{0, }：任意多次    *

		{1, }：最少一次   +

		{0,1}：可有可无   ？
```

##### 	在python里面如何使用

```
		import	re

		pattern = re.compole(r'xxx')

		re.match( )

		re.search()

		re.findall()

```



```
import re

string = 'i love you very much'
pattern = re.compile(r'love')

ret = pattern.match(string)

print(ret)    # 此时返回值是none，因为从字符串开头查找、找到一个结束

print(ret.group())     # 匹配的第一个的内容
print(ret.span())	#匹配的位置

pattern.search()

```

​	边界修饰：

```
	^：以某某开头

	$：以某某结尾

	分组：（正则的高级功能）

	（）

	1、视为一个整体  		(a\d){5}

	2,子模式、分组

		（？P<goudan>)   (?P=goudan)

		\1      \2   第一、二个小括号匹配的内容

		$1     	$2	 第一、二个小括号匹配的内容

		如果有子模式  ret.group(1)就是第一个子模式匹配的内容

	贪婪：

		.*

		.+

		.*?      取消贪婪

		.+？	

```

在python里面如何使用？

```
import re

pattern = re.compile(r'xxx')

pattern.match()		# 从字符串开头查找，找到一个结束
pattern.search()	# 从字符串任意位置开始查找，找到一个立马结束
re.group()	# 得到匹配内容
re.span()	# 得到匹配位置
pattern.fallall()	# 查到所有匹配的内容，得到的是一个列表
```



模式修正：

```
	re.I	# 忽略大小写

	re.M	# 视为多行模式

	re.S	# 视为单行模式
```



### 4、正则案例

案例：糗事百科糗图

图片下载：

​	防盗链，直接吐过urllib.request.urlretrieve()

​	在请求头部有一个referer，判断头部是不是从这个网站过来的，如果是，可以看图片，如果不是，图片不让看，通过程序看图片的时候，需要手动定制Referer:，定制为网站的首页即可。需要通过构建请求对象，发送请求，将响应写到文件中这种进行下载

```
import urllib.request
import urllib.parse
import time
import re

def main():
	start_page = int(input('请输入起始页码:'))
	end_page = int(input('请输入结束页码:'))
	url = 'http://www.yikexun.cn/lizhi/qianming/list_50_{}.html'
	# 打开文件
	fp = open('lizhi.html', 'w', encoding='utf8')
	for page in range(start_page, end_page + 1):
		# 构建请求对象
		request = handle_request(url, page)
		# 发送请求，得到响应
		content = urllib.request.urlopen(request).read().decode('utf8')
		# 解析内容
		parse_content(content, fp)
		time.sleep(2)
	fp.close()

def parse_content(content, fp):
	pattern = re.compile(r'<div class="art-t">.*?<a href="(.*?)">(<b>)?(.*?)(</b>)?</a>.*?</div>', re.S)
	ret = pattern.findall(content)

	# 遍历列表，取出标题和链接
	for tp in ret:
		href = 'http://www.yikexun.cn' + tp[0]
		title = tp[2]
		# 取出b标签
		# title = title.strip('</b>')
		text = get_text(href)
		# 打开文件，写入文件中
		string = '<h1>%s</h1>%s' % (title, text)
		fp.write(string)
		time.sleep(2)


def get_text(href):
	# 构建请求对象
	request = handle_request(href)
	content = urllib.request.urlopen(request).read().decode('utf8')
	pattern = re.compile(r'<div class="neirong">(.*?)</div>', re.S)
	ret = pattern.search(content)
	# print(ret.group(1))
	# exit()
	return ret.group(1)

def handle_request(url, page=None):
	if page:
		url = url.format(page)
	headers = {
		'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36'
	}
	request = urllib.request.Request(url=url, headers=headers)
	return request

if __name__ == '__main__':
	main()
```





