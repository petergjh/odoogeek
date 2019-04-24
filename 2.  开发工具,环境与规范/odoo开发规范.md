# odoo开发规范
### 模块结构
#### 文件夹列表及对应作用

data/:演示和数据文件
models/:模型定义
controllers/:控制器(包含HTTP路由)
views/:视图和模版
static/:web资源，包括css/, js/, img/, lib/, ...
wizard/:向导及其视图
report/:报表
tests/:单元测试代码

#### 文件命名
model命名:
业务model放置在一个文件里，如果模块只包含一个model，它的名字就与模块名一致。如：

models/<main_model>.py
models/<inherited_main_model>.py
views/<main_model>_templates.xml
views/<main_model>_views.xml
data/<main_model>_demo.xml
data/<main_model>_data.xml

例如：销售模块包含sale_order和sale_order_line两个模型，并且sale_order是主模型，所以文件将被命名为models/sale_order.py和 views/sale_order_views.py。
对于数据文件命名，按用途进行命名：demo或者data。例如:data/sale_order_demo.xml和data/sale_order_data.xml
每个模块的控制器都放在一个文件中，命名为main.py。如果是从另一个模块继承的，则将其命名为<module_name>.py。

统计报表命名：
report/<report_name>_report.py
report/<report_name>_report_views.py

可打印报表：
report/<print_report_name>_reports.py
report/<print_report_name>_templates.xml

#### 完整的文件列表如下：

```
addons/<module_name>/
|-- __init__.py
|-- __manifest__.py
|-- controllers/
|   |-- __init__.py
|   |-- <inherited_module_name>.py
|   `-- main.py
|-- data/
|   |-- <main_model>_data.xml
|   `-- <inherited_main_model>_demo.xml
|-- models/
|   |-- __init__.py
|   |-- <main_model>.py
|   `-- <inherited_main_model>.py
|-- report/
|   |-- __init__.py
|   |-- <main_stat_report_model>.py
|   |-- <main_stat_report_model>_views.xml
|   |-- <main_print_report>_reports.xml
|   `-- <main_print_report>_templates.xml
|-- security/
|   |-- ir.model.access.csv
|   `-- <main_model>_security.xml
|-- static/
|   |-- img/
|   |   |-- my_little_kitten.png
|   |   `-- troll.jpg
|   |-- lib/
|   |   `-- external_lib/
|   `-- src/
|       |-- js/
|       |   `-- <my_module_name>.js
|       |-- css/
|       |   `-- <my_module_name>.css
|       |-- scss/
|       |   `-- <my_module_name>.scss
|       `-- xml/
|           `-- <my_module_name>.xml
|-- views/
|   |-- <main_model>_templates.xml
|   |-- <main_model>_views.xml
|   |-- <inherited_main_model>_templates.xml
|   `-- <inherited_main_model>_views.xml
`-- wizard/
    |-- <main_transient_A>.py
    |-- <main_transient_A>_views.xml
    |-- <main_transient_B>.py
    `-- <main_transient_B>_views.xml
```


### XML文件
#### 格式
当定义一个记录的xml时，需要<record>标记:


id属性放在model属性前
字段(field)定义中，name属性放在第一个，其他属性根据重要性的高低顺序排放
字段的值放在<field>标签内，或者放在eval属性中。
标签<data>仅用于设置不可更新的数据noupdate=1,如果整个xml文件都是不可更新数据，则noupdate=1属性可以设置在<odoo>标签上，而不需要<data>标签。

```
<record id="view_id" model="ir.ui.view">
    <field name="name">view.name</field>
    <field name="model">object_name</field>
    <field name="priority" eval="16"/>
    <field name="arch" type="xml">
        <tree>
            <field name="my_field_1"/>
            <field name="my_field_2" string="My Label" widget="statusbar" statusbar_visible="draft,sent,progress,done" />
        </tree>
    </field>
</record>
```

Odoo定义了一些标签作为快捷方式：

menuitem：作为ir.ui.menu的快捷方式
template: 表示只需要arch视图部分的QWeb视图
report: 用于定义报表action

act_window：当record用不了的时候用它

#### xml_id命名
权限(Security)、视图(View)和动作(Action)使用的命名规则：

菜单(menu): <model_name>_menu

视图(view): <model_name>_view_<view_type>,view_type可能的取值有：kanban, form, tree, search

动作(action): 主动作命名为<model_name>_action，其他的动作命名为<model_name>_action_<detail>，其中<detail>使用小写字母简单描述动作
组(group): <model_name>_group_<group_name>,group_name可能的取值包括:user, manager,...
规则(rule): <model_name>_rule_<concerned_group>,concerned_group可能的取值包括: 模型用户规则user, 公共用户规则public,多公司用户规则company


