# ralph 资产管理系统汉化方案
## 目录
	0.准备工作
	1、修改settings.py文件
		1.1 设置ralph支持本地格式化
		1.2 设置ralph支持语言国际化
		1.3 启动本地化语言的中间件
		1.4 设置本地化语言文件默认格式
		1.5 设置本地化语言文件默认路径
	
	2、编辑语言文件django.po
		2.1 创建 ralph 的语言文件依赖目录
		2.2 创建 ralph 的语言文件 django.po
		2.3 简介 ralph 的语言文件 django.po 
		2.4 编辑语言文件 django.po
		2.5 将 django.po 语言文件编译成本地化识别的二进制文件  django.mo
	3、本地化语言生效
		3.1 关闭当前运行的 ralph
		3.2 重新加载菜单内容
		3.3	启动 ralph
		3.4 测试
## 需求
下面是ralph的原始界面，我们需要将其调整为中文界面
![](http://i.imgur.com/C2u92kE.png)

## 概述
本文的主要内容是 ralph 资产管理系统的汉化工作，该文涉及到的技术部分包括三部分：

* 1、django 项目本地化
* 2、ralph 菜单加载操作
* 3、ralph 启动基本操作


## 0.准备工作
进入ralph的应用程序目录

	cd /opt/ralph/ralph-core/lib/python3.4/site-packages/ralph

## 1、修改ralph的全局配置文件
	vim settings/base.py
### 1.1 设置ralph支持本地格式化
		USE_L10N = True
### 1.2 设置ralph支持语言国际化
		USE_I18N = True
### 1.3 启动本地化语言的中间件
		MIDDLEWARE_CLASSES = (
			...
			'django.contrib.sessions.middleware.SessionMiddleware',
			'django.middleware.locale.LocaleMiddleware',	# 该语句必须 在SessionMiddleware后面
			...
		)
### 1.4 设置本地化语言文件默认格式
		LANGUAGE_CODE = 'zh-hans'
### 1.5 设置本地化语言文件默认路径
		LOCALE_PATHS = (os.path.join(BASE_DIR, 'locale'), )

## 2、编辑语言文件django.po
### 2.1 创建 ralph 的语言文件依赖目录
		mkdir locale
### 2.2 创建 ralph 的语言文件 django.po
		django-admin.py makemessages -l zh_Hans
### 2.3 简介 ralph 的语言文件 django.po 
		#: dashboards/admin.py:75  # 需要翻译文字的位置
		msgid "Link"			   # 需要翻译的文字
		msgstr ""				   # 翻译后的文字

### 2.4 编辑语言文件 django.po
		vim locale/zh_Hans/LC_MESSAGES/django.po
		根据 2.3 提示信息，将所有关键字翻译为本地化文字
### 2.5 将 django.po 语言文件编译成本地化识别的二进制文件  django.mo
		django-admin.py  compilemessages
## 3、本地化语言生效
### 3.1 关闭当前运行的 ralph
		进入 ralph 执行命令窗口，输入 Ctrl + c，退出当前 ralph 软件
### 3.2 ralph 重新加载菜单内容
		ralph sitetree_resync_apps
		
			Sitetrees found in `ralph.admin` app ...
			Processing `ralph_admin` tree ...
			Adding `数据中心` tree item ...
			Adding `硬件` tree item ...
			Adding `Add data center asset` tree item ...
			Adding `{{ original }}` tree item ...
			。。。

### 3.3	启动 ralph
		ralph runserver 0.0.0.0:8000
### 3.4 测试
		登录搜狗浏览器，或者火狐浏览器查看 ralph界面效果
![](http://i.imgur.com/KjwKGfm.png)
