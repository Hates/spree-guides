Application Processes
---------------------

This guide will explain how the Spree Deployment Service uses Foreman &
Upstart to define and manage all the processes required by your
application.

endprologue.

### Using Foreman

All application processes are managed using
[Foreman](https://github.com/ddollar/foreman) and a default Procfile is
generated for you in the following location:

<shell>/data/spree/shared/config/Procfile</shell>

By default this includes a single Unicorn master process, the unicorn
workers are configured via the Deployment Server UI.

**Default Procfile contents:**

<ruby>web: bundle exec unicorn\_rails ~~c
/data/app\_name/shared/config/unicorn.rb~~p \$PORT ~~E ENV</ruby>
\
The provided Capistrano deployment script symlinks the Procfile into
location after each deploy.\
Capistrano Procfile Snippet:
\
<ruby>\
namespace :deploy do\
 desc “Symlink shared configs and folders on each release.”\
 task :symlink\_shared do\
 run “ln ~~nfs \#/config/database.yml \#/config/database.yml"\
 run "ln~~nfs \#{shared\_path}/config/Procfile
\#{release\_path}/Procfile”\
 end\
end
\
after ‘deploy:update\_code’, ‘deploy:symlink\_shared’\
</ruby>
\
NOTE: It’s important to remember to not edit the Procfile above
directly, any changes to this file will be reset automatically by our
configuration management system. See the “Customizing Processes” section
below for more details on how to correctly change this file.
\
h3. Upstart
\
After each deployment of your application the Procfile is exported to
Upstart compatible configuration files which the operating system uses
to manage your application’s processes.
\
Upstart is responsible for starting, stopping and restarting the
processes and will detect if a process has stopped for some reason and
automatically restart it. Upstart does not monitor for memory size
growth, and will not restart leaky processes automatically.\
Controlling Processes
\
Using Upstart you control the processes of application in a number of
ways:
\
h4. All Processes
\
You can use following commands to start, stop or restart all the
processes in your application at once:
\
<shell>\
sudo stop spree\
sudo start spree\
sudo restart spree\
</shell>
\
h4. Processes by type
\
To control all processes of a specific type you can use:
\
<shell>\
sudo start spree-web\
sudo stop spree-worker\
sudo restart spree-solr\
</shell>
\
h4. Individual Processes
\
You can control individual processes by prepending the process number at
the end:
\
<shell>\
sudo start spree-web-1\
sudo stop spree-worker-3\
sudo restart spree-solr-2\
</shell>
\
NOTE: The process number is the concurrency number specified in the
foreman export, and not the PID.

\
h3. Customizing Processes
\
To add additional processes such as Delayed Job workers to your
application, complete the following steps:
\
\* Copy the contents of the default Procfile and save it in the root our
application.\
\* Make any additions or changes you require.\
\* Remove the Procfile symlink link line from your config/deploy.rb
file.\
\* Commit your changes and deploy the application.\
\* When your application is deployed the new processes will be
automatically started.
\
**Sample Procfile containing Delayed Job workers:**
\
<ruby>\
web: bundle exec unicorn\_rails~~c
/data/app\_name/shared/config/unicorn.rb ~~p \$PORT~~E production\
worker: bundle exec rake jobs:work\
</ruby>

### Changing process concurrency

By default the provided Capistrano script will export a single process
for each entry in Procfile, for some process types you want to increase
the number of processes started, for example Delayed Job workers. You
can customize the number of process exported for each process type by
editing the foreman:export task in the Capistrano script.\
Custom Capistrano deploy.rb - creating 3 worker processes:

<ruby>\
namespace :foreman do\
 desc “Export the Procfile to Ubuntu’s upstart scripts”\
 task :export, :roles =\> :app do\
 run "cd \#{current\_path} && bundle exec foreman export upstart
/etc/init ~~a \#~~c worker=3 ~~u spree"\
 end
\
 … \
end\
</ruby>
\
The~~c (or —concurrency) switch accepts the process type and an integer
for the number of processes to start up. You specify mulitple types by
comma separating as follows: processname=2,processname=4

WARNING: It’s important to not export more than one :web process when
using the default unicorn configuration, as this refers to the master
process and not the workers, the worker count can be configured via the
Deployment’s Service UI.

.
