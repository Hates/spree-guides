Products and Variants
---------------------

This guide covers products and variants and discusses how they are
managed in the administration interface as well as programmatically.

After reading it you should know:\
\* what products and variants are and how they work together\
\* how to add new products and variants via the administration
interface\
\* how to configure and use options to distinguish variants\
\* how to programmatically create and manage products and variants

This guide does not cover:\
\* [managing stock/inventory](inventory.html)\
\* [categorizing/organizing](taxonomy.html)

endprologue.

### Overview

Every store has something to sell. In Spree, that <i>something</i> is a
product at the highest level. Sometimes products come in different
colors and sizes. We refer to those as variants. Neither of these models
represent actual stock or inventory in your store (which is
InventoryUnit model). Together, Products and Variants describe what is
for sale.

#### What is a Product?

A Product describes something that is for sale, be it a physical item
one ships, or instantly downloadable goods, or a service that is neither
shipped nor downloaded. These are all collectively known as “Products”
in the Spree world. Products can have variations such as different
colors, styles, sizes, and sometimes a combination of two or more such
varying factors. Sometimes it’s difficult for new shop administrators to
determine what should be entered as a product and what should be keyed
as a variation of a product. The best way to decide this is to ask
yourself, “What are my unique goods irrespective of variations?” A
Product in the spree store should represent something conceptually
unique. A product has these key attributes

  Attribute                           Purpose
  ----------------------------------- -------------------------------------------------------------------------------------------------------------------
  Name                                short name for the product
  Description                         The most elegant, poetic turn of phrase for describing your product’s benefits and features to your site visitors
  Permalink                           An SEO slug based off the product name that is placed into the URL for the product
  “Tax Category”:taxation.html        How the product is classified for the purpose of applying taxes at purchasing time
  “Shipping Category”:shipping.html   How the product should be handled with regards to shipping and handling at purchasing time
  Available On                        The first date the product becomes available for sale online in your shop
  Deleted At                          The date the product is no longer available for sale in the store
  Meta Description                    A description targeted at search engine for search engine optimization (SEO)
  Meta Keywords                       Several words and short phrases separated by comma, also targeted at search engines

A product may be withdrawn from sale by marking it as ‘deleted’ in the
product listings admin page. Details are still kept on the system for
reference, but will not be shown on the store pages. A similar mechanism
exists for variants.

#### What is a Variant?

Products that can have different sizes, styles, colors, and so on, are
usually keyed into Spree as variations of the product and these
variations are represented by the Variant model in Spree. A Product can
have zero or more Variants. A Variant has the following attributes that
further define a Product:

  Attribute    Purpose
  ------------ ------------------------------------------------------------------------------------------------------------------------------------------------
  SKU          Short for Stock-Keeping Unit. An external/human supplied and managed unique identifier for a specific, stockable product item that can be sold
  Price        How much to charge a customer for the product when it is placed into the shopping cart
  Weight       The shipping weight of the product, entered in the units your shipping calculator requires
  Height       The height of the shipping container for the product in the units your shipping calculator requires
  Width        The width of the shipping container for the product in the units your shipping calculator requires
  Depth        The width of the shipping container for the product in the units your shipping calculator requires
  Deleted At   The date the variant is no longer available for sale in the online store

When Variants are associated with a Product, they are distinguished from
each other by their Option Values, which must a unique combination of
the Option Types that have been associated with a given Product. Option
Types and Option Values are discussed in more detailed below.

#### The Master Variant

You may be asking yourself, “If the Product model doesn’t have fields
for price, weight, height, etc., then how do we associate those values
to a product that has no variants?”

The answer is through the use of a “Master Variant,” which is generally
<i>tucked away</i> as far as many site administrators are concerned.
This special variant record is always created with each new product
record, so there is a direct one-to-one relationship between a product
and a variant with the *is\_master* flag set to true.

NOTE: You only need to be cognizant of the master variant if you begin
to manage products and inventory programmatically. The administration
interface keeps all of this nitty-gritty transparent to the end-user. If
you’re looking to get your store set up quickly through the
administration interface, feel free to skip to the next section as it is
not necessary to know how the master variants work to use the
administration interface.

The master variant will not show up in the *Product.variants* array and
plays no role in *Product.has\_variants?* For all intents and purposes,
the master variant is transparent and you can access all the fields of
the master variant instance as though it were a part of the Product
model itself. That is,

*Product.price  Product.master.price =\> true+\<br/\>
  +Product.sku  Product.master.sku =\> true*

When a Product has no option types, values, or variants assigned to it,
then the product’s price, SKU, physical size, weight, and inventory
units are all managed through this Master Variant. As soon as the first
variant with option values is assigned to a product, the *on\_hand* of
the “master variant” is set to zero and henceforth, all inventory levels
are managed within each Variant of the Product.

