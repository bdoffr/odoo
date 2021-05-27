# 建立一个网站

> 警告
>
> 本指南假设您具备[Python的基本知识]()
>
> 本指南假定安装了[Odoo]()

## 创建基本模块

在Odoo中，任务通过创建模块来执行。

模块通过添加新行为或修改现有行为(包括其他模块添加的行为)来定制Odoo安装的行为。

[Odoo脚手架]()可以设置一个基本模块。要快速开始，只需调用:

```shell
$ ./odoo-bin scaffold Academy my-modules
```

这将自动创建一个my-modules模块目录，其中包含一个academy模块。如果需要，该目录可以是一个现有的模块目录，但模块名在该目录中必须是唯一的。

## 一个演示模块

我们有一个“完整的”模块准备安装。

虽然它没什么用，但我们可以安装它:

* 启动Odoo服务器
    ```shell
    $ ./odoo-bin --addons-path addons,my-modules
    ```
* 打开 http://localhost:8069

* 创建一个包含演示数据的新数据库

* 点击 **Settings ‣ Modules ‣ Modules**

* 在右上角，删除*Installed*过滤器，搜索*academy*

* 单击*Academy*模块的**Installed**按钮

## 到浏览器

[`Controllers`]()解释浏览器请求并发回数据。

添加一个简单的控制器，并确保它在`__init__.py`导入(这样Odoo可以找到它):

`academy/controllers.py`

```python
# -*- coding: utf-8 -*-
from odoo import http


class Academy(http.Controller):

    @http.route('/academy/academy/', auth='public')
    def index(self, **kw):
        return "Hello, world"
```

关闭服务器(^C)然后重新启动:

```shell
$ ./odoo-bin --addons-path addons,my-modules
```

打开一个页面到http://localhost:8069/academy/academy/，你应该看到你的“页面”出现:

![helloworld](https://www.odoo.com/documentation/13.0/zh_CN/_images/helloworld.png)

