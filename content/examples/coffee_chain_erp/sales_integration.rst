Sales Integration: Coffee Outlet
================================

This section describes how the Coffee Chain ERP module integrates with the Sales application.

Goal
----

- Link each sales order to a specific coffee outlet.
- Add fields for service type (Dine-in or Takeaway) and payment method (Cash, eSewa, etc.).
- Default the customer to “Walk-in Customer”.
- Pass outlet and payment method information from sales order to invoices.


Model Changes
-------------

Extend `sale.order` to add outlet and new fields:

.. code-block:: python

    from odoo import models, fields, api

    class SaleOrder(models.Model):
        _inherit = 'sale.order'

        outlet_id = fields.Many2one('coffee.outlet', string='Outlet', required=True)

        service_type = fields.Selection([
            ('dine_in', 'Dine-in'),
            ('takeaway', 'Takeaway'),
        ], string='Service Type', default='dine_in', required=True)

        payment_method_id = fields.Many2one(
            'account.payment.method',
            string='Payment Method',
            domain=[('payment_type', '=', 'inbound')]
        )

        def _prepare_invoice(self):
            invoice_vals = super()._prepare_invoice()
            invoice_vals.update({
                'outlet_id': self.outlet_id.id,
                'payment_method_id': self.payment_method_id.id,
            })
            return invoice_vals

        @api.model
        def default_get(self, fields_list):
            res = super(SaleOrder, self).default_get(fields_list)
            walkin_customer = self.env['res.partner'].search([('name', '=', 'Walk-in Customer')], limit=1)
            if walkin_customer:
                res['partner_id'] = walkin_customer.id
            cash_method = self.env['account.payment.method'].search([('name', '=', 'Cash')], limit=1)
            if cash_method:
                res['payment_method_id'] = cash_method.id
            return res

View Changes
------------

Extend the Sales Order form to add the `outlet_id`, `service_type`, and `payment_method_id` fields after the customer field:

.. code-block:: xml

    <record id="sale_order_form_inherit_outlet" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.outlet</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="outlet_id"/>
                <field name="service_type"/>
                <field name="payment_method_id"/>
            </xpath>
        </field>
    </record>

Notes
-----

- Each sales order must be linked to a specific coffee outlet.
- The `service_type` field allows choosing between Dine-in and Takeaway services.
- The payment method is linked to Accounting's inbound payment methods and defaults to "Cash" if available.
- The outlet and payment method are carried over to generated invoices for accurate financial tracking.