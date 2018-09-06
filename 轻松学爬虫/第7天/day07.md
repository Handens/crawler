# day07

做法：将chromedriver.exe放入到谷歌浏览器安装路径下
   	 c:/app   /chrome/app/bin
   	 意思就是讲chromedriver.exe和chrome.exe放到同一个路径
   	 然后将chrome.exe所在的路径添加到环境变量中
 	   如果还是不行，降级selenium

```
pip uninstall selenium
pip install selenium=2.48
```

 

多任务同时执行：抽烟打游戏，唱歌跳舞，开车手脚并用

电脑中：vscode、录屏工作、vnc、浏览器等



### 1、多任务

```
import time

def sing():
	for x in range(1, 6):
		print('我在唱歌')
		time.sleep(1)

def dance():
	for x in range(1, 6):
		print('我在跳舞')
		time.sleep(1)

def main():
	sing()
	dance()
	
if __name__ == '__mian__'
	main()
```

案例

```
from threading import Thread, Lock
from queue import Queue
import requests
from bs4 import BeautifulSoup
import json

class CrawlThread(Thread):
    def __init__(self, name, page_queue, data_queue):
        super().__init__()
        self.name = name
        self.page_queue = page_queue
        self.data_queue = data_queue
        self.url = 'http://www.fanjian.net/duanzi-{}'
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.84 Safari/537.36',
        }

    def run(self):
        print('线程--%s--启动成功----' % self.name)
        '''
        1、从页码队列取出一个页码
        2、拼接生成待发送的url
        3、发送请求，得到响应
        4、将响应添加到数据队列中
        '''
        while 1:
            page = self.page_queue.get()
            url = self.url.format(page)
            content = requests.get(url=url, headers=self.headers).text
            self.data_queue.put(content)
        print('线程-%s-运行结束--' % self.name)

class ParseThread(Thread):
    def __init__(self, name, data_queue, fp, lock):
        super().__init__()
        self.name = name
        self.data_queue = data_queue
        self.fp = fp
        self.lock = lock

    def run(self):
        print('线程--%s--启动成功----' % self.name)
        '''
        1、从数据队列中取出一个数据
        2、解析和处理数据
        '''
        while 1:
            content = self.data_queue.get()
            self.parse_content(content)
        print('线程-%s-运行结束--' % self.name)
    
    def parse_content(self, content):
        # 生成soup对象
        soup = BeautifulSoup(content, 'lxml')
        # 开始解析
        item = {}
        string = json.dumps(item, ensure_ascii=False)
        self.lock.acquire()
        self.fp.write(string + '\n')
        self.lock.release()

def create_queue():
    page_queue = Queue()
    data_queue = Queue()
    for page in range(1, 6):
        page_queue.put(page)
    return page_queue, data_queue

def main():
    # 搞一个列表，用来存放所有的线程对象
    t_list = []
    # 打开文件
    fp = open('jian.txt', 'w', encoding='utf8')
    # 创建锁
    lock = Lock()
    # 在这创建队列
    page_queue, data_queue = create_queue()
    # 创建采集线程
    crawl_name_list = ['采集线程1', '采集线程2', '采集线程3']
    for crawl_name in crawl_name_list:
        t_crawl = CrawlThread(crawl_name, page_queue, data_queue)
        t_crawl.start()
        # 将线程保存到列表中
        t_list.append(t_crawl)
    # 创建解析线程
    parse_name_list = ['解析线程1', '解析线程2', '解析线程3']
    for parse_name in parse_name_list:
        t_parse = ParseThread(parse_name, data_queue, fp, lock)
        t_parse.start()
        t_list.append(t_parse)
    
    # 主线程要等待所有的子线程结束我才能结束
    for t_tmp in t_list:
        t_tmp.join()
    
    # 关闭文件
    fp.close()
    print('主线程、子线程全部结束')

if __name__ == '__main__':
    main()

'''
1、文件解析，每一个段子解析为字典
2、采集线程和解析线程都是死循环，何时退出线程呢？
'''
```





这种实现不了，同时进行。一个函数一个任务。



所以，为了实现多任务：引入**多进程**和**多线程**

同步、异步、并行、并发

​	同步：先执行任务a，再执行任务b，称任务a和b是同步的

​	异步：任务a和b同时执行，称任务a和 b是异步的

​	实现异步的时候：还有不同，一种是真正的异步，一种是伪异步

​	并行：真正的异步

​	并发：伪异步，通过计算机的快速切换，达到同时运行的假象



##### 1，进程



​	进程：软件启动之后就是一个进程，liunx系统中中kill -9 进程id

​	代码中：没有运行之前，称之为程序，程序运行起来就是一个进程，如果只有一个进程，称之为主进程。如果通过这个进程创建了其他进程，称之为子进程

