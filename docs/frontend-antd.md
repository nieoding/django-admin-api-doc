## 框架介绍

[Ant Design](https://ant.design/)是蚂蚁（也就是阿里啦）出品的组件库，主要用于研发企业级中后台产品，antd致力于最基层的标准组件库构建，本身并不关注业务逻辑。

[Ant Design Pro](https://beta-pro.ant.design/)是基于 Ant Design 和 umi 的封装的一整套企业级中后台前端/设计解决方案，也就是antd的一套开箱即用后台。

[Antd Design Pro Vue](https://pro.antdv.com/)是Pro的Vue版本实现，说是说社区实现，但相信也是官方出品的，毕竟vue在中国是最受欢迎的前端框架，我们这里目前采用的基础框架就是它。

阿里出品，必属精品，Antd本身还是一套很有生命力的框架，是我们的项目选型首选！

## 安装
从码云下载脚手架代码
```bash
git clone https://gitee.com/django-extend/antd-django.git my-project
cd my-project
cnpm install
npm run serve
```
## 入门
> [官方文档传送门](https://pro.antdv.com/docs/getting-started)

官方文档已经足够详细了，可以直接阅读，下面列出的是在官方框架上做的修改部分

## 目录结构

> Django部分的目录结构 （src/components/Django）

```bash
.
├── api                           # 后端ajax请求封装
│   └── resource.js
├── fields                        # 字段列表
│   ├── DjangoField.vue           # models.Field （所有字段）
│   ├── DjangoFileField.vue       # models.FileField
│   ├── DjangoImageField.vue      # models.ImageField
│   ├── DjangoManyToManyField.vue # models.ManyToManyField
│   └── ForeignSelect.vue         # models.ForeignKey
├── form                          # 表单列表
│   ├── ChangePasswordForm.vue    # 修改密码
│   ├── EditForm.vue              # 新增页/编辑页
│   └── ViewForm.vue              # 查看页
├── index.vue                     # 列表页
├── models                        # 数据模型
│   └── meta.js                   # metainfo本地缓存
└── slots                         # 插槽列表
    ├── DjangoValue.vue           # 值插槽，列表页和查看页使用
    └── ImagePreview.vue          # 图片预览（暂保留，无用)
```
## 鉴权
```javascript
// src/utils/request.js 行 50
config.headers['Authorization'] = 'Bearer ' + token
```
与后端约定在请求头里面加入 ```Authorization: Bearer **** ``` 做为鉴权信息

## 动态菜单与路由

请求后端接口`/api/auth/userinfo/`，会将用户的菜单信息回传给前端
```json
{
    "menus": [
        {
            "name": "dashboard",
            "key": "dashboard",
            "component": "dashboard",
            "meta": {
                "icon": "dashboard",
                "title": "站点管理"
            }
        },
        {
            "name": "auth",
            "key": "auth",
            "component": "menu",
            "meta": {
                "icon": "table",
                "title": "认证和授权"
            },
            "children": [
                {
                    "name": "auth.group",
                    "key": "group",
                    "component": "django",
                    "meta": {
                        "title": "组",
                        "permission": "auth-group"
                    }
                },
                {
                    "name": "auth.user",
                    "key": "user",
                    "component": "django",
                    "meta": {
                        "title": "用户",
                        "permission": "auth-user"
                    }
                }
            ]
        }
    ]
}
```
其中的component比较重要，前端需要根据这个字符串去动态加载组件，组件库定义如下，可自行扩展
```javascript
// src/utils/components.js
import { RouteView } from '@/layouts'

const components = {
  defaults: {
    'menu': RouteView,
    'dashboard': () => import('@/views/Dashboard'),
    'django': () => import('@/components/Django')
  },
  custom: {

  },
  get (key) {
    return this.custom[key] || this.defaults[key]
  }
}
export default components

```
动态路由实现比较复杂，在源码中做了注释，可以搜索`动态菜单步骤`来阅读整个链路源码了解原理

步骤|代码
---|---
1.从后端读取菜单列表| src/permission.js
2.向vuex发起指令，通知生成菜单| src/permission.js
3.收到指令，调用步骤4函数生成路由| src/store/modules/async-router.js
4.动态生成菜单|src/router/generator-routers.js
5.将路由写入本地存储|rc/store/modules/async-router.js
6.将动态路由写入vue-router|src/permission.js


前端自定义菜单入口在`src/config/router.config.js`
```javascript
export const asyncRouterMap = [
]
```
可以在这里加入菜单，菜单的格式可以参考`constantRouterMap`的写法，也可以看[官方文档](https://pro.antdv.com/docs/router-and-nav)