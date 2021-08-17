## 框架介绍

[Vant](https://vant-contrib.gitee.io/vant/#/zh-CN/)是有赞前端团队开源的移动端组件库，于 2017 年开源，是业界主流的移动端组件库之一。

本项目基于Vue2，Vant开发，为实现管理后台的功能，还整合了如下第三方组件

依赖方式|组件|功能
---|---|---
dependencies|axios|与后端通讯
dependencies|moment|日期、时间解析
dependencies|vue-router|路由管理
dependencies|vuex|状态管理
devDependencies|eslint|编辑器语法检测
devDependencies|babel|按需加载

## 安装
从码云下载脚手架代码
```bash
git clone https://gitee.com/django-extend/vant-django.git my-project
cd my-project
cnpm install
npm run serve
```
## 目录结构
> 整个项目的目录结构

```bash
.
├── README.md
├── babel.config.js     # 按需加载配置
├── package.json
├── public
│   ├── favicon.ico     # LOGO
│   └── index.html      # Vue 入口模板
├── src
│   ├── App.vue         # Vue 模板入口
│   ├── api             # ajax
│   ├── assets          # 本地静态资源
│   ├── components      # 业务通用组件
│   ├── layouts         # 布局
│   ├── main.js         # Vue 入口 JS
│   ├── permission.js   # 路由守卫(路由权限控制)
│   ├── router          # 路由管理
│   ├── store           # 状态管理
│   ├── utils           # 工具库
│   └── views           # 业务页面入口
└── vue.config.js
```
> Django目录结构 (src/components/Django)

```bash
.
├── api
│   └── resource.js                     # 后端ajax请求封装
├── fields
│   ├── DjangoChangePasswordField.vue   # 独特的修改密码字段
│   ├── DjangoChoiceField.vue           # 下拉选择组件
│   ├── DjangoDatetimeField.vue         # models.DatetimeField
│   ├── DjangoField.vue                 # models.Field （所有字段）
│   ├── DjangoFileField.vue             # models.FileField
│   ├── DjangoImageField.vue            # models.ImageField
│   ├── DjangoManyToManyField.vue       # models.ManyToManyField
│   ├── DjangoValue.vue                 # 值插槽，列表页和查看页使用
│   └── ForeignSelect.vue               # models.ForeignKey
├── form
│   ├── AddForm.vue                     # 新增页
│   ├── EditForm.vue                    # 编辑页
│   └── ViewForm.vue                    # 查看页
├── index.vue                           # 列表页
├── models                              # 数据模型
│   └── meta.js                         # metainfo本地缓存
└── utils
    └── submit.js                       # 提交统一封装(新增/编辑)
```
## 鉴权
```javascript
// src/utils/request.js 行 35
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
import RouteView from '@/layouts/RouteView'

const components = {
  defaults: {
    'menu': RouteView,
    'dashboard': () => import('@/views/Dashboard'),
    'django': () => import('@/components/Django'),
    '_edit': () => import('@/components/Django/form/EditForm'),
    '_add': () => import('@/components/Django/form/AddForm'),
    '_view': () => import('@/components/Django/form/ViewForm')
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
3.收到指令，调用步骤4函数生成路由| src/store/modules/permission.js
4.动态生成菜单|src/store/modules/permission.js
5.将路由写入本地存储|src/store/modules/permission.js
6.将动态路由写入vue-router|src/permission.js


前端自定义菜单入口在`src/router/index.js`
```javascript
export const asyncRoutes = [
]
```
可以在这里加入菜单，菜单的格式如下
```javascript
  {
    path: '/demoIndex',
    name: 'demoIndex',
    component: () => import('@/layouts/RouteView'),
    children: [
      {
        path: '/demo',
        name: 'demo',
        meta: { title: '演示' },
        component: () => import('@/views/Demo')
      }
    ]
  }
```

## vant扩展组件 vanx-datetime-picker
时间选择器，时间格式支持到秒（原生只支持到分钟）

- 如何引用

```javascript
import VanxDatetimePicker from '@/components/vantx/DatetimePicker'
```

- 参数说明

参数|说明|类型|默认值
---|---|---|---
value/v-model|值|string|null
type|时间类型，可选值为`datetime`, `date`, `time`|string|datetime
show-toolbar|是否显示顶部栏|boolean|true

## vant扩展控件 vantx-list
列表组件，vantx-list是对原生van-list的二次封装，集成了如下功能

1. 下拉刷新下一页
2. 上拉刷新最新数据
3. 右滑删除
4. 多选

由于集成了服务端分页，对服务端返回结果做了如下数据结构约定

```json
{
    "pageSize": 20,         // 一页显示多少记录
    "pageNo": 1,            // 当前第几页
    "totalPage": 5,         // 总页数
    "totalCount": 100,      // 总记录数
    "data": []              // 当页记录集
}
```

- 参数说明

参数|说明|类型|默认值
---|---|---|---
data|数据源|function|-
columns|列信息|array|-
row-key|主键名|string|-
selectable|是否多选|boolean|false
deletable|是否可删除|boolean|false

- 事件

事件名|说明|回调参数
---|---|---
click|记录点击|row
select|记录选择|row
delete|记录删除|keys