The other thing to note about the Master Variant: When a new Variant is
associated with a Product and the *Variant.price* field is left blank,
the *Product.master.price* will be assigned as the new Variant’s price,
thus reducing repetitive keying during data entry.

#### Inventory Units

Inventory Units are the means of managing the stock levels in Spree.
When you set the “On Hand” value in the administration interface, you’re
causing Inventory Units to be adjusted up or down to match the desired
“On Hand” inventory level for the given Product or Variant. Inventory
Units are tied directly to the Variant model in all cases. When a
Product has no Variants, the Inventory Units are associated with the
Master Variant.

NOTE: Once you add variants to the Product, you will no longer be able
to set “On Hand” at the product level. Instead, set the “On Hand” on
each individual Variant’s data entry screen. When customizing your Spree
views, you should be able to determine special behavior for Products
with variants vs. without variants simply and easily through
*Product.has\_variants?* and iterating through *Product.variants* array.

TIP: The first variant added to the product will clear out “On Hand”
Inventory Units for the Product itself.

### Using the Administration Interface

In the Spree Administration interface, Products, Variants, Option Types,
Properties, and Prototypes are all managed under the Products Tab and
this guide will walk you through all aspects of managing your Products
under this tab.

NOTE: All topics in this section assume you are logged into the Spree
administration interface and have highlighted the “Products” tab.

#### Upfront Considerations

The Product and Variant models were kept intentionally minimal,
providing attributes that are generally needed to drive an uncluttered
store-front and feed into line items on an order (such as price, weight,
shipping size, etc.). But fear not for Spree offers many ways to extend
your products and make them quite rich. Here are the various up-front
considerations for setting up your new store:

-   How will your products be shipped and taxed? It is best to establish
    your Shipping Categories and Tax Categories before keying in your
    products so that you can choose appropriate values for these fields
    as you populate your store with new products.

-   If your products have additional defining properties (such as
    length, material, gender, etc.) that you wish to manage outside the
    description of the product, you should set up Product Properties.

-   If you have products that can vary by size, color, or other
    attributes that should be shopper selectable, then you should set up
    Option Types rather than Properties for the Product. Option Types
    then allow you to add Variants to the products with specific Option
    Value combinations (XL, red, etc.).

-   If you wish to speed up the repetitive task of adding products with
    the same extended properties or option types, then set up Product
    Prototypes, which you can then select from a drop-down list when
    adding a new Product to quickly get a new Product with all those
    extended Product Property fields and Option Types available.

The above are the common up-front considerations you should deal with
before keying in products and variants as the entry forms will require
the above “infrastructure” to be declared before you can fill in data
for Product Properties or associate variants to products.

#### Properties

Properties allow you to extend the Product model with additional
attributes that helps the store administrator to consistently describe
all details about the product such as length, material, gender, brand,
and so on. Properties enrich the Product Model, but does not provide
selectable options to the shopper to pick from when choosing a Product
to purchase. Since certain Properties may apply to many products, Spree
manages a global set of properties.

NOTE: In Spree parlance, a “Property” that is associated with a specific
Product is then called a “Product Property.” So, watch out for this
subtle distinction in the documentation. Properties have not been
associated with a given Product, yet, whereas Product Properties have!

To add a new Property, click on “Properties” sub-tab, then click the
“New Property” button. Here, you will be able to add a name (the
internal label) and presentation (what is presented to the shopper
on-screen).

NOTE: Properties are displayed after the description on the product’s
fly-page.

TIP: Prototypes can be set up to include sets of these properties when a
product is first created.

Once you have defined a Property, you can then associate that Property
with a given Product. You can specify which keys you want to associate
and then give corresponding values via the “Product Properties” menu in
the sidebar. If you selected a prototype when creating the product,
certain keys may already be set up. Otherwise, type in the (internal)
name of an existing property and a value.

NOTE: The Option Type must already exist - it is not automatically
created.

INFO: Product Properties are implemented using the
Entity/Attribute/Value (EAV) design pattern. If your implementation
needs require new fields to be added directly to the Product or Variant
tables, then see the [extension guide](/extensions.html) guide for more
information on easily extending the Spree core system.

#### Product Option Types

Before one can begin associating Variants with Products, Option Types
must be defined and the administration interface has a sub-tab called
“Option Types” for this. When you click on the “New Option Type” button,
you are presented with the entry form for adding an Option Type.

  Attribute      Purpose
  -------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name           A short, internal name (not displayed to shoppers) that easily distinguishes the option type. This is useful if you have options that may have similar values, but are clearly different in some meaningful aspect (such as “shirt color” vs. “mug colors”)
  Presentation   The label that is displayed in the online storefront to shoppers

##### Option Values

Each Option Type should then have one ore more Option Values assigned to
it. This is essentially the choices themselves (e.g. Red, Green, Blue)
that characterize the given Option Type. Option Values define or
characterize a variant, and a variant must have one value for each
“option type” that has been associated with the Product. For example,
where size and color options have been set up, one variant can be
‘Small’ and ‘Red’, or another could be ‘Large’ and ‘Green’.

