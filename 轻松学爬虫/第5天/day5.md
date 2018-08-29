# day05



### 1，xpath使用

##### 1）xpath使用

​	1)属性筛选

```
	//input[@id ='kw']

	//span[@class='全称']

```

​	通过class进行选择的时候，需要将所有class全部写进来才可以

##### 2）索引和层级

```
//div[@id='head']/div/div[@id='id']/a[2]
```

3）获取内容和属性

```
//div[@id='id']/a[2]/text()  # 获取内容

//div[@id='ul']/a[2]/@herf	# 获取属性
```

（4）xpath函数

选出class属性以mn开头的a

```
//div[@id="u1"]/a[starts-with(@class,"mn")]
```

选出内容以地开头的a

```
//div[@id="u1"]/a[starts-with(text(),"地")]
```

选出class属性包含av的a

```
//div[@id="u1"]/a[contains(@class,"av")]
```

选出内容包含多产的a

```
//div[@id="u1"]/a[contains(text(),"多产")]
```



带啊中使用xpath，首先将文档生成对象，然后根据对象的方法

代码中使用xpath，首先将文档生成对象，然后根据对象的方法找到想要的节点。可以是本地文件，也可以是网络文件
	xpath方法返回的是一个列表，通过下标进行取内容

```
//*[@id="u1"]/a[2]

//*[@id="su"]    #su   #form > input[type="hidden"]:nth-child(14)
```



```
from lxml import etree

# 生成对象
tree = etree.parse('test.html')
# print(tree)

# ret = tree.xpath('//a[@class="lala"]')
# ret = tree.xpath('//div[@class="tang"]/ul/li/b[2]')
# ret = tree.xpath('//a[@id="dudu"]/text()')
# ret = tree.xpath('//a[@id="dudu"]/@href')
# print(ret)


# 获取标签里面还有标签的内容可以使用如下两种方式
# ret = tree.xpath('//div[@class="hero"]//text()')
# string = ''.join(ret).replace('\n', '').replace(' ', '')
# print(string)

# 先找到div这个对象
# odiv = tree.xpath('//div[@class="hero"]')[0]
# string = odiv.xpath('string(.)').replace('\n', '').replace(' ', '')
# print(string)

# ret = tree.xpath('//b[starts-with(@class,"mei")]/text()')
# ret = tree.xpath('//b[contains(text(), "一")]/text()')
# print(ret)
```

```
import urllib.request
from lxml import etree
import time
import os

class TuPianSpider(object):
    def __init__(self, start_page, end_page):
        self.start_page = start_page
        self.end_page = end_page
        self.url = 'http://sc.chinaz.com/tupian/meinvtupian_{}.html'
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
        }
    
    def run(self):
        for page in range(self.start_page, self.end_page + 1):
            print('正在下载第%s页......' % page)
            request = self.handle_request(page)
            content = urllib.request.urlopen(request).read().decode('utf8')
            self.parse_content(content)
            print('结束下载第%s页' % page)
            time.sleep(2)
    
    def parse_content(self, content):
        # 解析内容
        tree = etree.HTML(content)
        # 提取这个页码所有图片的src属性
        image_srcs = tree.xpath('//div[@id="container"]/div/div/a/img/@src2')
        # 找到图片的标题
        image_titles = tree.xpath('//div[@id="container"]/div/div/a/img/@alt')
        # print(image_srcs)
        # print(len(image_srcs))
        # 遍历列表下载图片
        for image_src in image_srcs:
            # 生成图片的名字
            filename = image_titles[image_srcs.index(image_src)] + '.' + image_src.split('.')[-1]
            print('正在下载--%s...' % filename)
            dirname = 'meinv'
            filepath = os.path.join(dirname, filename)
            urllib.request.urlretrieve(image_src, filepath)
            print('结束下载--%s' % filename)
            time.sleep(2)
    
    def handle_request(self, page):
        if page == 1:
            url = 'http://sc.chinaz.com/tupian/meinvtupian.html'
        else:
            url = self.url.format(page)
        # print(url)
        request = urllib.request.Request(url=url, headers=self.headers)
        return request


def main():
    start_page = int(input('请输入起始页码:'))
    end_page = int(input('请输入结束页码:'))
    tupian = TuPianSpider(start_page, end_page)
    tupian.run()

if __name__ == '__main__':
    main()
```





### 2，xpath案例

懒加载：一个网页中有100个图片。需要101次请求才能加载完毕。首先加载出现在可视区内的图片，在可视区外的图片多没有加载。在用户滚动滚动条的时候，通过js动态的去监听当前这个图片有没有出现在可视区，如果出现在可视区，再去加载这个图片
	前端如何实现？

```
  <img src='xxx'> 			<img src2='图片地址'>
```

当图片出现在可视区的时候，js动态的将src2修改为src，就会加载这个图片
	出现的形式：
		src2   data-src   data-original  class="lazy"
	糗事百科
		谷歌中的xpath只能作为参考，实际都要以代码为准

糗事百科为例：

