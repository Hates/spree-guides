Developer Tips and Tricks
-------------------------

This guide presents accumulated wisdom from person-years of Spree use.

endprologue.

### Upgrade Considerations

#### The important commands

*spree —update*\
was removed in favor of Bundler.

Before updating, you will want to ensure the installed spree gem is\
up-to-date by modifying *Gemfile* to match the new spree version and\
run *bundle update*.

Thanks to Rails 3.1 Mountable Engine, the update process is
“non-destructive”\
than in previous versions of Spree. The core files are encapsulated
separately from\
sandbox, thus upgrading to newer files will not override nor replace\
sandbox’s customized files.

This makes it easier to see when and how some file has changed – which
is often\
useful if you need to update a customized version.

#### Dos and Don’ts

NOTE: Try to avoid modifying *config/boot.rb* and
*config/environment.rb*: use [initializers](#initializers) instead.

#### Tracking changes for overridden code

Be aware that core changes might have an impact on the components you
have overridden in your project.\
You might need to patch your local copies, or ensure that such copies
interact correctly with changed\
code (e.g. using appropriate ids in HTML to allow the JavaScript to
work).

If you can help us generalise the core code so that your preferred
effect is achieved by altering\
a few parameters, this will be more useful than duplicating several
files. Ideas and suggestions are always\
welcome.

#### Initializers

Initializers are run during startup, and are the recommended way to
execute certain settings. You can\
put initializers in extensions, thus have a way to execute
extension-specific configurations.

See the [extensions guide](extensions.html#extension-initializers) for
more information.

### Debugging techniques

#### Use tests!

Use *rake spec* and *rake test* to test basic functioning after you’ve
made changes.

#### Analysing crashes on a non-local machine

If you’re testing on a server, whether in production or development
mode, the following code in one\
of your *FOO\_extension.rb* files might save some time. It triggers
local behaviour for users who have\
an admin role. One useful consequence is that uncaught exceptions will
show the detailed error page\
instead of *404.html*, so you don’t have to hunt through the server
logs.

<shell>\
Spree::BaseController.class\_eval do\
 def local\_request?\
 ENV[“RAILS\_ENV] !=”production" || current\_user.present? &&
current\_user.has\_role?(:admin)\
 end\
end\
</shell>

### Managing large projects

#### To fork or not to fork…

Suppose there’s a few details of Spree that you want to override due to
personal or client preference,\
but which aren’t the usual things that you’d override (like views) - so
something like tweaks to the\
models or controllers.

You could hide these away in your site extension, but they could get
mixed up with your real site\
customizations. You could also fork Spree and run your site on this
forked version, but this can\
also be a headache to get right. There’s also the hassle of tracking
changes to *spree/master* and\
pulling them into your project at the right time.

So here’s a compromise: have an extra extension, say *spree-tweaks*, to
contain your small collection\
of modified files, which is loaded first in the extension order. The
benefits are:

-   it’s clear what you are overriding, and easier to check against core
    changes
-   you can base your project on an official gem release or a
    *spree/master* commit stage
-   such tweaks can become part of your client site project and be
    managed with SCM etc.

If you find yourself wanting extensive changes to core, this technique
probably won’t work so well.\
But then again, if this is the case, then you probably want to look
seriously at splitting some\
code off into stand-alone extensions and then see whether any of the
other code should be contributed\
to the core.

#### Setting up submodules

Some of us use [Git
submodules](https://git.wiki.kernel.org/index.php/GitSubmoduleTutorial)\
to tie large projects together: the basic Spree shell plus the\
*site/* extension is the main repository, and everything else is loaded
in as a submodule,\
including Spree itself. See [this Spree-demo
fork](https://github.com/paulcc/spree-demo/tree)\
for an example.

The basic command to set up a submodule is this. Call it for each of the
submodules you need.\
For Spree, you want to use the path *vendor/spree*.

<shell>\
git submodule add some\_repo [-b branch] vendor/extensions/some\_ext\
</shell>

If your project is public, then you probably want to give the public
version of your repo so\
other people can use it without change. When I need to change a
submodule in place, I add a\
new remote with the proper URL for changes to be pushed to.\
The branch option is recommended (and you probably want to specify
“master” most of the time).

#### Checking out submodules

On the first run, the following command fetches the submodule
repositories into the nominated\
directories.\
<shell>\
git submodule update —init\
</shell>

#### Managing submodules

*git submodule foreach* is very useful for managing your submodules. For
example, you can update all of them\
with *git submodule foreach “git pull”*, or check for modifications with
this\
(the *echo ok* gets round a return-code issue):\
<shell>\
git submodule foreach “git status || echo ok”\
</shell>

When pushing modifications, remember to check each of your repositories
(with *foreach* above),\
and then check in the main repository. Git tracks the last commit in
each submodule and includes\
this information in the main repository, hence allowing the right
versions to be checked out\
later.

NOTE: *git foreach* seems to be a recent addition, so you might need to
update your installation of *git*.

<!-- hack comment since guides gem doesn't allow you to end with INFO, NOTE, etc. -->

