# ORM API
### 对象关系映射模块
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