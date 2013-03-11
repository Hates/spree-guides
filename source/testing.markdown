Testing
-------

This guide covers the test strategies and test support for Spree. After
reading it you should be familiar with:

-   How to run the tests for the Spree code
-   How to test extensions to the Spree code

Before you dive into the detail, it’s worth reviewing the [Rails guide
on testing](http://guides.rubyonrails.org/testing.html) for background
information.

endprologue.

### Overview

The Spree project currently uses [RSpec](http://rspec.info) for all of
its tests. Each of the gems that makes up Spree has a test suite that
can be run to verify the code base.

The Spree test code is an evolving story. We started out with RSpec,
then switched to Shoulda and now we’re back to RSpec. RSpec has evolved
considerably since we first tried it. When looking to improve the test
coverage of Spree we took another look at RSpec and it was the clear
winner in terms of strength of community and documentation.

### Running the Tests

#### Building a Test App

Spree consists of several different gems (see the [Source Code
Guide](source_code.html) for more details.) Each of these gems has its
own test suite which can be found in the *spec* directory. Since these
gems are also Rails engines, they can’t really be tested in complete
isolation - they need to be tested within the context of a Rails
application.

You can easily build such an application by using the Rake task designed
for this purpose.

<shell>\
 \$ bundle exec rake test\_app\
</shell>

This will build the appropriate test application inside of your *spec*
directory. It will also add the gem under test to your *Gemfile* along
with the *spree\_core* gem as well (since all of the gems depend on
this.)

NOTE: This rake task will regenerate the application (after deleting the
existing one) each time you run it. It will also run the migrations for
you automatically so that your test database is ready to go.

#### Running the Specs

Once your test application has been built, you can then run the specs in
the standard RSpec manner:

<shell>\
 \$ bundle exec rake spec\
</shell>

Or you can also do

<shell>\
 \$ bundle exec rspec spec\
</shell>

We also set up a build script that mimics what Travis-CI, you can run it
from the root of the spree project like this:\
<shell>\
 \$ bash build.sh\
</shell>

If you wish to run spec for a single file then you can do so like this:

<shell>\
 \$ bundle exec rspec spec/models/state\_spec.rb\
</shell>

If you wish to test a particular line number of the spec file then you
can do so like this:

<shell>\
 \$ bundle exec rspec spec/models/state\_spec.rb:7\
</shell>

#### Using factories

spree uses [factory\_girl](https://github.com/thoughtbot/factory_girl)
to create valid records for testing purpose. All the factories are also
packaged in the gem. So ff you are writing an extension or if you just
want to play with spree models then you can use these factories as
illustrated below.

<shell>\
 \$ rails console\
 \$ require ‘spree/core/testing\_support/factories’\
</shell>

### Testing Your Extensions

The generator used by Spree to create new extensions will do all of the
work to get your extension ready for testing. There are some cases,
however, where you may wish to maintain an extension in its own
repository as a standalone gem (so you can redistribute and share the
code.) In these cases there is a little more work to be done before you
can start testing. We’ll cover both scenarios here.

#### Testing Integrated Extensions

When you first start out coding a new extension, you may want to work
with it right alongside the Spree project that you plan to use it in.
Even if you’re eventually planning to break it out into a gem and share
the source code it’s sometimes easier to develop with it side by side
with your application.

If you’ve read the [Creating Extensions
Guide](/creating_extensions.html) you know by now how easy it is to
create a Spree extension within the context of a Rails/Spree
application:

<shell>\
 \$ spree extension foofah\
</shell>

This generator automatically creates a *spec* directory along with a
*spec\_helper.rb* file. If you’re not a fan of RSpec you can feel free
to switch to a test framework of your choosing. You’ll also need to add
the rspec-rails gem to your Gemfile if it’s not already present.

<ruby>\
 \#Gemfile\
 gem ‘rspec-rails’\
</ruby>

Then it’s time to write some specs/tests for your extension. We’re not
going to cover how to write tests here - there’s already plenty of
resources that cover this topic in depth. Once your tests are written
you simply use the standard rspec command to verify them.

<shell>\
 \$ bundle exec rspec spec\
</shell>

#### Testing Standalone Extensions

Most extensions that you’ll be working with (including all of the ones
in the extension registry) are so-called isolated extensions. They exist
in their own GitHub repositories independent of any specific Rails
application. Fortunately we have devised a way to test these extensions
as well.

Let’s start by creating a new extension - although the procedure is
basically the same for an extension that you’ve cloned from GitHub.

<shell>\
 \$ spree extension foofah\
</shell>

There’s an extra step needed in the isolated case before we can run our
tests. We need to create a “dummy” rails application to serve as a
context for the specs.

Perform the following one-time step

<shell>\
 \$ bundle exec rake test\_app\
</shell>

WARNING: The resulting dummy Rails application should never be committed
to your source code respository. You can always regenerate them with the
rake task.

Once the test app has been created, your specs can run in the manner
you’re used to with a full-fledged Rails application.

<shell>\
 \$ bundle exec rspec spec\
</shell>

### Cucumber

WARNING: [Cucumber](http://cukes.info/) is dropped in favor for RSpec +
Capybara. All cucumbers tests are converted and located inside
*spec/requests* directory.

#### Factory girl

The spree\_core gem has a good number of factories which can be used for
testing. If you are writing an extension or just testing spree you can
make use of these factories.

<shell>\
 \$ require ‘spree/core/testing\_support/factories’\
</shell>

Above command might result in following message.

<ruby>\
LoadError: no such file to load — factory\_girl\
</ruby>

If you encounter above message then just add `factory_girl` to your
`Gemfile`.

<ruby>\
gem ‘factory\_girl’\
</ruby>
