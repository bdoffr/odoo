# VIews

视图定义了记录应该如何显示给最终用户。它们是用XML指定的，这意味着它们可以独立于它们所表示的模型进行编辑。它们是灵活的，并且允许对它们所控制的屏幕进行高级定制。有各种各样的观点。每一个都代表了一种可视化模式:形式、列表、看板等等。

## Generic structure

基本视图通常共享下面定义的公共结构。占位符用大写表示。

```xml

<record id="MODEL_view_TYPE" model="ir.ui.view">
    <field name="name">NAME</field>
    <field name="model">MODEL</field>
    <field name="arch" type="xml">
        <VIEW_TYPE>
            <VIEW_SPECIFICATIONS/>
        </VIEW_TYPE>
    </field>
</record>
```

## Fields

视图对象公开了许多字段。它们是可选的，除非另有说明。

* `name`(mandatory)[`Char`]()

当在某种排序的列表中寻找一个时，仅用于视图的mnemonic/description。

* `model` [`Char`]()

适用模型链接到视图

* `priority` [`Integer`]()

当一个视图被(`model`，`type`)请求时，匹配模型和类型，具有最低优先级的视图将被返回(它是默认视图)。

它还定义了[`view inheritance`]()期间视图应用程序的顺序。

* `groups_id` [`Many2many`]()->`odoo.addons.base.models.res_users.Groups

允许使用/访问当前视图的组。

如果该视图扩展了一个现有的视图，那么该扩展只适用于一个用户，如果该用户可以访问所提供的`groups_id`。

* `arch` [`Text`]()

视图布局的描述。

## Attributes

不同的视图类型具有各种各样的属性，允许对通用行为进行自定义。这里将解释一些主要属性。它们并不是对所有视图类型都有影响。

> 注解
>
> 当前上下文和用户访问权限也可能影响视图功能。

* `create`

  在视图上禁用/启用记录创建。

* `edit`(`form` & `list` & `gantt`)

  在视图上禁用/启用记录版本。

* `delete`(`form` & `list`)

  通过**Action**下拉菜单禁用/启用视图上的记录删除。

* `duplicate`(`form` & `list`)

  通过Action下拉菜单在视图上禁用/启用记录复制。

* `decoration-$`(`list` & `gantt`)
  根据相应记录的属性，以行文本的样式定义记录的条件显示。

  值是Python的表达式。对于每条记录，表达式将以记录的属性作为上下文值进行计算，如果为`true`，则将相应的样式应用于该行。其他上下文值是`uid`(当前用户的id)和`current_date`(当前日期为`YYYY-MM-DD`
  格式的字符串)。

```xml

<tree decoration-info="state == 'draft'"
      decoration-danger="state == 'help_needed'"
      decoration-bf="state='busy'">
    <TREE_VIEW_CONTENT>
</tree>
```

> 警告
>
> 两种视图类型支持的值不同。甘特视图只支持`success`、`info`、`warning`、`danger`和`secondary`显示。列表视图支持`bf`, `it`，`success`，`info`，`warning`，`danger`，`muted`和`primary`显示。

* `banner_route`要获取的路由地址并添加到视图前。

如果设置了这个属性，[controller route url]()将被获取并显示在视图上方。来自控制器的json响应应该包含一个" html "键。

如果html包含一个样式表<link>标记，它将被删除并附加到<head>。

为了与后端交互，你可以使用\<a type=”action”>标签。请查看AbstractController的_onActionClicked方法的文档(
addons/web/static/src/js/views/abstract_controller.js)以获得更多细节。

只有扩展了AbstractView和AbstractController的视图可以使用此属性，如[Form]()、[Kanban]()、[List]()等。

范例:

```xml

<tree banner_route="/module_name/hello"/>
```

```python
class MyController(odoo.http.Controller):
    @http.route('/module_name/hello', auth='user', type='json')
    def hello(self):
        return {
            'html': """
                <div>
                    <link href="/module_name/static/src/css/banner.css"
                        rel="stylesheet">
                    <h1>hello, world</h1>
                </div> """
        }
```

## Inheritance

### Inheritance fields

下面的两个`View`字段用于指定继承的视图。

* `inherit_id` [`Many2one`]()

  当前视图的父视图，默认未设置。使用`ref`属性指定父类:
  ```xml
  <field name="inherit_id" ref="library.view_book_form"/>
  ```

* `mode` [`Selection`](): `extension / primary`
  继承模式，如果设置了`inherit_id`，则默认`extension`，否则为`primary`。

  `mode`在使用时，要覆盖的一个示例`inherit_id`
  是委托继承。在这种情况下，你的派生模型将与其父模型分离，并且与一个模型匹配的视图将与另一个模型不匹配。假设您从与父模型关联的视图继承，并且想要自定义派生视图以显示派生模型中的数据。该`mode`的派生视图需要被设置为`primary`
  ，因为它是派生模型的基本(可能也是唯一)视图。否则[view matching]()规则将不适用。

### View matching

* 如果一个视图被`(model, type)`请求，视图具有正确的模型和类型，`mode=primary`和最低优先级的视图将被匹配。
* 当一个视图被`id`请求时，如果它的模式不是`primary`，那么它与`primary`模式最接近的父模式将被匹配。

### View resolution

分辨率`arch`为请求的/匹配的`primary`视图生成最终视图:

1.如果视图具有父视图，则父视图将完全解析，然后应用当前视图的继承规范

2.如果视图不具有父视图，`arch`则按原样使用它

3.`extension`查找当前视图的具有mode的子级，并深度优先应用其继承规范（先应用子级，然后是其子级，然后是其兄弟级）

应用子视图的结果产生最终结果`arch`
