### 1，正则替换

```
import re

string = '男生都喜欢20岁的女孩'

# ret = string.replace('18', '30')
# print(ret)

# 里面的数字无论多少都改为30，正则替换
# 将正则表达式匹配的内容替换为指定的内容
pattern = re.compile(r'\d+')
# 参数1：要替换的内容
# 参数2：在哪个字符串中查找
# ret = pattern.sub('30', string)


# 将这回调函数传递进去，该函数的要求是有一个参数，有一个返回值
# 参数就是正则匹配的对象，返回值的作用就是替换正则匹配的字符串
def fn(obj):
	# print(obj)
	age = int(obj.group())
	age += 1
	return str(age)

ret = pattern.sub(fn, string)

print(ret)

```

可以替换为固定的字符串

​	ret = pattern.sub('xxx', string)

也可以出难题一个函数，将函数的返回值替换匹配的内容

​	ret = parrern.sub(fn,string)



## Bs4

bs4是什么？是一个Python的第三方模块，用来解析html数据，其提供的api接口非常的人性化。

安装

```
pip install bs4
```

安装解析器

```
pip install lxml
```

有可能出现问题。将pip源，切换为国内源，豆瓣源、阿里源等

如何切换？

​	1）指令切换，  -i  源地址  只针对于这一次的指令安装有效

