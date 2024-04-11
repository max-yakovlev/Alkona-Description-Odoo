# Alkona-Description-Odoo_One2many
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
- Документация по определению представлений - https://doc.open-odoo.ru/developer/13.0/ru/reference/views.html
- Создаем сами представления. View, menu, action можно прописать в одном файле, можно логически разделить по файлам. Главное все обернуть в <odoo>...</odoo>   
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo> <!-- Тут мы в одном файле создаем 2 вида представлений, Tree и Form. -->
   <data>      
      <record id="my_main_model_tree" model="ir.ui.view" >  
         <field name="name">my.main.model</field>
         <field name="model">my_main_model</field>
         <field name="type">tree</field> <!-- Представление типа Tree -->
         <field name="arch" type="xml">
            <tree>  <!-- Тут указываем с каким типом представления мы хотим работать -->
               <field name="c_code" />  <!-- Поля модели -->
               <field name="c_name" />
            </tree>               
         </field>
      </record>

      <record id="my_main_depended_form" model="ir.ui.view" >  
         <field name="name">my.main.depended</field>
         <field name="model">my_main_depended</field>
         <field name="type">form</field> <!-- Представление типа Form -->
         <field name="arch" type="xml">
            <form>   <!-- Тут указываем с каким типом представления мы хотим работать -->            
               <group> <!-- Разделение на подгруппы -->
                  <field name="c_code" />
                  <field name="c_name" />
               </group> 
                  <notebook>   <!-- Создаем набор вкладок -->                
                  <page string="Имя страницы">   <!-- Куда помещаем страницу с именем, можно определить другие страницы ниже для расширения -->                  
                     <field name='f_method_vmp'>     <!-- Собственно само поле ссылка, предоставляющее данные из зависимой модели -->                 
                        <tree limit="10">      <!-- Кол-во страницы отображаемых в списке на одной странице -->
                           <field name="c_name"/>
                           <field name="c_diagnosis"/>
                           <field name="f_vid" />
                        </tree>
                     </field>              
                  </page>
               </notebook>                                        
            </form>
         </field>
      </record>
   </date>
</odoo>
```
- Для того, чтобы все заработало нам нужно еще создать модель-действие = action, на которое опирается наша основная модель.
```xml
<odoo>
   <record id="my_main_model_action" model="ir.actions.act_window"> <!-- id - Произвольное имя, на него мы будем ссылаться вызывая этот Action, model - Выбираем системную модель, которая определяет ее действие  -->
         <field name="name">Виды ВМП</field> <!-- Имя action, которое будет мы видеть как ссылку -->
         <field name="res_model">my_main_model</field> <!-- Указываем нашу основную модель -->
         <field name="view_mode">tree,form</field> <!-- Перечисляем список форм отображения, Tree - List, Form - form -->
         <field name="limit" eval="10"/> <!-- Сколько записей будет отображаться на одной странице -->
   </record>
</odoo>
```
- Осталось только создать кнопку/ссылку, которая будет вызывать наш Action определенный выше, а тот в свою очередь ссылается на нашу основную модель вызывая ее и ее формы, и она уже отображает себя и связаные с собой данные
```xml
<odoo>
    <data>
        <menuitem name="Справочники" id="main_menu_root" groups="tfoms-base.group_tfoms_user" /> <!-- Наше основное меню/кнопка/ссылка, которое мы видим в UI. groups- группа юзеров которые видят это меню -->        
        <menuitem name="Виды ВМП" id="sub_menu_root" action="my_main_model_action" groups="tfoms-base.group_tfoms_user" parent="main_menu_root"/> <!-- Наше подменю кнопка/ссылка. Тут нужно указать наш Action определенный выше. Здесь она ссылкается на основную кнопку/ссылкую -->
    </data>
</odoo>

```
В итоге должны получить примерную картину

![Пример формы](https://github.com/max-yakovlev/Alkona-Description-Odoo/assets/165757267/69d39e52-f6e4-4e36-9e5f-87fe528b4966)
