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

## 编辑字段

特定于记录的数据应该保存在该记录中，所以让我们为我们的老师添加一个新的传记字段:

##### `academy/models.py`

```python
class Teachers(models.Model):
    _name = 'academy.teachers'

    name = fields.Char()
    biography = fields.Html()
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
                <div>
                    <t t-esc="person.biography"/>
                </div>
            </div>
        </div>
        <div class="oe_structure"/>
    </t>
</template>
```

重启Odoo并更新视图，重新加载教师的页面，然后……该字段是不可见的，因为它不包含任何内容。

对于记录字段，模板可以使用特殊的`t-field`指令，该指令允许使用字段特定的接口从网站编辑字段内容。将*person*模板改为使用`t-field`字段:

##### `academy/templates.xml`

```xml

<div class="oe_structure">
    <div class="container">
        <h3 t-field="person.name"/>
        <div t-field="person.biography"/>
    </div>
</div>
```

重启Odoo并升级模块，现在老师的名字下有一个占位符，**编辑**模式下有块的新区域。删除到那里的内容存储在相应的教师`biography`
字段中，因此特定于该教师。

老师的名字也是可编辑的，保存后，更改在索引页面上可见。

`t-field`也可以采用取决于确切字段的格式设置选项。例如，如果我们显示教师记录的修改日期：

##### `academy/templates.xml`

```xml

<div class="oe_structure">
    <div class="container">
        <h3 t-field="person.name"/>
        <p>Last modified:
            <i t-field="person.write_date"/>
        </p>
        <div t-field="person.biography"/>
    </div>
</div>
```

它以非常“计算机化”的方式显示并且难以阅读，但是我们可以要求提供人类可读的版本：

##### `academy/templates.xml`

```xml

<div class="oe_structure">
    <div class="container">
        <h3 t-field="person.name"/>
        <p>Last modified:
            <i t-field="person.write_date" t-options='{"format": "long"}'/>
        </p>
        <div t-field="person.biography"/>
    </div>
</div>
```

或相对显示：

##### `academy/templates.xml`

```xml

<div class="oe_structure">
    <div class="container">
        <h3 t-field="person.name"/>
        <p>Last modified:
            <i t-field="person.write_date" t-options='{"widget": "relative"}'/>
        </p>
        <div t-field="person.biography"/>
    </div>
</div>
```

## 管理和ERP集成

### 对Odoo管理的简短而不完整的介绍

在网站支持部分中，对Odoo的管理进行了简要介绍。我们可以在菜单中使用**Administrator ‣ Administrator**(或者登录如果已登出)。

Odoo后端的概念结构很简单：

1.首先是菜单，记录树（菜单可以有子菜单）。菜单没有子菜单映射......

2.动作。动作有多种类型:Odoo应该执行的链接、报告、代码或数据显示。数据显示操作称为*窗口*操作，并告诉Odoo显示一个给定的*模型*根据一组视图…

3.视图具有类型、与之相对应的广泛类别(列表、图表、日历)和自定义模型在视图中显示方式的*体系结构*。

让我们为模型创建一个菜单:

##### `academy/__manifest__.py`

```python
# always loaded
'data': [
    'security/ir.model.access.csv',
    'templates.xml',
    'views.xml',
],
```

##### `academy/views.xml`

```xml

<odoo>
    <record id="action_academy_teachers" model="ir.actions.act_window">
        <field name="name">Academy teachers</field>
        <field name="res_model">academy.teachers</field>
    </record>

    <menuitem sequence="0" id="menu_academy" name="Academy"/>
    <menuitem id="menu_academy_content" parent="menu_academy"
              name="Academy Content"/>
    <menuitem id="menu_academy_content_teachers"
              parent="menu_academy_content"
              action="action_academy_teachers"/>
</odoo>
```

然后在左上角访问http://localhost:8069/web/ 应该是一个**Academy**
菜单，它是默认选择的，因为它是第一个菜单，并且已经打开了一个教师列表。从列表中可以**创建**新的教师记录，并切换到“表单”的记录视图。

如果没有关于如何呈现记录([视图]())的定义，Odoo会自动动态创建一个基本的记录。在我们的例子中，它目前适用于“list”视图(只显示教师的名字)
，但在“form”视图中，HTML`biography`字段与`name`
字段并排显示，没有给出足够的空间。让我们定义一个自定义表单视图，让查看和编辑教师记录有一个更好的体验:

##### `academy/views.xml`

```xml

<record id="academy_teacher_form" model="ir.ui.view">
    <field name="name">Academy teachers: form</field>
    <field name="model">academy.teachers</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <field name="name"/>
                <field name="biography"/>
            </sheet>
        </form>
    </field>
</record>
```

### 模型之间的关系

我们已经看到了直接存储在记录中的一对“基本”字段。有很多[基本字段]()。第二大类字段是[关系型]()的，用于将记录相互链接(在模型内或跨模型)。

