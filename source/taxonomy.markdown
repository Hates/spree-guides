Taxonomy
--------

This guide covers the basics of Spree’s Taxonomy system. It will cover:\
\* basic ideas\
\* editing taxonomies in the admin interface

endprologue.

### Introduction

As of the 0.4.0 release, Spree now includes a new and improved method of
categorizing / structuring \
products which is known as Taxonomies. \
Taxonomies provide a simple, yet robust way of categorizing products by
enabling store administrators to define \
as many separate structures as needed.

When working with Taxomonies there are two keys terms to understand:

1.  Taxonomy – a hierarchical list which is made up of individual
    Taxons.
2.  Taxon – a single child node which exists at a given point within a
    Taxonomy. Each Taxon can contain many (or no) sub / child taxons.

Store administrators can define as many Taxonomies as required, and link
a product to multiple Taxons from each Taxonomy.\
Examples

The sample store contains two separate Taxonomies to help highlight
their flexibility:\
The Category taxonomy is the standard categorical product breakdown.

<shell>\
Category\
 \_ Clothing\
 | \_ Shirts\
 | \_ T-Shirts\
 \_ Bags\
 \_ Mugs\
</shell>

Brand taxonomy lists products associated with each brand.

<shell>\
Brand\
 \_ Ruby\
 \_ Ruby on Rails\
 \_ Apache\
</shell>

The “Ruby on Rails Baseball Jersey” product is linked to the following
two taxons:

<shell>\
 Category \>\> Clothing \>\> Shirts \>\> T-Shirts\
 Brands \>\> Ruby on Rails\
</shell>

So when a customer browses the site, the “Ruby on Rails Baseball Jersey”
will be appear in under each of these Taxons.\
Note: On the sample store, products “bubble” up a Taxonomy tree so the
“Ruby on Rails Baseball Jersey” will also appear under:

<shell>\
 Category \>\> Clothing \>\> Shirts \
 Category \>\> Clothing\
</shell>

This method is intended as an example of how to you can use Taxonomies
to structure your store. The choice of how to display products using
taxonomies is store specific, and the customer experience can be easily
changed to suit any filtering / categorizing set up that you might need.

### Creating Taxonomies

Log in to the admin screen. On the configuration page, find and click on
the taxonomies link.

Now, let’s create a taxonomy called “Computers” that contains several
brands. \
In the text field type “Computers” and click “create”. After you have
created “Computers”, you will see the page indicating “name” of the
taxonomy and its “tree”. To the right of the word “tree”, right click
with your mouse over into the word “Computers”. This will bring the
contextual menu and an option to create a “New Child”. Click into the
“New Child” contextual menu. This will bring the text field that will
appear at the bottom of the “Computers” word. Type into this new field
“Apple” and click “ok”. That’s it! We have created a new taxonomy for
Computers of which it has a child to it, as a brand, called “Apple”. You
can begin expanding this tree adding more computer brands.

### Modifying Taxonomies

Right-clicking on a taxonomy nodes yields options for editing, deleting,
and moving existing nodes, as well as for inserting new nodes.

You can also move taxonomy nodes within their hierachies via drag and
drop.

### Displaying Taxonomies in the Store

Briefly: Spree’s default sidebar allows navigation by taxons, and when
viewing products or taxons you will see breadcrumbs near the top of the
page which indicate the ‘position’ of the item in the taxon hierarchy.
