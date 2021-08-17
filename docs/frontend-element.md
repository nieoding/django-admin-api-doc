## 框架介绍

[Element-UI](https://element.eleme.cn/)是饿了么前端团队开源组件库，在国内受众比较多，贴合国人使用习惯

[Element-Admin](https://panjiachen.gitee.io/vue-element-admin-site/zh/) 是一个后端前端解决方案，基于Vue和Element-UI实现，内置了后台基本功能模块（多语言、路由、权限等），本项目是基于Elemnt-Admin上做的二次开发

## 安装
从码云下载脚手架代码
```bash
git clone https://gitee.com/django-extend/element-django.git my-project
cd my-project
cnpm install
npm run dev
```
## 入门
> [Element 官方文档传送门](https://element.eleme.cn/#/zh-CN/component/installation)
> 
> [Element-Admin 官方文档传送门](https://panjiachen.gitee.io/vue-element-admin-site/zh/guide/)

官方文档已经足够详细了，可以直接阅读，下面列出的是在官方框架上做的修改部分

## 目录结构

> Django部分的目录结构 （src/components/Django）

```bash
.
├── api                                 # 后端ajax请求封装
│   └── resource.js
├── components                          # element扩展
│   └── Table.vue                       # elx-table 扩展列表组件
├── fields                              # 字段列表
│   ├── DjangoField.vue                 # models.Field （所有字段）
│   ├── DjangoFileField.vue             # models.FileField
│   ├── DjangoImageField.vue            # models.ImageField
│   ├── DjangoManyToManyField.vue       # models.ManyToManyField
│   ├── DjangoValue.vue                 # 值插槽，列表页和查看页使用
│   └── ForeignSelect.vue               # models.ForeignKey
├── form                                # 表单列表
│   ├── ChangePasswordDialog.vue        # 修改密码弹框
│   ├── ChangePasswordForm.vue          # 修改密码表单
│   ├── DialogOrDrawer.vue              # 弹框抽象（对话框/抽屉）
│   ├── EditDialog.vue                  # 编辑弹框
│   ├── EditForm.vue                    # 编辑表单
│   ├── ViewDialog.vue                  # 查看弹框
│   └── ViewForm.vue                    # 查看表单
├── index.vue                           # 总入口页
└── models                              # 数据模型
    └── meta.js                         # metainfo本地缓存
```
## 鉴权
```javascript
// src/utils/request.js 行 22
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
import Layout from '@/layout'

const components = {
  defaults: {
    'menu': Layout,
    'dashboard': () => import('@/views/Dashboard'),
    'django': () => import('@/components/Django')
  },
  custom: {

  },
  get(key) {
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
可以在这里加入菜单，菜单的格式在该文件头部有注释，也可以看[官方文档](https://panjiachen.github.io/vue-element-admin-site/zh/guide/essentials/router-and-nav.html)

> 注意，菜单中的roles已被废除掉

## element扩展组件 elx-table
exl-table是对ex-table列表组件的扩展，增强了如下功能

1. 集成了分页
2. 数据源参数data修改为function（原参是array），这样才能集成分页
3. 增加了多选功能，分页切换能保持选中

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

扩展参数说明

参数|说明|类型|默认值
---|---|---|---
data|数据源|function|-
row-key|主键名|string|-
page-size|分页页数|number|10
row-selection|列表项是否可选择|object|null

范例代码
```java
<template>
  <div class="app-container">
    <ext-table
      row-key="key"
      :data="loadData"
      :page-size="20"
      :row-selection="{selectedRowKeys: selectedRowKeys, onChange: handleSelectChange}"
    >
      <el-table-column prop="key" label="ID" />
      <el-table-column prop="name" label="名称" />
    </ext-table>
  </div>
</template>
<script>
import ExtTable from '@/components/Django/components/Table.vue'
export default {
  components: { ExtTable },
  data() {
    return {
      selectedRowKeys: []
    }
  },
  methods: {
    loadData(params) {
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          const data = []
          const pageSize = params.pageSize || 10
          const pageNo = params.pageNo || 1
          const maxCount = 150
          for (let i = 0; i < pageSize; i++) {
            const key = (pageNo - 1) * pageSize + i + 1
            if (key > maxCount) {
              break
            }
            const item = {
              key: key,
              name: `Item${key}`
            }
            data.push(item)
          }
          const result = {
            pageSize: pageSize,
            pageNo: pageNo,
            totalPage: Math.ceil(maxCount / pageSize),
            totalCount: maxCount,
            data: data
          }
          resolve(result)
        }, 1000)
      })
    },
    handleSelectChange(keys) {
      this.selectedRowKeys = keys
      console.log('select keys:', keys)
    }
  }
}
</script>
```