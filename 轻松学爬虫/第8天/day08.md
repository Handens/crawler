# day08

### 1、登录古诗文

​    登录：直接发送post，然后发送get
    登录：先发送get，获取一下信息，然后再发送post，然后发送get
    登录：get、post、get、get。  访问登录后的页面
    验证码，下载到本地，手动输入

```
import requests
from bs4 import BeautifulSoup
import urllib.request
from PIL import Image
import pytesseract
import time

def shibie(image_path):
    img = Image.open(image_path)
    # 转化为灰度图片
    img = img.convert('L')
    # 二值化处理
    threshold = 140
    table = []
    for i in range(256):
        if i < threshold:
            table.append(0)
        else:
            table.append(1)
    out = img.point(table, '1')

    # out.show()
    # 识别图片
    img = img.convert('RGB')

    return pytesseract.image_to_string(img)

# 创建一个会话
s = requests.Session()

i = 1

while 1:
    login_url = 'https://so.gushiwen.org/user/login.aspx?from=http://so.gushiwen.org/user/collect.aspx'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
    }
    r_log = s.get(url=login_url, headers=headers)

    # 将验证码图片下载到本地
    soup = BeautifulSoup(r_log.text, 'lxml')
    image_src = 'https://so.gushiwen.org' + soup.find('img', id="imgCode")['src']
    # urllib.request.urlretrieve(image_src, 'code.png')
    r_image = s.get(image_src, headers=headers)
    with open('code.png', 'wb') as fp:
        fp.write(r_image.content)

    # 获取表单隐藏框中的值
    views = soup.find('input', id="__VIEWSTATE")['value']
    viewg = soup.find('input', id="__VIEWSTATEGENERATOR")['value']

    # 发送post请求
    # code = input('请输入验证码------')
    code = shibie('code.png')

    post_url = 'https://so.gushiwen.org/user/login.aspx?from=http%3a%2f%2fso.gushiwen.org%2fuser%2fcollect.aspx'
    formdata = {
        '__VIEWSTATE': views,
        '__VIEWSTATEGENERATOR': viewg,
        'from': 'http://so.gushiwen.org/user/collect.aspx',
        'email': '1090509990@qq.com',
        'pwd': '123456',
        'code': code,
        'denglu': '登录',
    }
    r_post = s.post(url=post_url, headers=headers, data=formdata)

    # with open('gushi.html', 'wb') as fp:
    #     fp.write(r_post.content)
    # 判断是否登录成功
    if '修改昵称' in r_post.text:
        print('亲，第%s次登录成功------' % i)
        break
    print('不好意思，第%s次登录失败' % i)
    i += 1
    time.sleep(2)
```





### 自动识别验证码

​    （1）光学识别  tesseract
        指令识别
            识别率不行，训练它，有兴趣自己看看
        代码识别

```
pip install pytesseract
pip install pillow
```



 **直接可以用的验证码处理代码**

通过图像处理处理一下图片，然后再去识别，提高识别率

```
# 转化为灰度图片
img = img.convert('L')

# 二值化处理
threshold = 140
table = []
for i in range(256):
    if i < threshold:
        table.append(0)
    else:
        table.append(1)
out = img.point(table, '1')

img = img.convert('RGB')
enhancer = ImageEnhance.Color(img)
enhancer = enhancer.enhance(0)
enhancer = ImageEnhance.Brightness(enhancer)
enhancer = enhancer.enhance(2)
enhancer = ImageEnhance.Contrast(enhancer)
enhancer = enhancer.enhance(8)
enhancer = ImageEnhance.Sharpness(enhancer)
img = enhancer.enhance(20)







# 验证码识别，此程序只能识别数据验证码  

import Image    
import ImageEnhance    
import ImageFilter    
import sys    
from pytesser import *  
# 二值化    
threshold = 140    
table = []    
for i in range(256):    
	if i < threshold:    
		table.append(0)    
	else:    
		table.append(1)    
 
#由于都是数字    
#对于识别成字母的 采用该表进行修正    
rep={'O':'0',    
	'I':'1','L':'1',    
	'Z':'2',    
	'S':'8'    
	};    
 
def  getverify1(name):          
	#打开图片    
	im = Image.open(name)    
	#转化到灰度图  
	imgry = im.convert('L')  
	#保存图像  
	imgry.save('g'+name)    
	#二值化，采用阈值分割法，threshold为分割点   
	out = imgry.point(table,'1')    
	out.save('b'+name)    
	#识别    
	text = image_to_string(out)    
	#识别对吗    
	text = text.strip()    
	text = text.upper();      
	for r in rep:    
		text = text.replace(r,rep[r])     
	#out.save(text+'.jpg')    
	print text    
	return text    
getverify1('1.jpg')  #注意这里的图片要和此文件在同一个目录，要不就传绝对路径也行
```



​            
​    （2）打码平台
        云打码



### scrapy

​    scrapy是什么？是一个非常强大、精悍的Python网络爬虫框架，它的底层使用Python语言实现的，肯定就都集成了多进程、多线程、队列等技术，人家会给我们留下一些接口，只需要将精力放到提取url和解析页面中即可
	http://scrapy-chs.readthedocs.io/zh_CN/0.24/intro/tutorial.html
    http://www.index.html?username=goudan&password=123
    http://www.index.html?password=123&username=goudan
    url指纹去重

安装：

```
pip install scrapy
```


​    （1）认识框架
        引擎（engine）\爬虫文件（spider）\调度器（scheduler）\下载器（downloader）\管道（pipeline）
    （2）使用框架
        scrapy startproject xxx
    （3）认识目录结构
        firstbloodpro               工程总目录
            firstbloodpro           工程目录
                __pycache__         缓存目录
                spiders             爬虫目录
                    __pycache__     缓存目录
                    __init__.py     包的标记
                    lala.py         爬虫文件（*）
                __init__.py         包的标记
                items.py            定义数据结构的地方（*）
                middlewares.py      中间件
                pipelines.py        管道文件（*）
                settings.py         配置文件（*）
            scrapy.cfg              工程配置信息（一般不用）
    （4）生成爬虫文件
        cd firstbloodpro
        scrapy genspider xxx www.xxx.com
        生成的参数的意思见  qiubai.py
    （5）运行起来
        cd firstbloodpro/firstbloodpro/spiders
        scrapy crawl qiubai
        修改settings.py,将遵从robots协议去掉，将UA定制一下
    （6）认识response对象
        response.text : 字符串格式的内容
        response.body : 字节格式的内容
        response.url : 请求的url
        response.headers : 响应的头部
        response.status_code : 得到状态码
        在scrapy里面，已经为你集成了xpath，直接使用即可
        response.xpath('')
    （7）一键指定输出
        scrapy crawl qiubai -o qiubai.json
        scrapy crawl qiubai -o qiubai.xml
        scrapy crawl qiubai -o qiubai.csv
        解决输出csv有空行问题
        https://blog.csdn.net/qq_38282706/article/details/80279912