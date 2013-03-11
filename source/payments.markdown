Payments
--------

This guide covers how Spree handles payments and how you can configure
available payment methods or define your own.\
After reading, you should be familiar with:

-   How to configure payment methods
-   How payments are processed
-   Gateway payment methods and payment source
-   Custom payment actions
-   Creating your own payment methods

endprologue.

### Overview

Spree has a highly flexible payments model which allows multiple payment
methods to be available during checkout. The logic for processing
payments is decoupled from the checkout making it easy to define custom
payment methods with their own processing logic. Additional payment
methods can be created in the admin interface. You can define your own
payment methods to be available to customers during checkout and for
different methods to be available based on the current Rails
environment.

Payment methods typically represent a payment gateway which will process
card payments but may also include non-gateway methods of payment such
as *Check* which is provided in Spree by default.

There is also a very detailed [Payment Gateways
Guide](payment_gateways.html) which will cover more details related to
payment gateways.

### Payment Architecture

#### *Payment*

The foundation of the Spree payment mechanism is the *Payment* model
itself. This model has several important associations that we’ll now
examine.

<ruby>\
class Payment \< ActiveRecord::Base\
 belongs\_to :order\
 belongs\_to :source, :polymorphic =\> true\
 belongs\_to :payment\_method\
 …\
end\
</ruby>

A *Payment* is always associated with an order and a *PaymentMethod*.
Optionally, the *Payment* can also have a *Source*.

*Payments* also have a state machine. The possible states for a payment
are as follows:

  State        Description
  ------------ ---------------------------------------------------------------------------------------
  checkout     Checkout has not been completed
  processing   The payment is being processed (temporary - intended to prevent double submission)
  pending      The payment has been processed but not yet complete (ex. authorized but not captured)
  completed    The payment is completed - only payments in this state count against the order total
  failed       The payment was rejected (ex. credit card was declined)
  void         The payment should not be counted against the order

#### *PaymentMethod*

*PaymentMethods* represent the different options a customer has for
making payment. Most sites will accept creditcard payments through a
payment gateway but there are other options. Spree also comes with built
in support for a *Check* payment (which can be used to represent any
offline payment). There are also third party extensions that provide
support for some other interesting options such as [Paypal Express
Checkout](https://github.com/spree/spree_paypal_express).

#### *Source*

*Payments* can have an optional *Source.* The primary purpose for this
is to allow for credit card payments. In the case of a credit card
payment the *PaymentMethod* is a *Gateway*. Unfortunately that is not
enough information to process the payment.

In order to process a creditcard payment Spree also needs to know
information about the specific creditcard. This is the primary purpose
of *Source* (to provide a link to the *Creditcard* used to generate the
payment.)

NOTE: Think of the *PaymentMethod* as the form of payment (credit card,
check, etc.) and the *Source* as the thing responsible for generating
the payment (the actual credit card for instance.)

INFO: Don’t forget the [Payment Gateways Guide](payment_gateways.html)
for more information on *Gateways*.

#### *LogEntry*

*Payments* can also be associated with a collection *LogEntry* objects.
This provides for an optional storage of transactional information
related to the payment. In the credit card case, for example, this can
store information related to the gateway response for each transaction.

### Payment Processing

Spree will process all payments once the checkout is complete by calling
*Payment\#process!* on all of the order payments (generally only one
will be present at a single time.) If the *Payment* has a *Source* then
this is the opportunity to perform additional logic associated with
payment processing. So in the case of a creditcard payment, this is when
an authorization or purchase should be performed using the relevant
gateway.

INFO: *Payments* are put in the ‘processing’ state when the user submits
the order. Once in this state the payment cannot be processed again.
This prevents double authorization for credit cards and other potential
problems.

Let’s look more closely at the logic that results from a call to
*process![](+ with a credit card payment:

<ruby>
  def process)\
 if Spree::Config\
 purchase\
 else\
 authorize\
 end\
 end\
</ruby>
\
Depending on whether or not Spree is configured to “auto capture” the
credit card, either a “purchase” or an “authorize” operation will be
performed on the card
\
NOTE: The default setting for for Spree::Config is false.
\
h3. Managing Payments
\
h4. List of Payments
\
To view and manage the payments for an order, click the ‘Payments’ link
in the sidebar sub-menu when viewing the order.
\
INFO: Keep in mind that only payments in the “completed” state will
actually be used to offset the order balance.
\
h4.*Order\#payment\_state+