```
import urllib.request
from lxml import etree

def handle_request(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
    }
    return urllib.request.Request(url=url, headers=headers)

def parse_content(content, fp):
    tree = etree.HTML(content)
    # 获取每一个段子的所有信息
    odiv_list = tree.xpath('//div[@id="content-left"]/div')
    # print(len(odiv_list))
    for odiv in odiv_list:
        # 获取头像
        try:
            image_src = odiv.xpath('./div[@class="author clearfix"]/a/img/@src')[0]
        except Exception as e:
            image_src = ''
        
        # 获取用户名
        name = odiv.xpath('./div[@class="author clearfix"]//h2/text()')[0]
        # 获取年龄
        try:
            age = odiv.xpath('./div[@class="author clearfix"]/div/text()')[0]
        except Exception as e:
            age = ''
        
        # 获取内容  自己获取
        item = {
            '头像': image_src,
            '用户名': name,
            '年龄': age
        }
        fp.write(str(item) + '\n')

def main():
    fp = open('qiu.txt', 'w', encoding='utf8')
    url = 'https://www.qiushibaike.com/'
    request = handle_request(url)
    response = urllib.request.urlopen(request)
    content = response.read().decode('utf8')
    parse_content(content, fp)
    fp.close()

if __name__ == '__main__':
    main()
```



### 3，json数据解析

认识json？

前台：用户看到的界面称之为前台

后台：管理员负责的查看的，后台管理系统

前端：html、css、js，在浏览器那一块工作的

后端：服务器端的，php，java，python

短信发送平台。会提供单独的接口

天气接口。

前端后端交互一般都是json格式。

后端服务器和特定功能服务器之间的交互，后端服务器称之为前端，提供服务的称之为后端，之间交互的格式也是json格式

http://blog.csdn.net/luxideyao/article/details/77802389



json语法：

(1) 数据都字啊键值对中

(2)数据以逗号隔开

(3){}保存对象

(4)[]保存数组

字符串都是以双引号括起来的

json的值可以是：数字、字符串、逻辑值、数组、字典、null

```
import json

lt = [
    {'name': '王静', 'age': '7', 'height': 130},
    {'name': '闫晓红', 'age': 13, 'height': 160},
    {'name': '刘惠芬', 'age': 16, 'height': 158},
    {'name': '赵铁柱', 'age': 20, 'height': 170}
]

string = json.dumps(lt, ensure_ascii=False)

# print(string)
obj = json.loads(string)

print(type(obj))
```





Python如何解析json？

（1）原生解析，通过字典、列表解析

​	import  json

​	import.dumps()：将python的字典或者列表转化为json格式字符串

​	import.loads()：将json格式的字符串转化为python的对象

​	import.dump()：将python对象转化为json字符串之后直接写入到文件中

​	import.load()：将文件中的json字符串直接读到python对象中



（2）jsonpath

jsonpath用来干什么？用来解析json数据,当解析复杂一点的json数据就要用到这个。

```
obj[0]['lala']['goudan'][2]['dudu']['xixi']
```


​		http://blog.csdn.net/luxideyao/article/details/77802389
		安装：

```
pip install lxml
pip install jsonpath
```

和xpath对比
		/	$	根元素
		/	.	路径分隔符，直接子元素
		.	@	当前元素
		//	..	任意位置开始查找
		下标xpath从1开始，jsonpath从0开始

```
import json
import jsonpath

fp = open('book.json', 'r', encoding='utf8')
string = fp.read()
fp.close()

obj = json.loads(string)

# 得到所有book的author
ret = jsonpath.jsonpath(obj, '$.store.book[*].author')
# 直接找到所有的author
ret = jsonpath.jsonpath(obj, '$..author')
# 得到store下面的所有的price
ret = jsonpath.jsonpath(obj, '$.store..price')
# 得到第3本书
ret = jsonpath.jsonpath(obj, '$..book[2]')
# 得到最后一本书
ret = jsonpath.jsonpath(obj, '$..book[(@.length-1)]')
# 
print(ret)
```

淘宝评论为例

```
import json
import jsonpath
import urllib.request

url = 'https://rate.tmall.com/list_detail_rate.htm?itemId=562099309982&spuId=893336129&sellerId=2616970884&order=3&currentPage=2'
headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
    }
request = urllib.request.Request(url=url, headers=headers)
content = urllib.request.urlopen(request).read().decode('gbk')
content = '{' + content + '}'
# print(content)
fp = open('taobao.txt', 'w', encoding='utf8')
obj = json.loads(content)
# print(type(obj))
# 找到所有的评论列表
comments_list = obj['rateDetail']['rateList']
# 遍历列表，依次获取每一个评论内容
for com_obj in comments_list:
    # 评论内容
    comment = com_obj['rateContent']
    # 用户名
    name = jsonpath.jsonpath(com_obj, '$..displayUserNick')[0]
    # 上传照片
    images = jsonpath.jsonpath(com_obj, '$..pics')[0]
    # 评论时间
    tt = jsonpath.jsonpath(com_obj, '$..rateDate')[0]
    # print(images)
    # print('*' * 50)
    item = {
        '评论内容': comment,
        '用户名': name,
        '照片': images,
        '评论时间': tt
    }
    fp.write(str(item) + '\n')

fp.close()
```