<!-- views and menus -->
<record id="model_name_view_form" model="ir.ui.view">
    ...
</record>

<record id="model_name_view_kanban" model="ir.ui.view">
    ...
</record>

<menuitem
    id="model_name_menu_root"
    name="Main Menu"
    sequence="5"
/>
<menuitem
    id="model_name_menu_action"
    name="Sub Menu 1"
    parent="module_name.module_name_menu_root"
    action="model_name_action"
    sequence="10"
/>

<!-- actions -->
<record id="model_name_action" model="ir.actions.act_window">
    ...
</record>

<record id="model_name_action_child_list" model="ir.actions.act_window">
    ...
</record>

<!-- security -->
<record id="module_name_group_user" model="res.groups">
    ...
</record>

<record id="model_name_rule_public" model="ir.rule">
    ...
</record>

<record id="model_name_rule_company" model="ir.rule">
    ...
</record>


视图名称(name)使用点表示法:my.model.view_type 或者 my.model.view_type.inherit


#### 继承XML的命名
继承视图的命名规则：<base_view>_inherit_<current_module_name>,其中_inherit_<current_module_name>是扩展视图的模块的技术名称。
```
<record id="inherited_model_view_form_inherit_my_module" model="ir.ui.view">
    ...
</record>
```


### Python
Odoo源代码基本准守Python标准PEP8，但是忽略其中一些规则：

E501：行太长了
E301：预计有1个空行，找到0
E302：预计有2个空行，找到1
E126：延长线过度缩进以用于悬挂缩进
E123：关闭支架与开口支架线的压痕不匹配
E127：延伸线过度缩进以进行视觉缩进
E128：用于视觉缩进的缩进的延续线
E265：阻止评论应以'＃'开头

#### Import
import 顺序

外部库
导入odoo
导入odoo的模块

在每组中的导入按字母顺序排序

```
# 1 : 导入python库
import base64
import re
import time
from datetime import datetime
# 2 :  imports of odoo
import odoo
from odoo import api, fields, models # alphabetically ordered
from odoo.tools.safe_eval import safe_eval as eval
from odoo.tools.translate import _
# 3 :  imports from odoo modules
from odoo.addons.website.models.website import slug
from odoo.addons.web.controllers.main import login_redirect
```

#### 编程习惯

每个python文件都应该以 # -*- coding: utf-8 -*- 作为第一行。
简单易读的代码

#### Odoo中编程

避免创建生成器和装饰器：仅使用Odoo API已有的
使用filtered，mapped，sorted方法来提升代码可读性和性能。
使用更易理解的方法名

#### 让你的方法可以批量处理
当添加一个函数时，确保它可以处理多重数据，如通过api.multi()装饰器，可以在self上进行循环处理
```
@api.multi
def my_method(self)
    for record in self:
        record.do_cool_stuff()

```

避免使用api.one装饰器，因为它可能不会像你想象中一样工作。
为了更好的性能，比如当定义一个状态按钮时，不在api.multi循环里用search和search_count方法，而用read_group一次计算

```
@api.multi
def _compute_equipment_count(self):
""" Count the number of equipement per category """
    equipment_data = self.env['hr.equipment'].read_group([('category_id', 'in', self.ids)], ['category_id'], ['category_id'])
    mapped_data = dict([(m['category_id'][0], m['category_id_count']) for m in equipment_data])
    for category in self:
        category.equipment_count = mapped_data.get(category.id, 0)
```

#### 上下文环境
在新API中，context变量是不能修改的。可以通过with_context来使用新的运行环境调用方法。

```


records.with_context(new_context).do_stuff() # all the context is replaced
records.with_context(**additionnal_context).do_other_stuff() # additionnal_context values override native context ones
```

#### 尽量使用ORM
当ORM可以实现的时候尽量使用ORM而不要直接写sql，因为它可能会绕过orm的一些规则如权限、事务等，还会让代码变得难读且不安全。

```
# 错误的写法，注入风险，代码效率低
self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in (' + ','.join(map(str, ids))+') AND state=%s AND obj_price > 0', ('draft',))
auction_lots_ids = [x[0] for x in self.env.cr.fetchall()]

# 不会被注入，但仍然是错误的写法
self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in %s '\
           'AND state=%s AND obj_price > 0', (tuple(ids), 'draft',))
auction_lots_ids = [x[0] for x in self.env.cr.fetchall()]

# 推荐的写法
auction_lots_ids = self.search([('auction_id','in',ids), ('state','=','draft'), ('obj_price','>',0)])
```


