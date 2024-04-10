# Alkona-Description-Odoo
Odoo - TUTORIAL
1. Создание связей One2many.
   - Реализация их связей.
   - Отображение связанной информации в форме.
        
   **1.1.** Мы хотим привязать одну модель к другой. Для этого нам нужно создать сами модели. Для каждой модели создается своя таблица.
     - Процесс создания модели заключается в создании класса в Python.
     ```python
     class TestMainModel():
       _name = 'my_main_model'
     ```   
     Располагаться он должен в папке **"{your_custom_module}/models"**    
        Для того, чтобы Odoo видел эту модель нужно создаеть еще один .py файл в той же директории, а именно   
       ```
       __init__.py
       ```
     В нем прописать:   
   ```python
   import . from {your_model_file_name}
   ```
     Описание атрибутов таблицы происходит путем создания полей этого класса [имя = тип(различные настройки, описание поля...)]
     
     - И так мы создали 2 модели и хотим их связать связью One2many.
     Нужно в основной модели создать поле, которое будет ссылаться на зависимую модель, в последствии через это поле мы будем получать связанные данные из зависимой модели. 
    Здесь мы указываем что поле будет иметь тип One2many.     Первый аргумент - принимает имя связанной модели, второй - поле из зависимой модели которое ссылается на основную модель. Третьим аргументом в данной случае указывается группа, юзеры которой имеют доступ к полю.

**Основная модель**
```python
from odoo import models, fields, api

class TestMainModel(models.Model):
    _name = 'my_main_model'
    .....
    c_name = fields.Char(string='Наименование', size=800, required=True, index='trigram')
    c_code = fields.Char(string='Код', size=50)
    f_method_vmp = fields.One2many('my_depended_model', 'f_vid', groups='tfoms-base.group_tfoms_user')
    .....
```
  - В зависимой таблице нужно создать поле с типом Mane2one, которое будет ссылаться на основную, через него мы так же можем получить информацию о главной модели.
  Это поле и используется в основной модели вторым аргументом.
  Так же в аргументах этого поля мы должны ссылаться на основную модель.    

**Зависимая модель**
```python
from odoo import models, fields, api

class MyDependedModel(models.Model):
    _name = 'my_depended_model'
    .....
    c_name = fields.Text(string='Наименование', required=True, index='trigram')
    c_diagnosis = fields.Text(string='Диагнозы')
    f_vid = fields.Many2one(string='Произвольное имя', comodel_name='my_main_model', ondelete='restrict', index='btree')
    .....
```
Аргументы передаются как именованные либо в порядке указания.

Теперь мы можем получить информацию в основной модели из зависимой по полю - **f_method_vmp**, и наоборот в зависимой модели по полю - **f_vid** можно получить значения из основной модели.

**1.2.**  Далее нам нужно создать представления для этих моделей.
Представления представляют собой файлы с расширением **.xml**
Распологаться они должны в **"{your_custom_module}/views"**    
Так же их нужно зарегистрировать в файле ```__manifest__.py``` который находится в папке вашего модуля.

```python
{
    'name': "Имя",
    'summary': "Краткое содержание",
    'description': """
Описание
    """,
    'author': "Имя Автора",
    'website': "Ссылка на ваш сайт",
    'category': 'категория',
    'version': '0.1',
    # any module necessary for this one to work correctly
    'depends': ['base'],
    'application': True,
    # always loaded
    'data': [
        .....
        'views/{your_view_name}.xml',
        .....
    ],
}
```
- Документация по определению представлений - https://doc.open-odoo.ru/developer/13.0/ru/reference/data.html
- Создаем сами представления.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
   <data>      
      <record id="cs_kind_vmp_tree" model="ir.ui.view" >
         <field name="name">cs.kind.vmp.tree</field>
         <field name="model">tfoms_dict.cs_kind_vmp</field>
         <field name="type">tree</field>
         <field name="arch" type="xml">
            <tree>
               <field name="c_code" />
               <field name="c_name" />
               <field name="f_group_vmp" />                
            </tree>               
         </field>
      </record>

      <record id="cs_kind_vmp_form" model="ir.ui.view" >
         <field name="name">cs.kind.vmp.form</field>
         <field name="model">tfoms_dict.cs_kind_vmp</field>
         <field name="type">form</field>
         <field name="arch" type="xml">
            <form>               
               <group>
                  <field name="c_code" />
                  <field name="c_name" />
                  <field name="f_group_vmp" />
               </group>
                  <notebook>             
                  <page string="Метод ВМП">                     
                     <field name='f_method_vmp'>                     
                        <tree limit="10">
                           <field name="c_name"/>
                           <field name="c_diagnosis"/>
                        </tree>
                     </field>              
                  </page>
               </notebook>                                        
            </form>
         </field>
      </record>
```