​	(2）永久切换，在指定地方写一个配置文件即可

​		windows

​			1）在文件资源管理器上面输入 %appdata%
			（2）手动创建一个pip的文件夹
			（3）新建一个文件 pip.ini
			（4）写入如下内容即可

```
	[global]
	timeout = 6000
	index-url = http://pypi.douban.com/simple  
	trusted-host = pypi.douban.com
```

liunx系统下

```
cd ~
mkdir ~/.pip
vim ~/.pip/pip.conf
编辑内容和windows的内容一模一样
```



##### 语法学习

```
from bs4 import BeautifulSoup
```



步骤：通过BeautifulSoup这个类，可以将你的html文档生成一个对象，然后这个对象会有一些方法供你使用，就可以得到你想要的节点内容、或者节点属性



可以将本地文件或者网络文件生成对象，先从本地开始学习



（1）根据标签名进行查找

```
soup.a
```

只能得到第一个符合要求的标签

（2）获取属性

```
soup.a.attrs 
```

返回一个字典，里面包含所有的属性和值

```
soup.a.attrs['href']
soup.a['href']
```

（3）获取标签内容

```
obj.string
obj.text
obj.get_text()
```

如果标签里面只有内容，那么三个获取结果都一样
如果标签里面还有标签，那么第一个获取的是None，后两个获取的是纯文本内容

（4）find方法

```
soup.find('a', title='xxx')
soup.find('a', id='xxx')
soup.find('a', class_='xxx')
```

返回一个对象，只能找到第一个符合要求的节点

***选看：soup.find('a', class_=re.compile(r'xxx'))  可以写正则表达式，一般用的不多

（5）find_all方法
返回一个列表,列表里面都是对象
用法和上面的find一样，只不过这个是找到所有，find只是找到一个

```
soup.find_all('a', limit=2)  取出前两个
```

（6）select方法

根据选择器得到自己想要的节点
常用的选择器：
标签选择器   

```
a  div
```

类选择器     

```
.lala  .dudu
```

id选择器     

```
#lala  #dudu
```

后代选择器

```
div .lala a     # 后面的是前面的子节点就行
div > p > a     # 后面的必须是前面的直接子节点才行
```

群组选择器   

```
div, #lala, .dudu
```

属性选择器   

```
input[name=xxx]  div[class=xxx]s
```

返回的是一个列表，就算选择器精确到一个，也是一个列表，列表中只有一个对象也可以通过子对象来查找内容，得到当前子对象里面符合要求的标签内容

### 3、bs4实例

三国演义：

```
import urllib.request
import time
from bs4 import BeautifulSoup

def handle_request(url):
	headers = {
		'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
	}
	request = urllib.request.Request(url=url, headers=headers)
	return request

def parse_first():
	url = 'http://www.shicimingju.com/book/sanguoyanyi.html'
	request = handle_request(url)
	# 发送请求，得到响应
	content = urllib.request.urlopen(request).read().decode('utf8')
	# 解析内容
	soup = BeautifulSoup(content, 'lxml')
	# 得到所有的章节链接
	oa_list = soup.select('.book-mulu > ul > li > a')
	fp = open('三国演义.txt', 'w', encoding='utf8')
	# print(oa_list)
	# print(len(oa_list))
	# 遍历列表，通过对象依次获取内容和链接
	for oa in oa_list:
		# 得到链接
		href = 'http://www.shicimingju.com' + oa['href']
		# 得到章节标题
		title = oa.text
		print('正在下载%s......' % title)
		# 向href发送请求，通过bs得到该章节的内容
		text = get_text(href)
		string = title + '\n' + text + '\n'
		fp.write(string)
		print('结束下载%s' % title)
		time.sleep(2)
	fp.close()

def get_text(href):
	request = handle_request(href)
	content = urllib.request.urlopen(request).read().decode('utf8')
	soup = BeautifulSoup(content, 'lxml')
	odiv = soup.find('div', class_='chapter_content')
	return odiv.text

def main():
	parse_first()

if __name__ == '__main__':
	main()
```

智联招聘：

```
import urllib.request
import urllib.parse
from bs4 import BeautifulSoup
import time

class ZhiLianSpider(object):
	def __init__(self, jl, kw, start_page, end_page):
		# 将这些都保存到成员属性中
		self.jl = jl
		self.kw = kw
		self.start_page = start_page
		self.end_page = end_page
		self.url = 'https://sou.zhaopin.com/jobs/searchresult.ashx?'
		self.headers = {
			'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
		}

	def run(self):
		# 打开文件
		self.fp = open('work.txt', 'w', encoding='utf8')
		# 要爬取多页，所以需要循环爬取
		for page in range(self.start_page, self.end_page + 1):
			print('正在爬取第%s页......' % page)
			request = self.handle_request(page)
			# 发送请求，获取响应
			content = urllib.request.urlopen(request).read().decode('utf8')
			# 解析内容
			self.parse_content(content)
			print('结束爬取第%s页' % page)
			time.sleep(2)
		self.fp.close()
	
	def parse_content(self, content):
		# 生成soup对象
		soup = BeautifulSoup(content, 'lxml')
		# 提取内容
		# 首先提取所有的table
		otable_list = soup.find_all('table', class_='newlist')
		# print(len(otable_list))
		# 过滤掉表头
		otable_list = otable_list[1:]
		# 遍历这个列表，依次提取每一个工作的详细信息
		for otable in otable_list:
			# 职位名称
			zwmc = otable.select('.zwmc > div > a')[0].text.rstrip('\xa0')
			# 公司名称
			gsmc = otable.select('.gsmc > a')[0].string
			# 职位月薪
			zwyx = otable.select('.zwyx')[0].string
			# 工作地点
			gzdd = otable.select('.gzdd')[0].string
			# print(zwmc)
			# exit()
			# 保存到字典中
			item = {
				'职位名称': zwmc,
				'公司名称': gsmc,
				'职位月薪': zwyx,
				'工作地点': gzdd,
			}
			self.fp.write(str(item) + '\n')

	def handle_request(self, page):
		# 拼接url
		# 将get参数写成一个字典
		data = {
			'jl': self.jl,
			'kw': self.kw,
			'p': page
		}
		# 处理data
		data = urllib.parse.urlencode(data)
		url = self.url + data
		# print(url)
		request = urllib.request.Request(url=url, headers=self.headers)
		return request

def main():
	# 工作地点
	jl = input('请输入工作地点:')
	# 工作关键字
	kw = input('请输入工作关键字:')
	# 起始页码和结束页码
	start_page = int(input('请输入起始页码:'))
	end_page = int(input('请输入结束页码:'))

	# 面向对象封装
	zhilian = ZhiLianSpider(jl, kw, start_page, end_page)
	zhilian.run()

if __name__ == '__main__':
	main()
```

### 4、xpath简介

​	xml是什么？被设计用来传输和存储数据，和json同处于一个位置，但是目前以json居多

​	xml和html的不同点
	（1）xml用来传输数据，html用来显示数据
	（2）xml的标签没有被预定义，html的标签是预定义好的
	（3）xml具有自我描述性



​	常用的路径表达式
	/ : 从根节点开始查找
	// : 从任意位置开始查找
	. : 从当前节点开始查找
	.. : 从父节点开始查找
	@ ：选取属性



路径表达式举例：
	bookstore/book		 : 从bookstore下面找book节点，book必须是bookstore的直接子节点
	bookstore//book 		: 从bookstore下面找book节点, book可以使直接子节点也可以是孙子节点
	//book		 : 从文档中任意位置找book节点
	//@lang		 : 查找所有拥有lang属性的节点
	bookstore/book[1]		 : 直接子节点第一个book，下标从1开始
	bookstore/book[last()] 		: 直接子节点最后一个book
	bookstore//book[last()]		 : 所有book里面的最后一个
	bookstore/book[last() - 1] 		: 倒数第二个book
	bookstore/book[position()<3] 		: 取出前两个book
	//title[@lang]		 : 所有拥有lang属性的title节点
	//title[@lang=eng] 		: 所有的lang属性值为eng的title节点
	bookstore/* 		: bookstore下面所有的直接子节点
	bookstore//* 		: bookstore下面所有的子节点
	//title[@*]		 : 有属性的title节点
	//book/title | //book/price		 ： 找到所有book节点下面的直接子节点title和price



	xpath函数
		contains : 包含
		starts-with : 以某某开头
		ends-with : 以某某结尾（貌似不能用）


​	我们学的xpath是用在了html中，由于html和xml很像，所以有一个第三方库就封装了使用xpath来解析html数据的接口

```
pip install lxml
```

使用的时候，首先需要使用插件进行测试xpath，然后再来到代码中进行编写
	xpath插件使用：
	（1）属性筛选
		//input[@id="kw"]  找到属性id值为kw的input节点
	（2）层级筛选
		通过属性和层级筛选
		//div[@id="head"]/div/div[@id="u1"]/a[2]