Searching for products
----------------------

This section explains Spree’s support for product search and selection.\
After reading it you should know:\
\* how the *Searcher* API works\
\* how to install and configure a searcher instance\
\* how to configure and use faceted search

This guide is aimed at developers.

endprologue.

### Overview

Searching for products is a key operation for commerce sites, where we
need both speed and flexibility.\
Spree supports this by providing an API that allows plug-in and
configuration of a variety of specialized,\
high-performance search platforms. Furthermore, the core team maintains
instances of this API for a few\
of the current top platforms, such as
[Solr](http://lucene.apache.org/solr/) and\
[Sphinx](http://www.sphinxsearch.com/).

The primary use of these platforms in Spree is to provide flexible
full-text searching of product\
details. Spree submits a query and then paginates the result.

Depending on the exact platform in use, Spree may also advise on
searches with alternative spellings,\
or support refinement or drilling down in the search result via\
[facets](http://en.wikipedia.org/wiki/Faceted_search).

### Searcher API

#### Configuration

You can set the current search implementation by assigning a suitable
object to\
*Spree::Config.searcher\_class*.\
If it is not set, the searcher defaults to an instance of the base
implementation.

#### Integration with Spree

The primary modification is to override the named scope for general
keyword lookup.\
Thus *Product.keywords+ will use the configured search implementation,
and return\
a new scope which limits its result to products which match the query.
\
One standard use of the*keywords+ scope is to implement the default
Spree search box.\
The method *retrieve\_products* (now in *lib/spree/search.rb*) handles
display of product\
lists based on a mix of keywords, taxon selections, product group(s) and
facets.\
It uses the searcher API to set up searches, then the actual search
occurs via (indirect)\
calls to the *keywords* scope, e.g. as a clause in a product group or in
response to\
presence of a keyword parameter in the query.\
Note that *retrieve\_products* is used to generate listings in the
*Product* and *Taxon*\
controllers: you’ll probably need to use it if generating similar
listings.

#### Standard interface

See the existing search implementations for concrete examples, e.g.\
the [base
implementation](https://github.com/spree/spree/blob/master/core/lib/spree/core/search/base.rb)\
or [spree-solr-search](https://github.com/romul/spree-solr-search).

##### *new*

The API is managed through a class, so if you are replacing the default
instance, you need to create\
a new instance of your search class.

##### *get\_products\_conditions\_for+
\
This is the main call. It’s purpose is to format the query appropriately and retrieve information from\
the search platform, then return it in the form of a named scope.\
Typically, you will retrieve a list of product ids*i1, i2, …+\
from the platform and produce the result *+ .
\
The format of input*query+ is not fixed, though you should expect to be able to handle a string or\
a list of strings.

##### *prepare+
\
This call is intended to set up the context for complex calls to the search platform, e.g. to allow\
for pagination or faceting. It is normally called in*retrieve\_products*, with the parameters\
from a product display query.
\
IMPORTANT: it seems that*prepare()+ should act more like a wrapper around restricted calls, and clear the (modified) properties after the important call - else subsequent calls WILL be affected by old (or a previous request’s) data.

##### *manage\_pagination*

This should return *true* if the search platform can manage its own
pagination of results.

##### *method\_missing*

The base implemementation defines this to provide easy access to the
properties stored with *prepare* or\
added during *get\_products\_conditions\_for*, such as the pagination
flag above (which is nil/absent for the\
base implementation).

##### The base implementation

The most notable point is the implementation of
*get\_products\_conditions\_for*: it just uses a named\
scope (*Product.name\_or\_description\_like\_any*, provided by
SearchLogic) to search for a substring in\
the name or description fields.

#### The Solr instance

[Solr](http://lucene.apache.org/solr/) is an industrial-scale search
platform, with many features and\
settings.

Roman Smirnov has produced an extension
[spree-solr-search](https://github.com/romul/spree-solr-search)\
for a direct connection into Spree. It contains a *searcher* instance,
and configures Solr for\
indexing on certain attributes of the *Product* model. There is also
code for declaring facets\
and for enabling drill-down by those facets (with the **facets+
partial).\
The *acts\_as\_solr* plugin is used to handle low-level communication.

\
h5. Installation
\
1. *script/extension install
git://github.com/romul/spree-solr-search.git* to install the extension\
1. Copy *solr\_search/config/solr.yml* to *RAILS\_ROOT/config/solr.yml*,
and edit if you need to change host names or port numbers.\
1. Check that you have all required gems/plugins, e.g. using *rake
gems:install*
\
You also need a running Solr server . There are several options for
this. For development and testing, we recommend\
Diego Carrion’s [jetty-solr](https://github.com/dcrec1/jetty-solr)
project . Once you have cloned the project, just run *./start* in the\
base directory to run a development server on port 8982. Other options
and configurations are given\
in the documentation. You can also start the server from your rails
project via\
*rake solr:start SOLR\_PATH=XYZ* ,\
and stop it with *rake solr:stop*.
\
Jetty-solr can be used for*production\_ use too, though the\
[Solr
documentation](http://wiki.apache.org/solr/FAQ#Solr_Comes_with_Jetty.2C_is_Jetty_the_recommended_Servlet_Container_to_use_when_running_Solr.3F)
advises exploring other options and experimenting with the\
various configuration parameters to get best performance in high-traffic
scenarios.
\
h5. How Solr is tied to Spree
\
The first step is to declare which ‘attributes’ of which models should
be indexed, and of these, which\
should be treated as facets. The attributes can be conventional model
attributes, or the names\
of methods which generate appropriate result strings. If non-string
values are needed, then indicate\
the type as done for*:price*.
\
<ruby>\
Product.class\_eval do\
 acts\_as\_solr :fields =\> ,\
 :facets =\> \
</ruby>
\
The second step is to format the query
in*get\_product\_conditions\_for*. The current code sets the\
appropriate sort order and injects taxon constraints, if relevant. In
particular, a request within\
a particular taxon is expanded to include all of the taxon’s
descendents. The final step is to\
set up the pagination.
\
NOTE: Some of the query parameters are derived from the information
registered in the*prepare()+ call.

##### Indexing and reindexing

Creating the index and updating it is done with the same command: *rake
solr:reindex*

The underlying plugin recognises several options, e.g. *BATCH=500* to
submit new products to the\
Solr server in batches of 500 (the default is currently 300).

##### Solr tuning and configuration

There are many settings to control the behaviour of Solr instances. See
the documentation and\
config files in Jetty-solr for basic information, or the official Solr
documentation.

### Facet support

Facets allow large search results to be filtered according to properties
a user may be interested in.\
For example, we can search for *rails* and then select the clothing
taxon(s) and a certain price range\
to end up with a more manageable number of products.

#### Faceting with Solr

Solr directly supports faceting, and can index declared facets alongside
the conventional data. You can\
include facet constraints in a Solr query, and can obtain information
about the available facets\
that can be used to drill down in a given query.

[Smirnov’s extension](https://github.com/romul/spree-solr-search) makes
it easy to incorporate facet\
functionality into Spree. By default it generates facets for the
following:

-   product brand (via a “brand” property)
-   colors of a product’s variants (via the variant options)
-   sizes of a product’s variants (via the variant options)
-   ‘primary’ taxons - the taxons to which the product is explicitly
    assigned (not the implicit descendants)
-   price range, in bands of 0-25, 25-50, 50-100,100-200,200 and over.

The extension also includes a basic partial for displaying and
requesting facet results, which you can\
include or replace in your own site theme.

WARNING: The price range code does not work at the moment.

NOTE: The description text of the price bands will need modification if
used in non-US locales. The text is held in *Spree::Config*.

\
h5. Declaration of facets
\
Facet declaration requires two steps. First, to declare the facet as an
indexable field of the\
product, and second, to declare that the field is a facet.
\
<ruby>\
Product.class\_eval do\
 acts\_as\_solr :fields =\> ,\
 :facets =\> \
</ruby>
\
Field names should be methods of the object being indexed. For the
default facets, we have methods\
which return a product’s brand , or return the colors of any variants .\
If you need other return types, they should be declared as shown
for*:price*.

To add a new facet based on some other property or option value, you
should just provide a method\
which generates the relevant information and include it in the lists
above. You could also override\
the provided methods if you need more specialized treatment of brand
etc.

If you have changed or added to the facets, then you will need to
[reindex the\
database](#indexingandreindexing).

##### Query processing

There are several options to include in the Solr query when using
facets, including which facet (if any)\
is being requested and limits on result size. The only part a developer
may usually want to change is\
the list of facet results being requested: this is currently set near
the top of file\
*solr\_search/lib/spree/search/solr.rb*.

Solr returns the requested facet information as a nested hash, and the
Solr extension converts this to a\
list of *Spree::Search::Facet* values, each containing a facet name and
a list of facet options. Each\
*Spree::Search::FacetOption* value contains an option string (eg a
specific color) and the count of\
matched products. This information can be used to generate links.

##### Display and selection of options

The partial *app/views/products/\_facets.html.erb* provides a basic
interface to the facets in a format\
suitable for inclusion in a sidebar, using the list of
*Spree::Search::Facet* values returned by the\
most recent query. Each non-empty facet is mapped to a title and
unordered list of links.\
A helper function *link\_to\_facet* will generate links which add a
facet option to the currently\
active constraints (and in turn, this can be decoded by the solr query
generation).

Note that you can pick several options from certain facets (based on
variants or taxons), eg you can\
select color ‘Red’ and on the resulting page, select ‘Green’. The effect
is to return only the products\
whose color values include both red and green, or more precisely, only
the products which have\
variants whose colour option is either red or green.

Technically speaking, this means that the standard behaviour with Solr
is to produce an *intersection*\
of constraints, not a *union*. So if you want to allow display of
products whose brand is A *or* B,\
then some custom programming will be required.
