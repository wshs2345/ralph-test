# ralph admin url 文件源码详解

## 文件路径
  ralph/admin/sites.py
## 源码内容


        # 导入模块
      from collections import defaultdict
      from django.conf import settings
        # 来自于 django/c onf/__init__.py 文件
      from django.conf.urls import url
      from django.contrib.admin.sites import AdminSite
      
      
      class RalphAdminSiteMixin(object):
      
          """Ralph admin site mixin."""
          site_header = settings.ADMIN_SITE_HEADER
          site_title = settings.ADMIN_SITE_TITLE
      	# ralph的站点头部和标题，来自 ralph/settings/base.py 文件
      	# 虽然导入了django的settings 文件，但是优先使用 ralph 自己的 settings 文件内容
      	# 参考内容：
      	# ADMIN_SITE_HEADER = 'Ralph 3'
      	# ADMIN_SITE_TITLE = 'Ralph 3'
      
          index_template = 'admin/index.html'
      	# 该文件在 ralph/admin/templates/admin/index.html
      	
          app_index_template = 'admin/app_index.html'
      	# 该文件在 ralph/admin/templates/admin/app_index.html
      	
          object_history_template = 'admin/object_history.html'
      	# 该文件在 ralph/admin/templates/admin/object_history.html
      	
          def _get_views(self, admin):
      		# 定义不能导入的私有变量，一个参数 admin
              return (admin.change_views or []) + (admin.list_views or [])
      		# 返回一个拼接字符串，
      		# .change_views 方法和 .list_views 方法，来自于 ralph/admin/minx.py 文件里面的 RalphAdminMixin
      		# self.change_views = copy(kwargs.pop('change_views', [])) -- 将 kwargs 里面的 change_views 的值浅拷贝一份
      		# self.list_views = copy(self.list_views) or [] -- 因为 RalphAdminMixin 类的初始 list_views 是None， 故该值得到的值是None 或者 []
      
          def register(self, model_or_iterable, *args, **kwargs):
      		# 定义 register 函数，存在三个参数 model_or_iterable, *args, **kwargs
              super().register(model_or_iterable, *args, **kwargs)
      		# 继承父类的 register 函数
              # operate on admin class instance to get processed extra views
              for model in model_or_iterable:
      			# 当 model 处在一个迭代表中，
                  if model._meta.swapped:
      				# 符合条件就跳过去
                      continue
      			# 创建一个实例	
                  admin_instance = self._registry[model]
      			
                  for view in self._get_views(admin_instance):
                      view.post_register(self.name, model)
      				# .post_register 来自于 ralph/admin/views/extra.py 文件的 RalphExtraViewMixin 类方法
      				# 发送一个url 请求：  namespace:{app_label}_{model_name}_{url_name}
      
          def get_urls(self, *args, **kwargs):
      		# 继承 RalphAdminSiteMixin 的父类的 .get_urls() 方法
              urlpatterns = super(RalphAdminSiteMixin, self).get_urls(
                  *args, **kwargs
              )
      		#
      		# super 方法：
      		# class FooParent(object):  
      		#	...
      		#	def baba(self, message):
      		#	...
      		#
      		# class FooChild(FooParent):  
      		#	...
      		#	def erzi(self, message):
      		#		super(FooChild, self).baba(message)
      		#
      		
              for model, model_admin in self._registry.items():
      		# .items() 方法就是将 key值赋给 model， 将 value 值赋给 model_admin
                  for view in self._get_views(model_admin):
                      urlpatterns.insert(0, url(
                          view.get_url_pattern(model),
      					# .get_url_pattern(model) 方法就是：返回一个 {model._meta.app_label}/{model._meta.model_name}/{cls.url_name}/$ 格式的url
                          view.as_view(),
                          {
                              'model': model,
                              'views': getattr(
                                  model_admin, '{}_views'.format(view._type), []
                              ),
                          },
                          name=view.url_to_reverse
      					# name 的格式：cls.url_to_reverse = '{}_{}_{}'.format( model._meta.app_label, model._meta.model_name, cls.url_name )
                      ))
              return urlpatterns
      
          def get_extra_view_menu_items(self):
              """该方法返回 items 列表 for sitetree 机制."""
              items = defaultdict(list)
      		# 将列表形式的内容以 字典的方式展示出来。
      		# ps：s = [('xiaoming', 99), ('wu', 69)]
      		# d = defaultdict(list)
      		# 
              # for k, v in s:
              #  d[k].append(v)
              # 
              # d
              # defaultdict(<type 'list'>, {'lisi': [96], 'xiaoming': [99, 89]})
              # 
              # for k, v in d.items():
              #  print '%s: %s' % (k, v)
              # 
      		# wu: [69]
      		# xiaoming: [99]		
      		
      
              def get_item(model, view, change_view=False):
                  url = view.url_with_namespace
      			# .url_with_namespace 方法：cls.url_with_namespace = ':'.join([ cls.namespace, cls.url_to_reverse ])
                  if change_view:
      			# 因为 change_view=False，所以下句代码不执行
                      url += ' object.id'
                  return {'title': view.label, 'url': url}
      			# 返回一个字典 , title 是 url 中 model._meta.app_label 位置的值
              for model, model_admin in self._registry.items():
                  name = '{}_{}'.format(
                      model._meta.app_label, model._meta.model_name
                  )
      			# name的 格式 {model._meta.app_label}_{model._meta.model_name}
                  for view in model_admin.list_views or []:
                      items[name].append(get_item(model, view))
      				# 存在即添加
                  for view in model_admin.change_views or []:
                      items[name].append(get_item(model, view, True))
      				# 不存在，则创造条件添加
              return items
      
          def index(self, request, extra_context=None):
              from ralph.data_center.models import DataCenter
      		# 导入模块
              if extra_context is None:
                  extra_context = {}
      			# 如果 extra_context 值为None，则该块内容为空字典
      			
              extra_context['data_centers'] = DataCenter.objects.filter(
                  show_on_dashboard=True
              )
      		    # DataCenter 是 ralph/data_center/models/phycisal.py 文件中的类
      		    # 该句代码意思： 将 DataCenter 类中所有对象，show_on_dashboard 属性为 True 的都过滤出来。
      		
              return super().index(request, extra_context)
      		    # 重写父类的 index() 方法，调用当前类的 index() 方法，返回相应的值 
      
      class RalphAdminSite(RalphAdminSiteMixin, AdminSite):
      	  # 定义一个新类，继承父类 RalphAdminSiteMixin 和 AdminSite
          pass
      
      
      ralph_site = RalphAdminSite()
          # 调用 RalphAdminSite 类
