Taxation
--------

Spree has an extremely flexible system for tax calculation. After
reading this guide you should have an understanding of the following:

-   Setting up tax categories and zones
-   Configuring sales tax and/or price inclusive taxes
-   Using alternative methods to compute tax

endprologue.

### Overview

The standard sales tax policies commonly found in the USA can be modeled
as well as Value Added Tax (VAT) which is commonly used in Europe. These
are not the only types of tax rules that you can model in Spree. Once
you obtain a sufficient understanding of the basic concepts you should
be able to model the tax rules of your country or jurisdiction.

WARNING: There were major changes to Spree taxation in the 1.0 release.
If you have an existing store on a prior version of Spree then you
should consult the [Version 1.0 Release
Notes](release_notes_1_0_0.html).

### Basic Concepts

#### Tax Categories

The Spree default is to treat everything as exempt from tax. In order
for a product to be considered taxable, it must belong to a tax
category. The tax category is a special concept that is specific to
taxation. It has nothing to do with the categories (taxonomies) that
users use to navigate through your store. The tax category is normally
never seen by the user so you could call it something generic like
“Taxable Goods.” If you wish to tax certain products at different rates,
however, then you will want to choose something more descriptive (ex.
“Clothing.”).

NOTE: It can be somewhat tedious to set the tax category for every
product. We’re currently exploring ways to make this simpler. If you are
importing inventory from another source you will likely be writing your
own custom Ruby program that automates this process.

#### Zones

You must also create a tax zone before your products can be taxed. A tax
zone is a geographical area where products are shipped to. If an order
is being shipped to somewhere within the zone, then it will be taxed at
the appropriate rate. You will need to create one zone for each
different geographic area you need to account for when determining tax.

##### Basic Examples

Let’s say you need to charge 5% tax for all items that ship to New York
and 6% on only clothing items that ship to Pennsylvania. This will mean
you need to construct two different zones. One zone containing just the
state of New York and another zone consistng of the single state of
Pennsylvania.

Here’s another hypothetical scenario. You would like to charge 10% tax
on all electronic items and 5% tax on everything else. This tax should
apply to all countries in the European Union (EU.) In this case you
would construct just a single zone consisting of all the countries in
the EU. The fact that you want to charge two different rates depending
on the type of good does not mean you need two zones.

INFO: Please see the [Zones Guide](zones.html) for more information on
constructing a zone.

##### Default Tax Zone

Spree also has the concept of a default tax zone. When a user is adding
items to their cart we do not yet know where the order will be shipped
to. In some cases we may want to estimate the tax for the order by
assuming the order falls within a particular tax zone.

Why might we want to do this? The primary use case for this is for
countries where there is already tax included in the price. In the EU,
for example, most products have a Value Added Tax (VAT) included in the
price. There are cases where it may be desirable to show the portion of
the product price that includes tax. In order to calculate this tax
amount we need to know the zone (and corresponding Tax Rate) that was
assumed in the price.

We may also reduce the order total by the tax amount if the order is
being shipped outside the tax jurisdiction. Again, this requires us to
know the zone assumed in making the orginal tax calculation so that the
tax amount can be backed out.

