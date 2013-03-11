Security
--------

This guide covers basic issues related to security and data privacy as
it pertains to Spree. After reading it you should know:\
\* The mechanism by which Spree performs authentication.\
\* How Spree deals with role-based permissions for various resources.\
\* How to modify the default authentication and authorization schemes.\
\* The Spree approach to safeguarding credit card and other sensitive
data.

endprologue.

### General

Proper application design, intelligent programming, and secure
infrastructure are all essential in creating a secure e-commerce store
using any software (Spree included.) The Spree team has done its best to
provide you with the tools to create a secure and profitable web
presence but it is up to you to take these tools and put them in good
practice. We highly recommend reading and understanding the [Rails
Security Guide](http://guides.rubyonrails.org/security.html) and to keep
up with the [Ruby on Rails Security
Project](http://www.rorsecurity.info/).

### Reporting Security Issues

Please do not announce potential security vulnerabilities in public. We
have a [dedicated email address](mailto:security@spreecommerce.com) for
reporting these concerns. We will work quickly to determine the severity
of the issue and provide a fix for the appropriate versions.

### Authentication

If you install spree\_auth\_devise when setting up your app, we use a
third party authentication library for Ruby known as
[Devise](https://github.com/plataformatec/devise). This library provides
a host of useful functionality that is in turn available to Spree,
including the following features:

-   Authentication
-   Strong password encryption (with the ability to specify your own
    algorithms)
-   “Remember Me” cookies
-   “Forgot my password” emails
-   Token-based access (for REST API)

#### Devise Configuration

NOTE: A default Spree install comes with the
[spree\_auth\_devise](https://github.com/spree/spree_auth_devise) gem,
which provides authentication for Spree using Devise. This section of
the guide covers the default setup. If you’re using your own
authentication, please consult the manual for that authentication
engine.

We have configure Devise to handle only what is needed to authenticate
with a Spree site. The following details cover the default
configurations:

-   Passwords are stored in the database encrypted with the salt.
-   User authentication is done through the database query.
-   User registration is enabled and the users login is available
    immediately (no validation emails).
-   There is a remember me and password recovery tool built in and
    enabled through Devise.
-   *http\_auth\_basic* is enable to accommodate the API query
    interface. (See below for security notes relating to the API)

INFO: These configurations represent a reasonable starting point for a
typical e-commerce site. Devise can be configured extensively to allow
for a different feature set but that is currently beyond the scope of
this document. Developers are encouraged to visit the Devise wiki for
more details.

#### REST API

The REST API behaves slightly differently that a standard user. First,
an admin has to create the access key before any user can query the REST
API, this includes generating the key for the admin him/herself.
Obviously it is up to you to communicate that key. This is a 20
character key that is passed through the HTTP\_AUTHENTICATION header
(See the [documentation](rest.html) for how to authenticate with the
REST API). As an added measure this authentication has to occur on every
request made through the REST API as no session or cookies are created
or stored for the REST API.

### Authorization

Spree uses the excellent [CanCan](https://github.com/ryanb/cancan) gem
to provide authorization services. If you are unfamiliar with it, you
should take a look at Ryan Bates’ [excellent screen
cast](http://railscasts.com/episodes/192-authorization-with-cancan) on
the topic (or read the [transcribed
version](http://asciicasts.com/episodes/192-authorization-with-cancan.))
A detailed explanation of CanCan is beyond the scope of this guide.

#### Default Rules

The follow Spree source code is taken from *ability.rb* and provides
some insight into the default authorization rules:

<ruby>\
if user.has\_role? ‘admin’\
 can :manage, :all\
else\
 \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\
 can :read, User do |resource|\
 resource  user
  end
  can :update, User do |resource|
    resource  user\
 end\
 can :create, User\
 \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\
 can :read, Order do |order, token|\
 order.user  user || order.token && token  order.token\
 end\
 can :update, Order do |order, token|\
 order.user  user || order.token && token  order.token\
 end\
 can :create, Order\
 \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\
 can :read, Product\
 can :index, Product\
 \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\
 can :read, Taxon\
 can :index, Taxon\
 \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\
end\
</ruby>

The above rule set has the following practical effects for Spree users

-   Admin role can access anything (the rest of the rules are ignored)
-   Anyone can create a *User*, only the user associated with an account
    can perform read or update operations
-   Anyone can create an *Order*, only the user associated with the
    order can perform read or update operations
-   Anyone can read product pages and look at lists of *Products*
    (including search operations)
-   Anyone can read or view a list of *Taxons*.

#### Enforcing the Rules

CanCan is only effective in enforcing authorization rules if it’s asked.
In other words, if the source code does not check permissions there is
no way to deny access based on those permissions. This is generally
handled by adding the appropriate code to your Rails controllers.

#### Custom Authorization Rules

We have modified the original CanCan concept to make it easier for
extension developers and end users to add their own custom authorization
rules. For instance, if you have an “artwork extension” that allows
users to attach custom artwork to an order, you will need to add rules
so that they have permissions to do so.

The trick to adding custom authorization rules is to add an
*AbilityDecorator* to your extension and then to register these
abilities. The following code is an example of how to restrict access so
that only the owner of the artwork can update it or to view it.

<ruby>\
 class AbilityDecorator\
 include CanCan::Ability

def initialize(user)\
 can :read, Artwork do |artwork|\
 artwork.order && artwork.order.user  user
      end
      can :update, Artwork do |artwork|
        artwork.order && artwork.order.user  user\
 end\
 end\
 end

Ability.register\_ability(AbilityDecorator)\
</ruby>

#### Tokenized Permissions

There are situations where it may be desirable to restrict access to a
particular resource without requiring a user to authenticate in order to
have that access. Spree allows so-called “guest checkouts” where users
just supply an email address and they’re not required to create an
account. In these cases you still want to restrict access to that order
so only the original customer can see it. The solution is to use a
“tokenized” URL.

<shell>\
http://example.com/orders/token/aidik313dsfs49d\
</shell>

Spree provides a *TokenizedPermission* model used to grant access to
various resources through a secure token. This model works in
conjunction with the *Spree::TokenResource* module which can be used to
add tokenized access functionality to any Spree resource.

<ruby>\
module Spree::TokenResource

module ClassMethods\
 def token\_resource\
 has\_one :tokenized\_permission, :as =\> :permissable\
 delegate :token, :to =\> :tokenized\_permission, :allow\_nil =\> true\
 after\_create :create\_token\
 end\
 end

module InstanceMethods\
 def create\_token\
 create\_tokenized\_permission(:token =\>
ActiveSupport::SecureRandom::hex(8))\
 token\
 end\
 end

def self.included(receiver)\
 receiver.extend ClassMethods\
 receiver.send :include, InstanceMethods\
 end

end\
</ruby>

The *Order* model is one such model in Spree where this interface is
already in use. The following code snippet shows how to add this
functionality through the use of the *token\_resource* declaration:

<ruby>\
Order.class\_eval do\
 token\_resource\
end\
</ruby>

If we examine the default CanCan permissions for *Order* we can see how
tokens can be used to grant access in cases where the user is not
authenticated.

<ruby>\
can :read, Order do |order, token|\
 order.user  user || order.token && token  order.token\
end\
can :update, Order do |order, token|\
 order.user  user || order.token && token  order.token\
end\
can :create, Order\
</ruby>

This configuration states that in order to read or update an order, you
must be either authenticated as the correct user, or supply the correct
authorizing token.

The final step is to ensure that the token is passed to CanCan when the
authorization is performed.

<ruby>\
authorize! action, resource, session[:access\_token]\
</ruby>

Of course this also assume that the token has been stored in the
session. Generally this can be achieved with a route that maps the token
to the correct parameter

<shell>\
match ‘/orders/:id/token/:token’ =\> ‘orders\#show’, :via =\> :get, \
:as =\> :token\_order\
</shell>

This is followed by a call to store the token in the session for
possible future access.

<shell>\
session[:access\_token] ||= params[:token]\
</shell>

### Credit Card Data

#### PCI Compliance

All store owner wishing to process credit card transactions should be
familiar with [PCI
Compliance](http://en.wikipedia.org/wiki/Pci_compliance). Spree makes
absolutely no warranty regarding PCI compliance (or anything else for
that matter - see the [LICENSE](http://spreecommerce.com/license) for
details.) We do, however, follow common sense security practices in
handling credit card data.

#### Transmit Exactly Once

Spree uses extreme caution in its handling of credit cards. In
production mode, credit card data is transmitted to Spree via SSL. The
data is immediately relayed to your chosen payment gateway and then
discarded. The credit card data is never stored in the database (not
even temporarily) and it exists in memory on the server for only a
fraction of a second before it is discarded.

INFO: Spree does store the last four digits of the credit card and the
expiration month and date. You could easily customize Spree further if
you wanted and opt out of storing even that little bit of information.

#### Payment Profiles

Spree also supports the use of “payment profiles.” This means that you
can “store” a customers credit card information in your database
securely. More precisely you store a “token” that allows you to use the
credit card again. The credit card gateway is actually the place where
the credit card is stored. Spree ends up storing a token that can be
used to authorize new charges on that same card without having to store
sensitive credit card details.

NOTE: Spree has out of the box support for [Authorize.net
CIM](http://www.authorize.net/solutions/merchantsolutions/merchantservices/cim/)
payment profiles.

#### Other Options

There are also third-party extensions for Paypal’s [Express
Checkout](https://merchant.paypal.com/cgi-bin/marketingweb?cmd=_render-content&content_ID=merchant/express_checkout)
(formerly called Paypal Express.) These types of checkout services
handle processing of the credit card information offsite (the data never
touches your server) and greatly simplify the requirements for PCI
compliance.

[Braintree](http://www.braintreepaymentsolutions.com/) also offers a
very interesting gateway option that achieves a similar benefit as
Express Checkout but allows the entire process to appear to be taking
place on the site. In other words, the customer never appears to leave
the store during the checkout. They describe this as a “transparent
redirect.” The Braintree team is very interested in helping other Ruby
developers use their gateway and have provided support to Spree
developers in the past who were interested in using their product.

### Security Alerts

Spree will periodically check for important security and release
announcements. You will see them on the administration console pages.
Alerts may be dismissed after you have read them. The automatic checking
can be disabled under Configuration | General Settings. Some
configuration information is included when checking so the alerts can be
customized to your specific installation. Here is an example of the
configuration information included in the alert request:

<javascript>\
 {“name”=\>“Spree Demo Site”,\
 “rails\_version”=\>“3.1.1”,\
 “version”=\>“0.70.1”,\
 “rails\_env”=\>“production”,\
 “host”=\>“www.spreecommerce.com”}\
</javascript>
