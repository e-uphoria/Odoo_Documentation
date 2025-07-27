CRM Integration: Coffee Outlet
==============================

This section explains how the Coffee Chain ERP module integrates with Odoo's CRM app.

Goal
----

- Automatically create a CRM Lead (Opportunity) when a new Coffee Outlet is added.
- Link the outlet to an existing partner (customer).
- Display outlet information in CRM Lead form.

Model Changes
-------------

Extend `coffee.outlet` to reference CRM and partner models:

.. code-block:: python

    customer_id = fields.Many2one('res.partner', string='Customer')
    lead_id = fields.Many2one('crm.lead', string='Related Lead')

Auto-create CRM Lead on Outlet Creation
---------------------------------------

Override the `create()` method in `coffee.outlet`:

.. code-block:: python

    @api.model
    def create(self, vals):
        outlet = super().create(vals)
        lead_vals = {
            'name': f'Opportunity for {outlet.name}',
            'partner_id': outlet.customer_id.id or False,
            'coffee_outlet_id': outlet.id,
            'type': 'opportunity',
        }
        lead = self.env['crm.lead'].create(lead_vals)
        outlet.lead_id = lead.id
        return outlet

Extend `crm.lead` to link back to outlet(s):

.. code-block:: python

    class CrmLead(models.Model):
        _inherit = 'crm.lead'

        coffee_outlet_id = fields.Many2one('coffee.outlet', string='Coffee Outlet')
        coffee_outlet_ids = fields.One2many('coffee.outlet', 'lead_id', string='Coffee Outlets')

View Integration
----------------

Update the CRM Lead form view to include outlet selection:

.. code-block:: xml

    <record id="view_crm_lead_form_inherit_coffee" model="ir.ui.view">
        <field name="name">crm.lead.form.inherit.coffee</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="coffee_outlet_id"/>
            </field>
        </field>
    </record>

Partner Form Extension
----------------------

You can view all outlets linked to a customer:

.. code-block:: python

    class ResPartner(models.Model):
        _inherit = 'res.partner'

        coffee_outlet_ids = fields.One2many('coffee.outlet', 'customer_id', string="Coffee Outlets")

In the partner form view:

.. code-block:: xml

    <record id="view_res_partner_form_inherit_coffee" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.coffee</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <notebook position="inside">
                <page string="Coffee Outlets">
                    <field name="coffee_outlet_ids">
                        <list>
                            <field name="name"/>
                            <field name="location"/>
                        </list>
                    </field>
                </page>
            </notebook>
        </field>
    </record>