INFO: For more information on VAT and other similar scenarios see the
[Tax Included](#tax-included) section of this guide.

##### Shipping vs. Billing Address

Most tax jurisdictions base the tax on the shipping address of where the
order is being shipped to. So in these cases the shipping address is
used when determining the tax zone. Spree does, however, allow you to
use the billing address to determine the zone.

To determine tax based on billing address instead of shipping address
you will need to set the `tax_using_ship_address` preference to `false`.

INFO: See the [Preferences Guide](preferences.html) for more information
on setting preferences.

NOTE: `Zone.match` will return the zone that best matches the address
used for taxation. In the case of multiple matches, the closer match
will be used with State zone matches having priority over Country zone
matches.

#### Calculators

In order to charge tax in Spree you also need a `Calculator`. In most
cases you should be able to use Spree’s `DefaultTax` calculator. It is
suitable for both sales tax and price-inclusive tax scenarios.

NOTE: The `DefaultTax` calculator uses the item total (exclusive of
shipping) when computing sales tax.

It is also possible to code your own `Calculator` models in Spree if the
`DefaultTax` calcuator is not sufficient for your needs. You will need
to register the calculator with Spree in order to make it available in
the user interface when setting up your tax rates.

For reference, this is how Spree currently performs the registration:

<ruby filename="lib/spree/core/engine.rb">\
 initializer “spree.register.calculators” do |app|\
 \# snip\
 app.config.spree.calculators.tax\_rates =\
 [Spree::Calculator::DefaultTax]\
 end\
</ruby>

#### Tax Rates

Tax rates are set up through the admin interface (Configurations | Tax
Rates). A tax rate is essentially a percentage amount charged based on
the sales price. Tax rates also contain other important information.

-   Whether product prices are inlusive of this tax
-   The zone in which the order address must fall within
-   The tax category that a product must belong to in order to be
    considered taxable.

Spree will calculate tax based on the best matching zone for the order.
It’s also possible to have more than one applicable tax rate for a
single zone. In order for a tax rate to apply to a particular product,
that product must have a tax category that matches the tax category of
the tax rate.

### Tax Types

There are two basic types of tax that a store owner might need to
contend with. In the United States (and some other countries) store
owners sometimes need to charge what is known as “sales tax.” In the
European Union (EU) and other countries stores owners need to deal with
“tax inclusive” pricing which is often called Value Added Tax (VAT.)

NOTE: Most taxes can be considered one of these two types. For instance,
in Australia customers pay a Goods and Services Tax (GST.) This is
basically equivalent to VAT in Europe.

In some cases you may need to charge one type of tax for orders falling
within one zone and another type of tax for orders falling within a
different zone. There are even some rare situations where need to charge
both types of tax in the same zone. Spree supports all of these
scenarios.

#### Sales Tax

Sales tax is the default tax type for any tax rate in Spree. It’s the
simplest type of tax to calculate and it is the method by which your tax
will be determined assuming you leave the “Included in Price” checkbox
unchecked.

Let’s take an example of a 5% tax on clothing items for a zone that
covers all of North America. If the customer purchases a single clothing
item for \$17.99 and they live in the United States (which is within the
North America zone we defined) they are eligible to pay sales tax.

The sales tax calculation is \$17.99 x 0.05 for a total tax of \$0.90.
Note that in the case of sales tax the tax is added to the item total an
adjustment.

![Sales Tax Applied](images/taxes/sales_tax_applied.png "Sales Tax Applied")

INFO: See the [Adjustments Guide](adjustments.html) if you need more
information on adjustments.

Now let’s change the quantity on that clothing item from 1 to 2. The tax
is now \$35.98 x 0.05 for a total of \$1.80.

![Sales Tax Applied](images/taxes/sales_tax_applied2.png "Sales Tax Applied")

Now let’s add a coffee mug that costs \$13.99 to the order. Since the
coffe mug is not clothing it’s not taxable. The tax total therefore
should not change (\$35.98 x 0.05)

![Sales Tax Applied](images/taxes/sales_tax_applied3.png "Sales Tax Applied")

Finally, if we change the order address to Ireland (which is outside of
the tax zone) we see that sales tax is no longer applied.

![Sales Tax Not Applied](images/taxes/sales_tax_not_applied.png "Sales Tax Not Applied")

#### Tax Included

Many jurisdictions have what is commonly referred to as a Value Added
Tax (VAT.) In these cases the tax is typically applied to the price.
This means that prices for items are “inclusive of tax” and no
additional tax needs to be applied during checkout.

In the case of tax inclusive pricing the store owner should enter all
prices inclusive of tax. This makes it easy for Spree to display the tax
inclusive price since it won’t have to be constantly calculated on the
fly.

NOTE: Keep in mind that each order records the price a customer paid
(including the tax) as part of the line item record. This means you
don’t have to worry about changing prices or tax rates affecting older
orders.

If you are going designate one or more tax rates as being included in
the price then you must first set up a default tax zone. Spree will not
allow you to proceed without performing this step. This default zone is
needed in cases where a customer might not be eligible to pay the tax.

INFO: You must choose a default tax zone if you are going to mark at a
tax rate as being included in your product prices.

WARNING: Prior versions of Spree handled VAT differently. If you are
upgrading to version 1.0 or higher you should make sure your prices
include tax.

When tax is included in the price there is no order adjustment needed
(unlike the sales tax case.) Stores are, however, typically interested
in showing the amount of tax the user paid. These totals are for
informational purposes only and do not affect the order total.

Let’s start by looking at an example where there is a 5% included on all
products and it’s included in the price. We’ll further assume that this
tax should only apply to orders within the United Kingdom (UK).

In the case where the order address is within the UK and we purchase a
single clothing item for £17.99 we see an order total of £17.99. We also
see an additional line included for the customer’s reference that shows
they paid a total of £0.90 in tax.

![VAT tax breakout](images/taxes/vat_tax_breakout.png "VAT tax breakout")

INFO: Notice how the tax is included below the order total. This is
because the tax line is for informational purposes only and does not
affect the actual total of the order.

NOTE: These informational totals are considered adjustments to the
`LineItem` class. This allows us to preserve them for reporting purposes
and to avoid including them in the `Order#adjustment_total`.

Now let’s increase the quantity on the item from 1 to 2. The order total
changes to £35.98 with a tax total of £1.80.

![VAT tax breakout](images/taxes/vat_tax_breakout2.png "VAT tax breakout")

Next we’ll add a different clothing item costing £19.99 to our order.
The order total increases to £55.97 but also notice that there is still
just a single item for the clothing tax showing £2.80. Since both items
are clothing and taxed at the same rate so they can be reduced to a
single total.

![VAT tax breakout](images/taxes/vat_tax_breakout3.png "VAT tax breakout")

Now let’s assume an additional tax rate of 10% on consumer electronics.
When we add a product of this category to our order with a price of
£16.99 we will now also see a second tax total of £1.70.

![VAT tax breakout](images/taxes/vat_tax_breakout4.png "VAT tax breakout")

Finally, let’s see what happens when we change our order to have an
address in the United States and therefore not included in the UK tax
zone we have defined. In this case Spree will create two negative
adjustments for £2.80 and £1.70 which effectively reduces the order
total to £68.46.

![VAT tax refund](images/taxes/vat_tax_refund.png "VAT tax refund")

### Additional Examples

WARNING: All of the examples in this guide are meant to be used for
illustrative purposes. They are not meant to be used as definitive
intepretations of tax law. You should consult your accountant or
attorney for guidance on how much tax to collect and under what
circumstances.

#### Quebec Sales Tax (QST)

In Quebec, Canada there are two different tax rates to consider. There
is the Goods and Services Tax (GST) which iscalculated at 5% of the
sales price. There is also the Quebec Sales Tax of 9.5% on the sale
price including GST.

Since Spree does not currently support the notion of compound tax rates
you can calculate a combined sales tax amount instead. In this case you
can create a single sales tax rate of 14.975%. This approach is allowed
[according to the Quebec
government](http://www.revenuquebec.ca/en/entreprise/taxes/tvq_tps/calcul-taxes.aspx)
as long as you label the tax simply as “QST” and do not display the
percentage used in the calculation as part of the label.