The *Order* also has a *payment\_state* attribute which is displayed in
the admin payments screen on the right hand side (it is labeled as
“Payment:”). This is intended as a helpful value to show the status of
all payments in total and it is comitted to the database (as opposed to
calculated on-demand) so that you can write efficient queries against
it.

The following possible *payment\_state* values exist:

  State          Description
  -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  paid           The completed payments are exactly enough to offset the order total.
  balance\_due   The completed payments are not sufficient to completely offset the order total. The customer needs to make another payment or an adjustment should be made to the order to balance what has been paid so far with the order total.
  credit\_owed   The completed payments are more than enough to offset the order total. The customer is entitled to a refund.
  failed         The most recent payment made against this order resulted in a failure.

In production, you’re unlikely to see a completed order with a
*payment\_state* of “failed” except for cases where you deliberately
configure Spree to allow users to proceed despite a failed credit card
authorization. Orders that are in progress, however, may routinely fall
into this case because it is not uncommon for users to enter the wrong
credit card information.

NOTE: You may want to keep tabs on the number of orders with a
*payment\_state* of “failed.” A sudden increase in the number of such
orders could indicate a problem with your credit card gateway and most
likely indicates a serious problem affecting customer satisfaction.

#### Payment Actions

Each of the payments listed for an order may have certain actions that
are available to the admin (i.e. capture, void, etc.) The availability
of these methods depends on the source of the payment and the
*Order\#payment\_state*.

For example, if the source of the payment is a *Creditcard* then then
following method is used to determine whether a credit can be issued.

<ruby>\
 def can\_credit?(payment)\
 return false unless payment.state  "completed"
    return false unless payment.order.payment\_state  “credit\_owed”\
 payment.credit\_allowed \> 0\
 end\
</ruby>

This bit of code can be interpreted as follows

\* You cannot credit a payment unless it’s already been processed by the
gateway successfully\
 \* You cannot credit a payment unless there is an over-payment on the
order (which will happen if the order is canceled or if you make some
type of adjustment to the order that reduces its total.)\
 \* You cannot credit a payment if you’ve already issued one or more
credits against that payment that completely offset the original
payment.

WARNING: Certain actions on credit card payments may result in an error
due to a timing issue with the settlement. Once the funds have been
deposited in the merchant account it’s generally not possible to void
such a payment. Conversely, it is also generally not possible to issue a
credit against a payment before it has settled. Settlement typically
occurs 12-24 hours after the initial transaction.

#### Creating a New Payment

The ‘New payment’ button will only be available if the order has an
outstanding balance. You can enter an amount for the new payment, this
will default to the outstanding balance on the order and must not exceed
that amount. As with the checkout process, a payment method is selected
(if there’s more than one available) and any other details are entered
that the selected method requires.

### Configuring Payment Methods

Payment methods are configured in the admin interface in the “Payment
Methods” section under “Configuration”.

Each payment method has the following configuration options:

  Configuration Option   Description
  ---------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                   The name for the method shown to the customer during checkout e.g. “Creditcard”
  Description            Additional descriptive information about the configuration, not visible to customer
  Environment            The Rails environment for which this payment method will be used. This is the same as specifying the RAILS\_ENV for this configuration (ie. *development*, *test* or *production*.)
  Provider               The Class which handles the processing of this payment method

Once created, additional options may now be seen that are specific to
the selected provider. For example, all gateway type methods have the
following options:

  Configuration Option   Description
  ---------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------
  Server                 Which server the gateway should be connecting to (this is a reference to the gateway server, not the Spree server.) Choices are: *live* and *test*.
  Test Mode              Transactions should be processed in test mode (if such a mode is supported by your gateway.)
  Active                 Whether or not the configured *PaymentMethod* is active.

### Custom Payment Methods

#### Supporting a New Gateway