#### 防止注入
不要用python的+号连接符、%解释符来拼sql
```
# 错误的写法
self.env.cr.execute('SELECT distinct child_id FROM account_account_consol_rel ' +
           'WHERE parent_id IN ('+','.join(map(str, ids))+')')

# 推荐的写法
self.env.cr.execute('SELECT DISTINCT child_id '\
           'FROM account_account_consol_rel '\
           'WHERE parent_id IN %s',
           (tuple(ids),))
```

#### 不要手动提交事务
odoo有自己的一套机制用于事务处理

#### 正确的使用翻译方法
odoo用一个下划线方法来表明某字段需要翻译，该方法通过from odoo.tools.translate import _导入。一般情况下该方法只能被用在代码里的规定字符串的翻译，不能用于动态字符串的翻译，翻译方法的调用只能是_('literal string')，里面不能加其他的。

```
# 好的方式，简洁
error = _('This record is locked!')

# 好的方式，包含格式的字符串
error = _('Record %s cannot be modified!') % record

# 好的方式，多行文字的字符串
error = _("""This is a bad multiline example
             about record %s!""") % record
error = _('Record %s cannot be modified' \
          'after being validated!') % record

# 错误的方式：试图在字符串格式化后进行翻译
# 这样没有作用，而只会把翻译搞乱
error = _('Record %s cannot be modified!' % record)

# 错误：动态字符串，不能这样翻译
error = _("'" + que_rec['question'] + "' \n")

# 错误：字段值将会被系统字段翻译，而不会获取预期结果
error = _("Product %s is out of stock!") % _(product.name)
# 错误的方式：试图在字符串格式化后进行翻译
error = _("Product %s is out of stock!" % product.name)
```


### 符号和命名

##### 模型名-使用.分隔，模块名做前缀

定义odoo模型时，使用单数形式的名字如res.user,res.partner

定义wizard时，命名格式为<related_base_model>.<action>，related_base_model是关联模型名称，action是功能简称，如account.invoice.make

定义报表模型时，使用<related_base_model>.report.<action>，和wizard一样


##### python类-使用驼峰命名方式

```
class AccountInvoice(models.Model):
    ...

class account_invoice(osv.osv):
    ...
```


##### 变量名

模型变量使用驼峰命名方式
普通变量用下划线+小写字母
由于新api中记录是集合形式，当变量不包含id时不以id作后缀

```
ResPartner = self.env['res.partner']
partners = ResPartner.browse(ids)
partner_id = partners[0].id
```


One2Many， Many2Many字段一般以ids作为后缀如：sale_order_line_ids
Many2One 一般以_id为后缀如：partner_id, user_id


##### 方法命名

计算字段 - 计算方法一般是_compute_<field_name>

搜索方法 - _search_<field_name>

默认方法 - _default_<field_name>

onchange方法 - _onchange_<field_name>

约束方法 - _check_<constraint_name>

action方法 - 一个对象的动作方法一般以action_开头，它的装饰器是@api.multi，如果它只使用单条计算，可在方法头添加self.ensure_one()



##### 模型中属性顺序

私有属性：_name, _description, _inherit, ...
默认方法和_default_get
字段声明
计算和搜索方法和字段声明顺序一致
约束方法(@api.constrains)和onchange方法(@api.onchange)
CRUD方法
action方法
其他业务方法


```
class Event(models.Model):
    # 私有属性
    _name = 'event.event'
    _description = 'Event'

    # 默认方法
    def _default_name(self):
        ...

    # 字段声明
    name = fields.Char(string='Name', default=_default_name)
    seats_reserved = fields.Integer(oldname='register_current', string='Reserved Seats',
        store=True, readonly=True, compute='_compute_seats')
    seats_available = fields.Integer(oldname='register_avail', string='Available Seats',
        store=True, readonly=True, compute='_compute_seats')
    price = fields.Integer(string='Price')

    # 计算和搜索方法,与字段申明顺序一致
    @api.multi
    @api.depends('seats_max', 'registration_ids.state', 'registration_ids.nb_register')
    def _compute_seats(self):
        ...

    # 约束方法和onchange方法
    @api.constrains('seats_max', 'seats_available')
    def _check_seats_limit(self):
        ...

    @api.onchange('date_begin')
    def _onchange_date_begin(self):
        ...

    # CRUD方法
    def create(self, values):
        ...

    # action方法
    @api.multi
    def action_validate(self):
        self.ensure_one()
        ...

    # 其他业务方法
    def mail_user_confirm(self):
        ...
```


### Javascript和CSS

在所有javascript文件中使用use strict;
使用linter
不添加压缩javascript库
类名使用驼峰命名
如果javascript代码需要全局运行，在website模块中声明一个if_dom_contains 方法

```
odoo.website.if_dom_contains('.jquery_class_selector', function () {
    /*your code here*/
});
```


将所有的class命名为o_<module_name>,如o_forum
避免使用id
使用bootstrap的class
用下划线+小写来命名


