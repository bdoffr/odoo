# ORM API

**对象关系映射模块**

* 分级结构
* 约束性 一致性和验证
* 对象元数据取决于其状态
* 通过复杂查询优化处理（同时多线程处理）
* 默认字段赋值
* 权限优化
* 持久化对象：postgresql 数据库
* 数据转换
* 多级缓存系统
* 两种不同的继承机制
* 丰富的字段类型
    * 经典型（字符型，整形，布尔型）
    * 关系型（一对多，多对一，多对多）
    * 函数型

## 模型

模型字段被定义为模型本身的属性：

```python
from odoo import models, fields


class AModel(models.Model):
    _name = 'a.model.name'

    field1 = fields.Char()
```

    警告
    这意味着你不能定义相同名称的字段和方法，后者将会默默覆盖前者

默认情况下，字段的标签（用户可见的名称）是字段名称的大写，这可以用`string`参数覆盖

```python
field2 = fields.Integer(string="Field Label")
```

有关字段类型和参数的列表，请参阅[the fields reference]()。

默认值被定义为字段上的参数，或者作为值：

```python
name = fields.Char(default="a value")
```

或者作为一个计算默认值的函数调用，它应该返回该值:

```python
def _default_name(self):
    return self.get_value()


name = fields.Char(default=lambda self: self._default_name())
```

**API**

#### class`odoo.models.` `BaseModel`

Odoo模型的基类

Odoo模型是通过继承以下其中之一来创建的:

* [`Model`]()用于常规持久性数据库模型
* [`TransientModel`]()用于临时数据，存储在数据库中，但每隔一段时间自动清除
* [`AbstractModel`]()用于多个继承模型共享的抽象超类

系统对每个数据库的每个模型自动实例化一次。这些实例代表每个数据库上的可用模型，并依赖于该数据库上安装的模块。每个实例的实际类都是由创建并继承相应模型的Python类构建的。<br>
每个模型实例都是一个“记录集”，即模型记录的有序集合。记录集由诸如[`browse()`]()，[`search()`]()或字段访问等方法返回。记录没有显式表示:一个记录被表示为一个记录的记录集<br>
要创建一个不应该实例化的类，[`_register`]()属性可以设置为False。<br>

#### `_auto` = False

是否应该创建数据库表(默认值:True)。如果设置为False，重写init()来创建数据库表。

    小技巧
    要创建没有任何表的模型，请从AbstractModel继承

#### `_table` = None

如果[`_auto`]()，模型使用SQL表名

#### `_sequence` = None

用于ID字段的SQL序列

#### `_sql_constraints` = []

SQL约束[(name, sql_def, message)]

#### `_register` = True

在ORM注册表中不可见

#### _name = None

模型名称(点符号，模块名称空间)

#### `_description` = None

Python继承的模型:

    类型：str或list(str)

> 注解
>* 如果[`_name`]()已经设置，要继承父模型的名称
>* 如果[`_name`]()未设置，要就近拓展单个模型的名称

#### `_inherits` = {}

字典{'parent_model': 'm2o_field'}将父业务对象的_name映射为要使用的对应外键字段的名称:

```python
_inherits = {
    'a.model': 'a_field_id',
    'b.model': 'b_field_id'
}
```

实现基于组合的继承:新模型公开继承模型的所有字段，但不存储它们:值本身保存在链接的记录中。

    警告：
    如果在_inherited -ed模型中定义了多个同名的字段，则继承的字段将对应于最后一个字段(按照继承列表的顺序)。

#### `_rec_name` = None

用于标记记录的字段， 默认值：`name`

#### `_order` = 'id'

搜索结果默认顺序的字段

#### `_check_company_auto` = False

在写和创建时，调用`_check_company`以确保同伴在具有`check_company=True`属性的关系字段上的一致性

#### `_parent_name` = 'parent_id'

多对一字段中用作父字段

#### `_parent_store` = False

设置为True用于计算`parent_path`的字段

在[`parent_path`]()字段旁边，设置记录树结构的索引存储，以使用`child_of`和`parent_of`域操作符对当前模型的记录进行更快的分层查询。

#### `_abstract` = True

是否是抽象模型

> 参考
>
>[odoo.models.AbstractModel]()

#### `_transient` = False

是否是瞬时模型

> 参考
>
> [`odoo.models.TransientModel`]()

#### `_date_name` = 'date'

用于默认日程视图的字段

#### `_fold_name` = 'fold'

在看板视图中确定折叠的组的字段

### AbstractModel

`odoo.models.` `AbstractModel`

[odoo.models.BaseModel]()的别名

### Model

#### class `odoo.models.` `Model`

Odoo模型数据持久化的主要超类

Odoo模型是通过继承这个类创建的

```python
class user(Model):
    ...
```

之后，系统将对每个数据库实例化一个类(类的模块安装在该数据库上)。

###TransientModel

class `odoo.models.` `TransientModel`

为瞬时记录建模的超类，意味着临时持久化，并定期进行真空清理。
l
TransientModel具有简化的访问权限管理，所有用户都可以创建新的记录，并且只能访问他们创建的记录。超级用户可以无限制地访问所有TransientModel记录。

## Fields

#### class `odoo.fields.` `Field`

字段描述符包含字段定义，并管理记录上对应字段的访问和赋值。当实例化一个字段时，可以提供以下属性:



