# 建立一个网站

> 警告
>
> 本指南假设您具备[Python的基本知识](https://docs.python.org/2/tutorial/)
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

##### `academy/controllers.py`

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

打开一个页面到http://localhost:8069/academy/academy/ ，你应该看到你的“页面”出现:

![](https://www.odoo.com/documentation/13.0/zh_CN/_images/helloworld.png)

## 模板

用Python生成HTML并不是一件很愉快的事情。

通常的解决方案是[模板]()、带有占位符的伪文档和显示逻辑。Odoo允许任何Python模板系统，但提供自己的[QWeb]()模板系统，与其他特性集成。

创建一个模板，并确保模板文件注册在`__manifest__.py`清单中，并修改控制器以使用我们的模板:

##### `academy/controllers.py`

```python
class Academy(http.Controller):

    @http.route('/academy/academy/', auth='public')
    def index(self, **kw):
        return http.request.render('academy.index', {
            'teachers': ["Diana Padilla", "Jody Caroll", "Lester Vaughn"],
        })
```

##### `academy/templates.xml`

```xml

<odoo>

    <template id="index">
        <title>Academy</title>
        <t t-foreach="teachers" t-as="teacher">
            <p>
                <t t-esc="teacher"/>
            </p>
        </t>
    </template>

</odoo>
```

模板对所有教师进行迭代(`t-foreach`)(通过模板上下文传递)，并将每个教师打印在自己的段落中。

最后重启Odoo并更新模块数据(安装模板)，点击**Settings ‣ Modules ‣ Modules ‣ Academy**然后点击**
Upgrade**。

> 小技巧
>
> 作为选择，Odoo可以同时重启和[`更新模块`]():
>
>```shell
> $ odoo-bin --addons-path addons,my-modules -d academy -u academy
> ```

打开http://localhost:8069/academy/academy/ 现在应该是:

![](https://www.odoo.com/documentation/13.0/zh_CN/_images/basic-list.png)

## 在Odoo中存储数据

[Odoo模型]()映射到数据库表。 在前一节中，我们只显示了Python代码中静态输入的字符串列表。这不允许修改或持久存储，所以现在我们将把数据移到数据库中。

### 定义数据模型

定义一个teacher模型，并确保它是从`__init__.py`中导入的，以便正确加载:

##### `academy/models.py`

```python
from odoo import models, fields, api


class Teachers(models.Model):
    _name = 'academy.teachers'

    name = fields.Char()
```

然后为模型设置[`基本访问控制`]()，并将它们添加到清单中:

##### `academy/__manifest__.py`

```python
# always loaded
'data': [
    'security/ir.model.access.csv',
    'templates.xml',
],
```

##### `academy/security/ir.model.access.csv`

```postgresql
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_academy_teachers,access_academy_teachers,model_academy_teachers,,1,0,0,0
```

这只是为`perm_read`所有用户（`group_id:id` 保留为空）提供了读取访问权限。
> 注解
>
> [数据文件]()(XML或CSV)必须添加到模块清单中，Python文件(模型或控制器)不必，但必须从`__init__.py`(直接或间接)导入。

> 警告
>
> 管理员用户绕过访问控制，他们可以访问所有模型，即使没有授予访问权限

### 演示数据

第二步是向系统添加一些演示数据，以便能够轻松地对其进行测试。这是通过添加一个`demo`[数据文件]()来完成的，它必须从清单中链接:

###### `academy/demo.xml`

```xml

<odoo>

    <record id="padilla" model="academy.teachers">
        <field name="name">Diana Padilla</field>
    </record>
    <record id="carroll" model="academy.teachers">
        <field name="name">Jody Carroll</field>
    </record>
    <record id="vaughn" model="academy.teachers">
        <field name="name">Lester Vaughn</field>
    </record>

</odoo>
```

> 小技巧
>
> [数据文件]()可以用于演示和非演示数据。演示数据仅以“演示模式”加载，可用于流量测试和演示，非演示数据总是加载和用于初始系统设置。
>
> 在本例中，我们使用演示数据是因为系统的实际用户希望输入或导入他们自己的教师列表，这个列表只用于测试。

### 访问数据

最后一步是修改模型和模板来使用我们的演示数据:

1.从数据库中获取记录，而不是使用静态列表

2.因为[`search()`]()返回一组与过滤器匹配的记录(这里是“所有记录”)，所以修改模板以打印每个老师的`name`

##### `academy/controllers.py`

```python
class Academy(http.Controller):

    @http.route('/academy/academy/', auth='public')
    def index(self, **kw):
        Teachers = http.request.env['academy.teachers']
        return http.request.render('academy.index', {
            'teachers': Teachers.search([])
        })
```

##### academy/templates.xml

```xml

<odoo>

    <template id="index">
        <title>Academy</title>
        <t t-foreach="teachers" t-as="teacher">
            <p>
                <t t-esc="teacher.id"/>
                <t t-esc="teacher.name"/>
            </p>
        </t>
    </template>

</odoo>
```

重启服务并更新模块(以便更新清单和模板并加载演示文件)，然后导航到http://localhost:8069/academy/academy/
。页面看起来应该略有不同:名称应该简单地用数字作为前缀(教师的数据库标识符)。

## 网站支持

Odoo捆绑了一个专门用于构建网站的模块。

到目前为止，我们已经相当直接地使用了控制器，但是Odoo 8通过`网站`模块添加了更深层次的集成和其他一些服务(例如默认样式，主题)。

1.首先，添加`website`对`academy`的依赖

2.然后在控制器上添加`website=True`标签，这将在[请求对象]()上设置一些新变量，并允许在模板中使用网站布局

3.使用模板中的网络布局

##### `academy/__manifest__.py`

```python
'version': '0.1',

# any module necessary for this one to work correctly
'depends': ['website'],

# always loaded
'data': [
```

##### `academy/controllers.py`

```python
class Academy(http.Controller):

    @http.route('/academy/academy/', auth='public', website=True)
    def index(self, **kw):
        Teachers = http.request.env['academy.teachers']
        return http.request.render('academy.index', {
            'teachers': Teachers.search([])
        })
```

##### `academy/templates.xml`

```xml

<odoo>

    <template id="index">
        <t t-call="website.layout">
            <t t-set="title">Academy</t>
            <div class="oe_structure">
                <div class="container">
                    <t t-foreach="teachers" t-as="teacher">
                        <p>
                            <t t-esc="teacher.id"/>
                            <t t-esc="teacher.name"/>
                        </p>
                    </t>
                </div>
            </div>
        </t>
    </template>

</odoo>
```

在重新启动服务器并更新模块之后(为了更新清单和模板)，访问http://localhost:8069/academy/academy/
应该会产生一个外观更好的页面，带有品牌和许多内置的页面元素(顶级菜单、页脚……)
![](https://www.odoo.com/documentation/13.0/zh_CN/_images/layout.png)
网站布局还提供了对编辑工具的支持:单击**Sign In**(在右上角)，填写凭证(默认为`admin` / `admin`)，然后单击**Log In**。

您现在处于Odoo的“性能”模式:管理界面。现在单击**Website**菜单项(左上角)。

我们回到了网站，但作为一个管理员，访问由网站支持提供的高级编辑功能:

* 一个模板代码编辑器(**Customize‣HTML editor**)，在这里你可以看到并编辑当前页面使用的所有模板
* 在左上角的**编辑**按钮切换到“编辑模式”，块(片段)和富文本编辑可用
* 许多其他功能，如移动预览或SEO

## 网址和路由

控制器方法通过`route()`装饰器与路由关联，该装饰器接受一个路由字符串和许多属性来定制其行为或安全性。

我们已经看到了一个“文字”路由字符串，它与URL部分完全匹配，但是路由字符串也可以使用[转换模式]()
来匹配URL的位并将其作为本地变量。例如，我们可以创建一个新的控制器方法，它接受URL的位并打印出来:

##### `academy/controllers.py`

```python
# New route
@http.route('/academy/<name>/', auth='public', website=True)
def teacher(self, name):
    return '<h1>{}</h1>'.format(name)
```

重启Odoo，访问http://localhost:8069/academy/Alice/
和http://localhost:8069/academy/Bob/ ，看看区别。

顾名思义，[转换模式]()不只是进行提取，它们还进行验证和转换，因此我们可以将新控制器更改为只接受整数:

##### `academy/controllers.py`

```python
@http.route('/academy/<int:id>/', auth='public', website=True)
def teacher(self, id):
    return '<h1>{} ({})</h1>'.format(id, type(id).__name__)
```

重新启动Odoo，访问http://localhost:8069/academy/2
，注意旧值是字符串，而新值是如何转换为整数的。尝试访问http://localhost:8069/academy/Carol/，注意没有找到页面:因为“Carol”不是一个整数，路由被忽略，没有找到路由。

Odoo提供了一个名为`model`的额外转换器，它在给定id时直接提供记录。让我们使用它来创建一个教师传记的通用页面:

##### `academy/controllers.py`

```python
@http.route('/academy/<model("academy.teachers"):teacher>/', auth='public',
            website=True)
def teacher(self, teacher):
    return http.request.render('academy.biography', {
        'person': teacher
    })
```

##### `academy/templates.xml`

```xml

<template id="biography">
    <t t-call="website.layout">
        <t t-set="title">Academy</t>
        <div class="oe_structure"/>
        <div class="oe_structure">
            <div class="container">
                <h3>
                    <t t-esc="person.name"/>
                </h3>
            </div>
        </div>
        <div class="oe_structure"/>
    </t>
</template>
```

然后更改模型列表以链接到我们的新控制器:

##### `academy/templates.xml`

```xml

<template id="index">
    <t t-call="website.layout">
        <t t-set="title">Academy</t>
        <div class="oe_structure">
            <div class="container">
                <t t-foreach="teachers" t-as="teacher">
                    <p>
                        <a t-attf-href="/academy/{{ slug(teacher) }}">
                            <t t-esc="teacher.name"/>
                        </a>
                    </p>
                </t>
            </div>
        </div>
    </t>
</template>
```

重启Odoo并升级模块，然后就可以访问每个老师的页面了。作为一个练习，试着在一个老师的页面上添加块来写一个传记，然后去到另一个老师的页面，以此类推。您将发现，您的传记在所有教师之间是共享的，因为模板中添加了区块，并且传记模板在所有教师之间是共享的，当编辑一个页面时，所有的页面都是同时编辑的。