​	进程创建（process）：

​	（1）面向过程

```
p = Process(target=xxx,args=(xxx,))
target :进程启动之后要执行的函数
args :主进程给进程传递的参数
p.start() :启动进程
p.join()：主进程等待子进程的结束
os.getpid():获取进程id
os.getppid():获取父进程id
```

创建进程

```
from multiprocessing import Process
import time
import os

def sing(name):
    print('sing接受过来的明星为%s' % name)
    print('sing的id号为%s' % os.getpid())
    print('sing的父进程id为%s' % os.getppid())
    for x in range(1, 6):
        print('我在唱青藏高原')
        time.sleep(1)

def dance():
    print('dance的id号为%s' % os.getpid())
    print('dance的父进程id为%s' % os.getppid())
    for x in range(1, 6):
        print('我在跳拉丁舞')
        time.sleep(1)

# 一个主进程，用来负责创建子进程的，然后一个唱歌子进程，一个跳舞子进程
def main():
    # 主进程可以给子进程传递参数
    name = '高圆圆'
    print('主进程的id号为%s' % os.getpid())
    # 创建进程
    p_sing = Process(target=sing, args=(name,))
    p_dance = Process(target=dance)

    # 启动进程
    p_sing.start()
    p_dance.start()

    # 需要让主进程等待子进程结束之后再结束
    p_sing.join()
    p_dance.join()

    print('主进程、子进程全部运行结束')

if __name__ == '__main__':
    main()
```

##### 	（2）面向对象

```
from multiprocessing import Process
import time

class SingProcess(Process):
	#
	def __init__(self, name):
		super().__init__()
		self.name = name
	#进程启动之后，默认执行这个函数
	def run(self):
		for x in range(1, 6):
			print('我在唱歌')
			time.sleep(1)
	
class DancePricess(process):
	def run(self):
		for x in range(1, 6):
			print('我在跳舞')
			time.sleep(1)

def main():
	name = 'pythin'
	ps = SingProcess(name)
	pd = DanceProcess()
	ps.start()
	ps.start()
	ps.join()
	ps.join()
	print('主进程子进程全部结束')
	
if __name__ == '__main__'
	main()
```



```
class MyProcess(Process):
	def run(self):
		pass
```



##### 3，是否共享局部变量

不共享局部变量

```
from multiprocessing import Process

def demo():
	name = '朱茵'
	if lala = 'change':
		name = '张敏'
		print('change修改后的值为%s' % name)
	else:
		print('read进来读取那么的值为%s' % name)
def main():
	p1 = Process(target=demo, args=('change',))
	p2 = Process(target=demo, args=('read',))
	
	p1.start()
	p2.start()
	p1.join()
	p2.join()
	print('全部结束')
	
if __name__ == '__mian__':
	main()
```

##### 4，是否贡献全部变量

不贡献全部变量

```
from multiprocessing import Process

name = '宋小宝'

def change():
	global name
	name = '刘德华'
	print('change进程修改后的值为%s' % name)
	
def read():
	time.sleep(3):
	print('read进程的值为%s' % name)

def mian():
	p1 = Process(target = change)
	p2 = Process(target = read)
	
	p1.start()
	p2.start()
	p1.join()
	p2.join()
	print('全部结束')
	
if __name__ == '__mian__':
	main()
```



##### （5）进程池

​	进程是不是创建的越多越好呢？

​	以拷贝文件为例，文件夹里面有100个文件

​	一个进程，没有充分利用电脑的性能，也不行

​	最后测试结果为5个进程，100个文件全部拷贝成功

​	意思：最多就创建5个进程，然后你给他100个任务，5个进程将100个任务全部执行完毕

```
	po =Pool(5) # 创建进程池
	po.apply_async(demo, args=(xxx,)) # 进程池添加任务并且传递参数
	po.close() # 关闭进程池，不再向里面添加任务
	po.join() # 让主进程等待进程池中所有进程结束之后再结束

```



```
from multiprocessing import Pool
import name
import os

def demo(name):
	print('任务的名称为%s，当前进程的id号为%s' % (name,os.getpid()))
	time.sleep(2)
	
def mian():
	# 创建一个进程池对象
	po = Pool(5)
	# 给进程池扔任务
	lt = ['关羽', '张飞', '赵云', '马超', '黄忠', '张辽']
	
	for name in lt:
		po.apply_async(demo, args=(name, ))
	
	# 关闭进程池，不能再向进程池添加任务了
	po.close()
	# 让主进程等待进程池中进程全部结束之后再结束
	po.join()
	print('主进程、子进程全部结束')

if __name__ == '__mian__':
	main()
```



