## 前言
  Django自带的Admin模块是一个Bug级的存在，几行代码就能快速的生成一套完善的管理后台，但是一直以来这套后台的界面的美观度被人诟病，所以出来了很多美化方案.

  我记得最早的是国人写的[XAdmin](https://github.com/sshwsfc/xadmin)，当时出来确实眼前一亮，但是后面作者也不再更新了，而且需要在app下新建一个xadmin.py文件，开发者维护起来麻烦，建议弃坑；
 
  后来老外写了一个[Suit](https://djangosuit.com/)，这是我印象中第一个代码0侵入性的美化方案，只需要在settings.py里面加一行代码就把后台界面给改了，后面所有的其他方案都沿袭这种思路了，但是这哥们美化后的界面确实也不好看，还搞收费了，建议弃坑；
 
  最近国内出来一个[SimpleUI](https://simpleui.72wo.com/docs/simpleui)很火，试用了一下，整体界面是现代风格，符合国人使用习惯，一直在更新，推荐使用。

## 项目介绍
  django-admin-api设计初衷是有2个，一个也是美化原界面，但还有一个就是实现真正的前后端分离，前端可以自己选择自己熟悉的前端框架。实现的基本技术原理是将admin的基础功能都通过[DjangoRestframework](django-rest-framework.org)输出成api接口，前端遵循协议完成整合，最后生产环境前后端部署是分开部署的。
 
  如果你是后端开发工程师，那么可以把重点放在前端部分，学习如何撸前端代码，脱离现在日益严重的只会写CRUD现状；如果你是前端工程师，也不要畏惧，可以把重点放在后端部分，你会发现其实写后端也是很简单的，总而言之，全栈才是王道。
 
  django-admin-api面向的诉求应该是希望能快速生产现代化风格的管理后台，同时又希望这个后台能高度定制化。如果只是想简单的美化，就使用simpleUI吧。

## 安装
- 后端
  
> Python 要求版本 > 3.0

> Django 要求版本 > 2.2.5

使用`pip`安装...
```bash
pip install django-admin-api
```

- 前端

> NodeJS 要求版本 >= 8.9

开发环境安装cnpm，这样安装依赖包会加速（生产环境不需要，直接使用yarn安装就好了）
```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
## 开始
- 后端
  
创建一个简单项目
```bash
django-admin startproject example
cd example
./manage.py migrate
./manage.py createsuperuser
```

`INSTALLED_APPS` 加入如下依赖.
```python
# example/settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',
    'django_filters',
    'admin_api',
]
```
输出API接口
```python
# example/urls.py
...
import admin_api
urlpatterns = [
  ...
  path('api/', admin_api.site.urls),
]
```
启动后端项目
```bash
./manage.py runserver
```

- 前端

克隆前端项目(以antd为例)
```bash
git clone https://gitee.com/django-extend/antd-django.git my-project
```

安装项目依赖
```bash
cnpm install
cnpm install webpack@4.1.0  # 降级webpack版本，否则项目会跑不出来，报这个错误"TypeError: Cannot read property 'get' of undefined"
```

启动前端项目

```bash
npm run serve -- --port 5000 # 后端占用8000端口，前端指定5000端口
```

访问新后台地址[http://localhost:5000](http://localhost:5000)