为了进行演示，我们创建一个*课程*模型。每个课程应该有一个`teacher`字段，链接到单个教师记录，但每个教师可以教授许多课程:

##### `academy/models.py`

```python
class Courses(models.Model):
    _name = 'academy.courses'

    name = fields.Char()
    teacher_id = fields.Many2one('academy.teachers', string="Teacher")
```

##### `academy/security/ir.model.access.csv`

```postgresql
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_academy_teachers,access_academy_teachers,model_academy_teachers,,1,0,0,0
access_academy_courses,access_academy_courses,model_academy_courses,,1,0,0,0
```

我们也添加视图，这样我们就可以看到和编辑一个课程的老师:

##### `academy/views.xml`

```xml

<record id="action_academy_courses" model="ir.actions.act_window">
    <field name="name">Academy courses</field>
    <field name="res_model">academy.courses</field>
</record>
<record id="academy_course_search" model="ir.ui.view">
<field name="name">Academy courses: search</field>
<field name="model">academy.courses</field>
<field name="arch" type="xml">
    <search>
        <field name="name"/>
        <field name="teacher_id"/>
    </search>
</field>
</record>
<record id="academy_course_list" model="ir.ui.view">
<field name="name">Academy courses: list</field>
<field name="model">academy.courses</field>
<field name="arch" type="xml">
    <tree string="Courses">
        <field name="name"/>
        <field name="teacher_id"/>
    </tree>
</field>
</record>
<record id="academy_course_form" model="ir.ui.view">
<field name="name">Academy courses: form</field>
<field name="model">academy.courses</field>
<field name="arch" type="xml">
    <form>
        <sheet>
            <field name="name"/>
            <field name="teacher_id"/>
        </sheet>
    </form>
</field>
</record>

<menuitem sequence="0" id="menu_academy" name="Academy"/>
<menuitem id="menu_academy_content" parent="menu_academy"
          name="Academy Content"/>
<menuitem id="menu_academy_content_courses"
          parent="menu_academy_content"
          action="action_academy_courses"/>
<menuitem id="menu_academy_content_teachers"
          parent="menu_academy_content"
          action="action_academy_teachers"/>
```

也可以直接从教师的页面创建新课程，或查看他们所教的所有课程，所以添加`the inverse relationship`到教师模型:

##### `academy/models.py`

```python
class Teachers(models.Model):
    _name = 'academy.teachers'

    name = fields.Char()
    biography = fields.Html()

    course_ids = fields.One2many('academy.courses', 'teacher_id',
                                 string="Courses")


class Courses(models.Model):
    _name = 'academy.courses'

    name = fields.Char()
    teacher_id = fields.Many2one('academy.teachers', string="Teacher")
```

##### `academy/views.xml`

```xml

<record id="academy_teacher_form" model="ir.ui.view">
    <field name="name">Academy teachers: form</field>
    <field name="model">academy.teachers</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <field name="name"/>
                <field name="biography"/>
                <field name="course_ids">
                    <tree Sstring="Courses" editable="bottom">
                        <field name="name"/>
                    </tree>
                </field>
            </sheet>
        </form>
    </field>
</record>
```

### 讨论和通知

Odoo提供的技术模型不能直接满足业务需求，但可以为业务对象添加功能，而无需手动构建它们。

其中之一是*Chatter*
系统，Odoo电子邮件和信息系统的一部分，它可以向任何模型添加通知和讨论线程。模型只需`_inherit` `mail.thread`
，并将`message_ids`字段添加到其表单视图中，以显示讨论线程。讨论线程是每个记录。

对于我们的学院来说，允许讨论课程来解决问题是很有意义的，例如时间安排的改变或老师和助理之间的讨论:

##### `academy/__manifest__.py`

```python
'version': '0.1',

# any module necessary for this one to work correctly
'depends': ['website', 'mail'],

# always loaded
'data': [
```

##### `academy/models.py`

```python
class Courses(models.Model):
    _name = 'academy.courses'
    _inherit = 'mail.thread'

    name = fields.Char()
    teacher_id = fields.Many2one('academy.teachers', string="Teacher")
```

##### `academy/views.xml`

```xml

<record id="academy_course_form" model="ir.ui.view">
    <field name="name">Academy courses: form</field>
    <field name="model">academy.courses</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <field name="name"/>
                <field name="teacher_id"/>
            </sheet>
            <div class="oe_chatter">
                <field name="message_follower_ids" widget="mail_followers"/>
                <field name="message_ids" widget="mail_thread"/>
            </div>
        </form>
    </field>
</record>
```

在每个课程表单的底部，现在都有一个讨论线程，系统用户可以留言，关注或取消关注与特定课程相关的讨论。

### 课程销售

Odoo还提供允许更直接地使用或选择业务需求的业务模型。例如，该`website_sale`
模块根据Odoo系统中的产品建立电子商务网站。通过使我们的课程成为特定类型的产品，我们可以轻松地使课程订阅变得可销售。

