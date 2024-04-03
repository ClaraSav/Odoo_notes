## Vista Formulario Odoo

Version 11 - 12 - 14 - 15

Ejemplo sacado del módulo account de Odoo 14.
```xml
<record id="view_account_group_form" model="ir.ui.view">
    <field name="name">account.group.form</field>
    <field name="model">account.group</field>
    <field name="arch" type="xml">
        <form string="Account Group">
        <sheet>
            <group>
                <field name="name"/>
                <label for="code_prefix_start" string="Code Prefix"/>
                <div>
                    From <field name="code_prefix_start" class="oe_inline"/> to <field name="code_prefix_end" class="oe_inline"/>
                </div>
                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
            </group>
        </sheet>
        </form>
    </field>
</record>

```

Dentro de todas las vistas encontraremos el campo name que establecerá el nombre de la vista.

```xml
<field name="name">account.group.form</field>
```

En campo model contendrá el nombre del modelo 
```xml
<field name="model">account.group</field>
```

y el campo arch que contiene toda la estructura de la vista. Para una vista formulario se define una etiqueta ```<form>```

Las otras etiquetas sirven para organizar la vista, como se indica en la imagen. 
![form_view.png](..%2Fimg%2Fform_view.png)
