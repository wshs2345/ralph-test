# ralph首页 lincences 汉化展示问题
##目录

	1 当前的效果
		1.1 试验目的
	2 licences取值文件
		2.1 文件路径
		2.2 源代码
	3 查找 licences 源代码
		3.1 查找 licences.Licence 的 model._meta
	4 三个父类
		4.1 Regionalizable 内容分析
			4.1.1 PermissionsForObjectMixin 内容分析
			4.1.2 PermissionsBase 内容分析
			4.1.3 ModelBase 内容分析
		4.2 AdminAbsoluteUrlMixin 内容分析
		4.3 BaseObject 内容分析
			4.3.1 BaseObject的父类
			4.3.2 PolymorphicBase 内容分析
			4.3.3 PermissionsBase 内容分析
			4.3.4 TransitionWorkflowBase 内容分析
		4.4 结论
	5 解决方案
		5.1 查看模板文件
		5.2 解决办法
			5.2.1 {% trans %} 原理解析
			5.2.2 模板标签解决方法
		5.3 django 本地化文件编辑
			5.3.1 django.po 文件路径
			5.3.2 增加相关内容
			5.3.3 编译 django.py 语言文件
			5.3.4 ralph 加载菜单
			5.3.5 启动 ralph
		5.4 查看效果
	6 按照上述方法解决其他展示问题
		6.1 处理思路
		6.2 整体效果展示
	7 解决思路小结

## 1 当前的效果

![](http://i.imgur.com/JnTHu00.png)

### 1.1 试验目的：

将界面中展示的英文转换为中文

## 2 licences取值文件
### 2.1 文件路径

	ralph/admin/templatetags/dashboard_tags.py

### 2.2 源代码

	def ralph_summary():
	    models = [
			。。。
	        'licences.Licence',
			。。。
	    ]
	    results = []
	    for model_name in models:
	        app, model = model_name.split('.')
	        model = apps.get_model(app, model)
	        meta = model._meta
	        results.append({
	            'label': meta.verbose_name_plural,
				。。。
	            )
	        })
	    return {'results': results}


## 3 查找 licences 源代码

### 3.1 查找 licences.Licence 的 model._meta

文件：
	
	ralph/licences/models.py

源码：

	class Licence(Regionalizable, AdminAbsoluteUrlMixin, BaseObject):
		。。。

分析得知，该文件内没有 class Meta 内容， 那么则查看他所调用的父类中是否存在 Meta 类

## 4 三个父类

### 4.1 Regionalizable 内容分析
根据：

	from ralph.accounts.models import Regionalizable

所在文件：ralph/accounts/models.py

源码：

	class Regionalizable(PermissionsForObjectMixin):
	    region = models.ForeignKey(Region, blank=False, null=False)
	
	    class Meta:
	        abstract = True

分析得知：

该文件内存在 class Meta 内容，但是是继承的

#### 4.1.1 PermissionsForObjectMixin 内容分析
根据：

	from ralph.lib.permissions import ( PermByFieldMixin, PermissionsForObjectMixin, user_permission )

文件：

	ralph/lib/permissions/models.py

源码：

	class PermissionsForObjectMixin(models.Model, metaclass=PermissionsBase):
		。。。
	
	    class Meta:
	        abstract = True

根据源码得知，该爷类依然继承了他父类 PermissionsBase 的内容 

#### 4.1.2 PermissionsBase 内容分析
	
	class PermissionsBase(ModelBase):
		。。。

该类也没有 Meta 内容，查看他的父类 ModelBase 内容



##### 4.1.3 ModelBase 内容分析
根据：

	from django.db.models.base import ModelBase

文件:

	django/db/models/base.py

源码：




### 4.2 AdminAbsoluteUrlMixin 内容分析
根据： 

	from ralph.lib.mixins.models import AdminAbsoluteUrlMixin

所在文件：

	ralph/lib/mixins/models.py

源码：

	class AdminAbsoluteUrlMixin(object):
	    。。。

分析得知，该父类文件内没有 class Meta 内容

### 4.3 BaseObject 内容分析
根据：

	from ralph.assets.models.base import BaseObject