而不是以前经典的继承，这意味着用*product*模型替换我们的*course*模型，并就地扩展产品（添加我们需要的任何东西）。

首先，我们需要添加一个依赖项`website_sale`以便我们同时获得产品（通过`sale`）和电子商务接口：

##### `academy/__manifest__.py`

```python
'version': '0.1',

# any module necessary for this one to work correctly

'depends': ['mail', 'website_sale'],

# always loaded

'data': [
```

重新启动Odoo，更新您的模块，现在网站上有一个商店部分，列出了许多预填充（通过演示数据）的产品。

第二步是将课程模型替换为`product.template`，并为课程添加一个新的产品类别：

##### `academy/__manifest__.py`

```python
    'security/ir.model.access.csv',
'templates.xml',
'views.xml',
'data.xml',
],
# 仅在演示模式下加载
'demo': [
```

##### `academy/data.xml`

```xml

<odoo>
    <record model="product.public.category" id="category_courses">
        <field name="name">Courses</field>
        <field name="parent_id" ref="website_sale.categ_others"/>
    </record>
</odoo>
```

##### `academy/demo.xml`

```xml

<record id="course0" model="product.template">
    <field name="name">Course 0</field>
    <field name="teacher_id" ref="padilla"/>
    <field name="public_categ_ids"
           eval="[(4, ref('academy.category_courses'), False)]"/>
    <field name="website_published">True</field>
    <field name="list_price" type="float">0</field>
    <field name="type">service</field>
</record>
<record id="course1" model="product.template">
<field name="name">Course 1</field>
<field name="teacher_id" ref="padilla"/>
<field name="public_categ_ids"
       eval="[(4, ref('academy.category_courses'), False)]"/>
<field name="website_published">True</field>
<field name="list_price" type="float">0</field>
<field name="type">service</field>
</record>
<record id="course2" model="product.template">
<field name="name">Course 2</field>
<field name="teacher_id" ref="vaughn"/>
<field name="public_categ_ids"
       eval="[(4, ref('academy.category_courses'), False)]"/>
<field name="website_published">True</field>
<field name="list_price" type="float">0</field>
<field name="type">service</field>
</record>
```

##### `academy/models.py`

```python
class Courses(models.Model):
    _name = 'academy.courses'
    _inherit = ['mail.thread', 'product.template']

    name = fields.Char()
    teacher_id = fields.Many2one('academy.teachers', string="Teacher")
```

安装后，商店中现在可以使用一些课程，尽管它们可能需要寻找。

> ##### 笔记
> * 在适当的位置扩展模型，是`inherited`而不是给它一个新的`_name`
> * `product.template`已经使用了讨论系统，所以我们可以从我们的扩展模型中删除它
> * 我们正在创建默认`发布`的课程，因此无需登录即可查看

### 改变现有视图

到目前为止，我们已经简要地看到：

* 新模型的创建
* 新视图的产生
* 新记录的创建
* 现有模型的改变

我们只剩下现有记录的更改和现有视图的更改。我们将在**商店**页面上同时进行。

视图更改是通过创建*扩展*
视图来完成的，这些视图应用于原始视图的顶部并对其进行更改。可以在不修改原始视图的情况下添加或删除这些更改视图，从而更容易尝试和回滚更改。

因为我们的课程是免费的，所以没有理由在商店页面上显示他们的价格，所以我们将改变视图并隐藏价格，如果它是0。第一个任务是找出哪个视图显示价格，这可以通过**
Customize ‣ Editor**完成，它可以让我们阅读渲染页面所涉及的各种模板。通过其中的一些，“Product item”看起来很可能是罪魁祸首。

改变视图架构分3个步骤完成：

1.创建新视图

2.通过将新视图设置为修改`inherit_id`后视图的外部 id来扩展要修改的视图

3.在架构中，使用`xpath`标签从修改后的视图中选择和更改元素

##### `academy/templates.xml`

```xml

<template id="product_item_hide_no_price"
          inherit_id="website_sale.products_item">
    <xpath expr="//div[hasclass('product_price')]/b" position="attributes">
        <attribute name="t-if">product.price &gt; 0</attribute>
    </xpath>
</template>

```

我们将更改的第二件事是默认情况下使产品类别侧边栏可见：**Customize ‣ Product Categories**
允许您打开和关闭产品类别树（用于过滤主显示）。

这是通过扩展模板的`customize_show`和`active`字段完成的：扩展模板（例如我们刚刚创建的模板）可以是*
customize_show=True*。此选项将在带有复选框的**自定义**菜单中显示视图，允许管理员激活或禁用它们（并轻松自定义其网站页面）。

我们只需要修改*Product Categories*记录并将其默认设置为*active=”True”*：

##### `academy/templates.xml`

```xml

<record id="website_sale.products_categories" model="ir.ui.view">
    <field name="active" eval="True"/>
</record>
```

有了这个，产品类别侧边栏将在安装`Academy`模块时自动启用。