### 2，线程

 线程：一个qq，可以语音，可以视频，一个暴风影音，播放音频，播放视频，一个word文档，可以编辑，可以实时检查和保存等
    一个进程里面如果只有一个线程，称之为主线程，如果还有其它线程，称之为子线程
    cpu切换的基本单位是线程
    多进程：切换进程消耗资源大，但是稳定，如果一个子进程挂了，不影响其它进程
    多线程：切换线程消耗资源小，但是不稳定，如果一个线程挂了，整个进程就挂了

​    线程（thread）

##### （1）面相过程

t = threading.Thread(target=xxx, name=xxx, args=(xxx, ))
        target: 线程启动执行的函数
        name: 线程的名字
        args: 给子线程传递参数
        threading.current_thread().name  获取线程名字
        t.start() : 启动线程
        t.join() : 让主线程等待



```
import threading 
import time

def sing():
	print('sing线程的名字叫%s' % threading.current_thread().name)
	for x in range(1, 6):
		print('我喜欢唱走进新事件')
		time.sleep(1)
		
def dance():
	print('dance线程的名字叫%s' % threading.current_thread().name)
	for x in range(1, 6):
		print('我喜欢跳钢管舞')
		time.sleep(1)

# 一个主线程、两个子线程
def mian():
	s_sing = threading.Thread(target=sing，name='唱歌')
	s_dance = threading.Thread(target=dance, name='跳舞')
	
	# 启动线程
	t_sing.start()
	t_dance.start()
	
	# 让主线程等待子线程结束只有结束
	t_sing.join()
	t_dance.jion()
	
	# 结束进程
	print('全部结束')
	
if __name == '__mian__'
	main()
```



##### （2）面相对象

```
class MyThread(threading.Thread):
def run(self):
	pass
```


​      如果传递参数，需要重写构造方法，注意手动调用父类构造方法
    线程之间能否共享局部变量
        不共享
    线程之间是否共享全局变量
        共享
    线程安全
        使用线程锁来解决，谁先抢到谁先用

```
from threading import Lock
lock = Lock()
lock.acquire()
lock.release()
```

​        

```
import threading 

class SingThread(threading.Thread):
	def __init__ = ()
	
	def sing():
		print('sing线程的名字叫%s' % threading.current_thread().name)
		for x in range(1, 6):
			print('我喜欢唱走进新事件')
			time.sleep(1)
```



```
class MyThread(threading.Thead):
	def run(self):
		pass
```

如果传递参数，需要重写构造方法，注意手动调用父类构造方法



##### 线程安全

​	使用线程锁来解决，谁先抢到谁先用

```
from threading import Lock
lock = Lock()
lock.acquire()
lock.release()
```



```
import threading
from threading import Lock

count = 100

# 通过加锁机制解决线程安全问题
# 枷锁机制是建立在降低性能基础上的
lock = Lock

def test(n):
	gloal count
	for x in range(1, 100):
		lock.acquire()
		count += n
		count -= n
		lock.release()
	print('运行结束之后count的值为%s' % count)
	
def main():
	t1 = threading.Thread(target=test, args=(3, ))
	t2 = threading.Thread(target=test, args=(5, ))
	t1.start()
	t2.start()
	t1.join()
	t2.join()
	print('主线程读取count的值为%s' % count)
	print('全部结束')
	
if __name__ == '__main__':
	main()
```



##### 队列：

队列：先进先出

栈：先进后出

队列：买火车票，先进先出
        栈：先进后出  函数的运行
        使用队列？

```
from queue import Queue
q = Queue(5)
```


​        添加元素

```
q.put(xxx)
q.put(xxx, False)  如果队列满，立即抛出异常
q.put(xxx, True, 5) 如果队列满，5s之后抛出异常
```

​        

获取元素

```
q.get()
q.get(False)  如果队列为空，立即抛出异常
q.get(True, 5) 如果队列为空，5s之后抛出异常
```

​        

```
q.full()  队列是否满
q.empty() 队列是否空
q.qsize() 队列的元素的个数
```



线程加队列---生产者消费者模型

```
while 1:
     生产数据
     消费数据

     生产数据线程
         队列
     消费数据线程
```





```
from queue import Queue

# 先创建一个队列，可以指定队列的长度，如果不写长度，默认无限长
q = Queue(5)

# 向队列中添加元素
q.put('库里')
q.put('乔丹')
q.put('韦德')
q.put('恩比德')
q.put('易建联')
# q.put('不晓得了'，False)
q.put('不晓得了'，Ture, 5)

print(q.get())
print(q.get())
print(q.get())
print(q.get())
print(q.get())
# print(q.get(False))
# print(q.get(Ture,5))
```



3，多线程爬虫

while 1:
        根据url发送请求，得到响应（生产者）
        解析响应（消费者）
    生产者线程  3
        队列
    消费者线程  3