Spree comes with built-in support for many popular payment gateways.
Most of the gateway support is actually provided by the third party
[active\_merchant](https://github.com/shopify/active_merchant) gem. In
most cases Spree just needs to provide a thin wrapper around what’s
already supported in active\_merchant. This wrapper allows developers to
provide their own functionality on top of the active\_merchant
implementation as well as to apply “monkey patches” to fix bugs in
active\_merchant without forking the active\_merchant gem while you wait
for the next gem release, etc.

INFO: You can read more about supporting additional gateways in the
[Payment Gateways Guide](payment_gateways.html#addinga-new-gateway)

NOTE: It would be easy to add support for dozens of new gateways in
Spree but we are only adding official support for the ones we have
tested ourselves (or a trusted member of the community has tested.) If
your gateway is not “officially” supported it should not deter you from
adding it yourself and contributing it back to the community as a patch
to Spree.

#### Other Payment Methods

##### Gateways Not Supported By active\_merchant

Its possible that you may wish to support credit card payment via a
gateway that is not supported by active\_merchant. If you’re
implementing a previously unsupported gateway then you should extend
*PaymentMethod* directly.

NOTE: There is no need to extend either *Gateway* or
*BillingIntegration* class if you’re not planning on delegating to an
active\_merchant implementation.

##### Payments Unrelated to Credit Cards

There may be situations where you need a custom *PaymentMethod* that has
nothing to do with a credit card gateway. In these cases you just need
to create a class that extends *PaymentMethod*.

#### Registering Your Payment Method

Your payment method class must be registered before it will be available
to select as a provider during configuration. The following example
shows how the
[spree\_paypal\_express](https://github.com/spree/spree_paypal_express)
extension performs the registration inside the engine’s *activate*
method:

<ruby>\
 def self.activate\
 …\
 BillingIntegration::PaypalExpress.register\
 BillingIntegration::PaypalExpressUk.register\
 end\
</ruby>

#### Custom Payment Source

If you are creating a new *PaymentMethod* you may well need to also have
a corresponding Rails model to represent that method’s
[source](payments.html#source). If you look at an existing
*PaymentMethod* in Spree you can see an example of this in how Spree
handles credit card payments. In this case *Gateway* is the
*PaymentMethod* and the payments it creates have a source instance of
the *Creditcard* model.

Let’s consider another example where a *PaymentMethod* might require its
own payment source class as well. The
[spree\_paypal\_express](https://github.com/spree/spree_paypal_express)
extension has a *PaymentMethod* of *BillingIntegration::PaypalExpress*
and this class creates a *Payment* with a source of *PaypalAccount*.

#### Custom Actions

The *Payment\#actions* method is responsible for listing the actions
associated with a payment’s source. Your custom *PaymentMethod* may
require other actions to be performed in the admin interface (most
likely after the checkout process.) In this case you will need to
implement an *actions* method in the source class.

Along with a method for each action you must define a “can*…?" method to
indicate if that particular action is possible based on the state of the
payment or other rules. Both your custom action methods and their "can*”
predicates must be defined with a payment parameter.

Let’s return to the
[spree\_paypal\_express](https://github.com/spree/spree_paypal_express)
example once again to see how all of this is done.

<ruby>\
 class PaypalAccount \< ActiveRecord::Base\
 has\_many :payments, :as =\> :source

def actions\
 %w{capture credit}\
 end

def capture(payment)\
 …\
 end

def can\_capture?(payment)\
 …\
 end

def credit(payment, amount=nil)\
 …\
 end

def can\_credit?(payment)\
 …\
 end\
 end\
</ruby>

#### Preferences

If your *PaymentMethod* requires preferences, you can add them to your
class and they will be available in the edit form associated with your
payment. If your *PaymentMethod* extends *Gateway* it will inherit the
preferences of ‘server’ and ‘test\_mode.’

You can also add new preferences to your payment as shown below:

<shell>\
 class PaymentMethod::MyMethod\
 preference :login, :string\
 preference :password, :string\
 end\
</shell>

INFO: For more information on preferences you should consult the
[Preferences Guide](preferences.html)

#### Views

##### Checkout Related

You must create a partial template that provides the interface users
will see during checkout when your payment method is selected. The
partial name should be the same as the downcase version of the class
name of your payment method class. So if your class is
PaymentMethod::MyMethod, the partial should be
‘app/views/checkouts/payment/\_my\_method.html.erb’.

You have the option of providing information you want to show to the
customer about the method they have selected.\
If your payment method requires a [source](payments.html#source), then
the partial should include the necessary form fields to produce a params
hash with the key ‘payment\_source’. When the payment is created, this
hash will be used to build the source record for the payment.

NOTE: The partial may also be blank as in the case of the *Check*
payment method.

##### Admin Related

If you create a new *PaymentMethod* you must also create two distinct
partials:

-   A partial to be used on the “show” action for a payment
-   A form to be used when editing and creating new payments

NOTE: The form partial should include form fields that match those used
in the checkout partial described above.

In the example of a payment method with the class *MyMethod* you would
create the following views:

-   app/views/admin/payments/source\_views/\_my\_method.html.erb
-   app/views/admin/payments/source\_forms/\_my\_method.html.erb

