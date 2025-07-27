Module Creation: Coffee Chain ERP
=================================

This section describes how to create a basic Odoo module called **Coffee Chain ERP**, used for managing coffee outlets.

Module Overview
---------------

- Name: Coffee Chain ERP
- Technical name: `coffee_chain_erp`
- Purpose: Manage coffee outlet branches and integrate with CRM and Sales modules.

Manifest (`__manifest__.py`)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `__manifest__.py` declares metadata, dependencies, and loaded files:

.. code-block:: python

    {
        'name': 'Coffee Chain ERP',
        'version': '1.0',
        'summary': 'Manage coffee outlets',
        'author': 'Nush Ojha',
        'category': 'Management',
        'depends': ['base', 'crm', 'sale'],
        'data': [
            'views/coffee_views.xml',
            'data/coffee_outlet_data.xml',
            'security/ir.model.access.csv'
        ],
        'installable': True,
        'application': True,
    }

Model: `coffee.outlet`
-----------------------

The core model that stores outlet details.

.. code-block:: python

    class CoffeeOutlet(models.Model):
        _name = 'coffee.outlet'
        _description = 'Coffee Outlet'

        name = fields.Char(string="Outlet Name", required=True)
        location = fields.Char(string="Location")
        manager = fields.Char(string="Manager")

XML Views
---------

The module defines both form and list views for managing outlets.

**List View**

.. code-block:: xml

    <record id="view_coffee_outlet_list" model="ir.ui.view">
        <field name="name">coffee.outlet.list</field>
        <field name="model">coffee.outlet</field>
        <field name="type">list</field>
        <field name="arch" type="xml">
            <list>
                <field name="name"/>
                <field name="location"/>
                <field name="manager"/>
            </list>
        </field>
    </record>

**Form View**

.. code-block:: xml

    <record id="view_coffee_outlet_form" model="ir.ui.view">
        <field name="name">coffee.outlet.form</field>
        <field name="model">coffee.outlet</field>
        <field name="type">form</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="location"/>
                        <field name="manager"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

Menus and Actions
-----------------

The module adds a main menu and submenu:

.. code-block:: xml

    <menuitem id="coffee_chain_root" name="Coffee Chain"/>
    <menuitem id="coffee_outlet_menu" name="Outlets"
              parent="coffee_chain_root"
              action="action_coffee_outlet"/>

    <record id="action_coffee_outlet" model="ir.actions.act_window">
        <field name="name">Outlets</field>
        <field name="res_model">coffee.outlet</field>
        <field name="view_mode">list,form</field>
    </record>

Sample Data
-----------

You can pre-load sample outlets:

.. code-block:: xml

    <record id="coffee_outlet_kathmandu" model="coffee.outlet">
        <field name="name">Kathmandu Branch</field>
        <field name="location">Kathmandu</field>
        <field name="manager">Ram Limbu</field>
    </record>

    <record id="coffee_outlet_pokhara" model="coffee.outlet">
        <field name="name">Pokhara Branch</field>
        <field name="location">Pokhara</field>
        <field name="manager">Sita Kumari</field>
    </record>

Access Control
--------------

Define who can access the outlet model:

.. code-block:: csv

    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    access_coffee_outlet_user,access.coffee.outlet.user,model_coffee_outlet,base.group_user,1,1,1,1