To create a variant, you just give a (not yet used) combination of the
product’s option types, and enter any variant-specific information. See
the [inventory guide](inventory.html) for information on stock control
issues.

#### Product Prototypes

Spree provides prototypes as a kind of template for related products.
Selecting a prototype at the first stage of product creation associates
the new product with certain properties and option types. This is a
useful time-saving measure and helps get consistency within groups of
similar products. You can define new prototypes using the sub-tab under
the main Products menu, where you name a prototype and select the
properties and option types to use.

#### Images

Images can be associated with a either a product or any of it’s
variants. When uploading an image for products that contain variants a
drop-down menu allows you to select which object the image applies to.
Note if a product does not contain any variants then the drop-down is
not displayed to ensure that basic implementations are not cluttered
with unnecessary administration options.

Spree automatically handles creation and storage of several size
versions of each image (via the Paperclip plugin). The default is to
create four styles according to the ImageMagick spec.

<shell>\
:styles =\> {\
 :mini =\> ‘48x48\>’,\
 :small =\> ‘100x100\>’,\
 :product =\> ‘240x240\>’,\
 :large =\> ‘600x600\>’\
 }\
</shell>

##### Configuration

From the admin there are a variety of configuration options that can be
set relating to images:

-   Attachment Path - Where the images are stored on the filesystem
-   Attachment URL - Where the images are accessible through the site
-   Paperclip Styles - Override the styles above with your own sizes or
    names
-   Amazon S3 Configuration

### Going Deeper

For those of you looking to do more advanced handling of Products,
Variants, and Inventory beyond what the basic views and store
administration interface provides, this section is for you and covers
more intricate details about the Spree models and the ground rules for
programmatically handling and managing the products, variants, and
inventory units.

#### The Ground Rules

##### Product

-   Products may or may not have variants.
-   Every product has one master variant, which stores price and sku,
    size and weight, etc.
-   The master variant attributes are directly accessible on the Product
    model (via delegation)

##### Master Variant

-   The master variant does not have option values associated with it.
-   Price, SKU, size, weight, etc. are all delegated to the master
    variant.
-   Contains on\_hand inventory levels only when there are no variants
    for the product.

##### Variants

-   All variants can access the product properties directly (via reverse
    delegation).
-   Inventory units are tied to Variant.
-   The master variant can have inventory units, but not option values.
-   All other variants have option values and may have inventory units.
-   Sum of on\_hand each variant’s inventory level determine “on\_hand”
    level for the product.

#### The Product Model

The Product model has several methods that simplify the logic used in
views and controllers for retrieving, displaying, and making other
decisions about how a product is presented or purchased in the store
front.

##### has\_variants?

Returns true if the product has any variants (the master variant is not
a member of the variants array)

##### on\_hand

Returns the number of inventory units “on\_hand” for this product. If
the product has variants, then\
*on\_hand* is the sum of each variant’s inventory level.

##### on\_hand=(new\_level)

Adjusts the “on\_hand” inventory level for the product up or down to
match the given new\_level. Once a product has variants, this method is
disabled as on\_hand must then be set at each variant rather than at the
product level.

##### has\_stock?

Returns true if there are inventory units (any variant) with “on\_hand”
state for this product

##### product\_price helper

The price for a product or variant can be obtained using
*product\_price* helper method. By default, this returns a string
formatted with the locale’s currency settings, but this can be
overridden with an option: *product\_price+
\
h5. Product images
\
In views, you can generate an image’s URL file
with*image.attachment.url(:product)+, i.e. using the name of a declared
image format as an argument.

#### The Variant Model

##### on\_hand

Returns number of inventory units for a given variant that is in the
“on\_hand” state.

##### on\_hand=(new\_level)

Adjusts the inventory units to match the given new level. If inventory
is increased and there are currently inventory units for the given
variant in back\_ordered state, then the “back\_ordered” inventory units
are flagged as “sold” and the balance of the new inventory level is then
added as “on\_hand.”

##### on\_backorder

Returns number of units currently on backorder for this variant.

##### in\_stock?

Returns true if at least one inventory unit of this variant is
“on\_hand”

##### available?

Returns true if this variant is allowed to be placed on a new order.

### Further Reading

NOTE: The [migration section](migration.html) of the guides gives an
overview on how to import data programmatically into Spree.

[Steph Skardal](https://github.com/stephskardal) has produced a useful
blog post on\
[product
optioning](http://blog.endpoint.com/2009/12/rails-ecommerce-product-optioning-in.html).\
This discusses how the variant option representation works and how she
used it to build an\
extension for enhanced product option selection.\
The blog post appeared in the [End Point
Blog](http://blog.endpoint.com/).