所在文件：

	ralph/assets/models/base.py

源码：

	BaseObjectMeta = type(
	    'BaseObjectMeta', (
	        PolymorphicBase,
	        PermissionsBase,
	        TransitionWorkflowBase
	    ), {}
	)

	。。。
	
	class BaseObject(
	    Polymorphic,
	    TaggableMixin,
	    PermByFieldMixin,
	    TimeStampMixin,
	    WithCustomFieldsMixin,
	    models.Model,
	    metaclass=BaseObjectMeta
	):
	。。。

#### 4.3.1 BaseObject的父类

根据 代码提示： metaclass=BaseObjectMeta， 我们查看 BaseObjectMeta 的类，发现该类还有父类，则继续查看。

#### 4.3.2 PolymorphicBase 内容分析
代码：

	from ralph.lib.polymorphic.models import (Polymorphic, PolymorphicBase, PolymorphicQuerySet )

文件：

	ralph/lib/polymorphic/models.py

源码：

	class Polymorphic(models.Model):
		。。。	
	    class Meta:
	        abstract = True

查看源码得知，该文件存在 class Meta，内容是 abstract = True


#### 4.3.3 PermissionsBase 内容分析
代码：

	from ralph.lib.polymorphic.models import (Polymorphic, PolymorphicBase, PolymorphicQuerySet )

文件：

	ralph/lib/polymorphic/models.py

源码：


	class PolymorphicBase(models.base.ModelBase):
		。。。

#### 4.3.4 TransitionWorkflowBase 内容分析
代码：

	from ralph.lib.transitions.models import TransitionWorkflowBase

文件：

	ralph/lib/transitions/models.py

源码：
	
	class TransitionWorkflowBase(ModelBase):
	    。。。

经查看，该源码内 没有 class Meta 内容

### 4.4 结论

经过翻看所有涉及到的父类，甚至 ModelBase 都没有发现 .verbose_name_plural 方法，那么我们就用手工的方法来实现该功能：

## 5 解决方案

### 5.1 查看模板文件
文件：

	ralph/admin/templates/ralph_summary.html

代码：

	。。。
	   <div class="name">{{ result.label }}</div>
	。。。

### 5.2 解决办法

使用 {% trans %} 标签方法

#### 5.2.1 {% trans %} 原理解析

如果启用 i18n 的国际化格式扩展，则可以将 模板中的部分标记为可译的。 而实现可译的功能就需要使用 trans。

{% trans %} 模板标签内可以用来翻译一个变量字符串，也就是说，该变量字符串只是代表若干个字母，而不是变量代表的"值".

#### 5.2.2 模板标签解决方法
将 {{ result.label }} 变量标签，改为模板标签表达式：{% trans result.label %}

代码如下：

	。。。
	   <div class="name">{% trans result.label %}</div>
	。。。

### 5.3 django 本地化文件编辑
#### 5.3.1 django.po 文件路径

	ralph/locale/zh_Hans/LC_MESSAGES/django.po

#### 5.3.2 增加相关内容
	
	...
	#：
	msgid "lincences"
	msstr "许可证"
	...
#### 5.3.3 编译 django.py 语言文件

	django-admin.py compilemessages

#### 5.3.4 ralph 加载菜单
	
	Ctrl + c 终止当前ralph
	ralph sitetree_resync_apps

#### 5.3.5 启动 ralph
	
	ralph runserver 0.0.0.0:8000

### 5.4 查看效果

![](http://i.imgur.com/mCaIstL.png)

## 6 按照上述方法解决其他展示问题
### 6.1 处理思路
supports 和 domains 的解决方法与 lincences 方法一致，Back Office Assets 的解决方法同 5.3 一节

### 6.2 整体效果展示

![](http://i.imgur.com/XHruqrS.png)

## 7 解决思路小结

根据解决该问题的思路来看，主要包括几个方面

* 0、 从网页源码方向来分析问题
* 1、 理解 class Meta的原理
* 2、 了解模板标签 {{ }} 和 {% trans %} 的原理和使用
* 3、 django本地化文件的使用
* 4、 测试的时候，要清理浏览器缓存。
