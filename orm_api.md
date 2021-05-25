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
* <b>丰富的字段类型</b>
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
<table>
    <tr>
        <td style="background-color: #fff0c9">
            <div style="color: #d57f28">警告</div>
            <div style="color: #5b4311">这意味着你不能定义相同名称的字段和方法，后者将会默默覆盖前者</div>
        </td>
    </tr>
</table>

默认情况下，字段的标签（用户可见的名称）是字段名称的大写，这可以用`string`参数覆盖
```python
field2 = fields.Integer(string="Field Label")
```
有关字段类型和参数的列表，请参阅the fields reference。
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
**API**<br>
#### class`odoo.models.` `BaseModel`<br>
Odoo模型的基类<br>
Odoo模型是通过继承以下其中之一来创建的:<br>
* `Model`用于常规持久性数据库模型<br>
* `TransientModel`用于临时数据，存储在数据库中，但每隔一段时间自动清除<br>
* `AbstractModel`用于多个继承模型共享的抽象超类<br>

系统对每个数据库的每个模型自动实例化一次。这些实例代表每个数据库上的可用模型，并依赖于该数据库上安装的模块。每个实例的实际类都是由创建并继承相应模型的Python类构建的。<br>
每个模型实例都是一个“记录集”，即模型记录的有序集合。记录集由诸如`browse()`，`search()`或字段访问等方法返回。记录没有显式表示:一个记录被表示为一个记录的记录集<br>
要创建一个不应该实例化的类，`_register`属性可以设置为False。<br>
#### `_auto` = False
是否应该创建数据库表(默认值:True)。如果设置为False，重写init()来创建数据库表。
<table>
    <tr>
        <td style="background-color: #f8f8f8">
            <div>
                <span style="color: #477674"><b>小技巧</b></span><br>
                <div style="color: #477674">要创建没有任何表的模型，请从`AbstractModel`继承</div>
            </div>
        </td>
    </tr>
</table>
