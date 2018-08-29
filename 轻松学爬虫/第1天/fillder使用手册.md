1、配置抓取https包
	tools==》teletick options==》https
	选中capture https connects\decrypt https traffic\ingnore xxxx
	点击actions==》trust root certificate
	配置完毕，重启fiddler让其生效

	如果有问题，参考这个博客
	https://blog.csdn.net/d1240673769/article/details/74298429
2、清楚所有请求
	叉号==》remove all
3、暂停和启动抓包
	file==>capture traffic
4、fiddler介绍
	左边栏
		显示所有的请求
		<> : 代表的是html请求
	右边栏
		点击某个请求，查看这个请求的详细信息
		选中inspectors
		右上：请求信息
			raw：有关所有请求的纯文本内容
			webforms：请求参数
				上面：query_string   get参数
				下面：body   post参数
		右下：响应信息
			raw：所有响应的纯文本内容
			json：如果响应为json格式数据，在这里查看
	左下角黑色窗口输入指令
		cls：清除掉所有请求
		select html: 选择html请求
		select js: 选择js请求
		?baidu: 选择带baidu的请求