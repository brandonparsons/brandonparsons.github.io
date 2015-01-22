---
layout: post
title: "Chef Recipe Details: Bootstrap your Vagrant Machine"
shortname: "Chef Recipe Details for Rails Deployment"
date: 2012-11-02 09:00
tags:
  - tools
  - web development
  - deployment 
description: "Getting up and running with Rails on Chef. This is a group of recipes I hacked together in order to deploy a Rails app."
---

In my last post, I described [how to combine Vagrant and Chef to achieve dev/prod parity](http://brandonparsons.me/2012/vagrant-and-chef-for-ubuntu-deployment-server/).

Due to the feedback I received, this post is intended to share some of those Chef recipes to get you up and running quickly.

For now I'm not going to share this on Github, as I'd need to go through the entire thing and make sure I wasn't crazy and posted a password in there somewhere.  However with everything I put on this post it should be a simple copy/paste.

## My Run List ##

Here is the node JSON file that bootstraps my deployment server.

```javascript
{
  "run_list": [
    "recipe[apt]",
    "recipe[ohai]",
    "recipe[build-essential]",
    "recipe[openssl]",
    "recipe[git]",
    "recipe[user]",
    "recipe[logrotate]",
    "recipe[brandon::add_deployer_user_REAL_VM]",
    "recipe[brandon::change_ssh_port]",
    "recipe[ruby_build]",
    "recipe[rbenv::user]",
    "recipe[brandon]",
    "recipe[fail2ban]", // Fail2Ban needs to come after 'brandon::change_ssh_port' so that ssh port has been correctly changed
    "recipe[brandon::fail2ban_setup]",
    "recipe[postgresql::server]",
    "recipe[postgresql::client]",
    "recipe[nginx::http_gzip_static_module]",
    "recipe[nginx::http_ssl_module]",
    "recipe[nginx::source]",
    "recipe[brandon::set_nginx_conf]",  // Has to come after installation of nginx
    "recipe[redis::server]", // Has to come after brandon (create log directory)
    "recipe[brandon::set_redis_log_permissions]",
    "recipe[brandon::upstart_recipes]",  // Has to come after recipe[brandon] as monit must be installed already
    "recipe[brandon::logrotate]" // Leave at end to ensure logfile in place
  ],
  "username": "deployer",
  "time_zone": "Canada/Mountain",
  "ssh_port": 555,
  "rbenv": {
    "user_installs": [
      {
        "user": "deployer",
        "rubies": [
          "1.9.3-p194"
        ],
        "global": "1.9.3-p194",
        "gems": {
          "1.9.3-p194": [
            {
              "name": "bundler"
            },
            {
              "name": "rake"
            },
            {
              "name": "awesome_print"
            }
          ]
        }
      }
    ]
  },
  "user": {
    "ssh_keygen": true
  },
  "nginx": {
    "default_site_enabled": false,
    "workers": 4,
    "init_style": "init",
    "source": {
      "prefix": "/opt/nginx"
    }
  },
  "redis": {
    "init_style": "init",
    "install_type": "source",
    "config": {
      "logfile": "/var/log/redis-server.log"
    }
  },
  "memcached": {
    "memory": 64
  },
  "upstart": {
    // Whether or not to boot services on system startup
    "memcached": true,
    "mongo": true,
    "monit": true,
    "nginx": true,
    "postgres": true,
    "redis": true
  }
}
```

A fair chunk of those recipes are self-explanatory, or just load default cookbooks directly from an Opscode repo cookbook.  I'll go through some of the custom recipes here.

## Customized Deployment Recipes ##

### Contents of the 'Brandon' Recipe ###

The `brandon` recipe really just calls a number of individual recipes that I've kept together in a single location.  The only trouble is that some of the other recipes had to be run at specific times, and that's why you will see something like this in the run list:

`"recipe[brandon::add_deployer_user_REAL_VM]"`

In those cases, it directly pulls the recipe from the `/site_cookbooks/brandon` directory.

Here are the contents of the `default.rb` recipe:

```ruby
include_recipe "brandon::update_packages"
include_recipe "brandon::ensure_ruby_setup"
include_recipe "brandon::install_irb"
include_recipe "brandon::copy_dotfiles"
include_recipe "brandon::set_timezone"
include_recipe "brandon::install_nodejs" # Need a javascript runtime
include_recipe "brandon::set_up_postgres_hstore"
include_recipe "brandon::install_mongodb"
include_recipe "brandon::install_memcached"
include_recipe "brandon::install_monit"
include_recipe "brandon::firewall"
```

### Add Deployer User ###

```ruby
user_account 'deployer' do
  # keys for file ~/.ssh/authorized keys
  ssh_keys  ['paste your public ssh key here']
end

group "admin" do
  members ['brandon', 'deployer']
  action :create
end

group "sudo" do
  members ['brandon', 'deployer']
  action :create
end
```

### Change the Default SSH Port ###

*Because everyone is going to guess 22...*

```ruby
execute "change the ssh port for security purposes" do
  command <<-EOC
    sudo service ssh stop
    sudo mv /etc/ssh/sshd_config /etc/ssh/sshd_config_old
    sudo sed 's/Port 22/Port #{node['ssh_port']}/g' /etc/ssh/sshd_config_old | sudo sed 's/PermitRootLogin yes/PermitRootLogin no/g' > /tmp/sshd_config
    sudo mv /tmp/sshd_config /etc/ssh/sshd_config
    sudo service ssh start
  EOC
  action :run
  creates "/etc/ssh/sshd_config_old"
end

execute "disable password login for security purposes" do
  command <<-EOC
    sudo service ssh stop
    sudo mv /etc/ssh/sshd_config /etc/ssh/sshd_config_old2
    sudo sed 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config_old2 | sed 's/^ChallengeResponseAuthentication yes/ChallengeResponseAuthentication no/' | sed 's/^#ChallengeResponseAuthentication yes/ChallengeResponseAuthentication no/' > /tmp/sshd_config
    sudo mv /tmp/sshd_config /etc/ssh/sshd_config
    sudo service ssh start
  EOC
  action :run
  creates "/etc/ssh/sshd_config_old2"
end
```

### Update Packages ###

```ruby
execute "upgrade packages" do
  action :run
  command "apt-get -y upgrade"
end
```

### Ensure Ruby is Set Up ###

```ruby
execute "set up path for bash" do
  command <<-EOC
    sudo echo 'export PATH="/home/#{node['username']}/.rbenv/bin:$PATH"' >> /home/#{node['username']}/.bashrc
    sudo echo 'eval "$(rbenv init -)"' >> /home/#{node['username']}/.bashrc
  EOC
  action :run
end
```

### Install IRB ###

```ruby
execute "install irb" do                                             
    action :run
    command "apt-get -y install irb"      
end
```

### Copy Dotfiles ###

```ruby
template "/home/#{node['username']}/.gemrc" do
  source "gemrc"
  owner "#{node['username']}"
  group "#{node['username']}"
  mode "0755"
end

template "/home/#{node['username']}/.gitignore_global" do
  source "gitignore_global"
  owner "#{node['username']}"
  group "#{node['username']}"
  mode "0755"
end

template "/home/#{node['username']}/.irbrc" do
  source "irbrc"
  owner "#{node['username']}"
  group "#{node['username']}"
  mode "0755"
end
```


### Set Timezone ###

```ruby
bash "modify timezone" do
  code <<-EOC
    sudo cp /etc/timezone /etc/timezone_old
    echo "#{node['time_zone']}" > /etc/timezone    
    dpkg-reconfigure -f noninteractive tzdata
  EOC

  creates "/etc/timezone_old"
end
```


### Install NodeJS ###

*Because you need a JavaScript runtime*

```ruby
execute "install nodejs" do                                      
    action :run
    command "apt-get -y install nodejs"      
end
```

### Set up PostgreSQL HStore Add-in ###

```ruby
# Allows installation of hStore
execute "install postgres contrib" do                                             
    action :run
    command "apt-get -y install postgresql-contrib-9.1"      
end

# Install hStore on all databases created on this node
execute "hstore extension on template1" do
  command <<-EOC
    sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS hstore";
  EOC
  action :run
end
```

### Install MongoDB ###

```ruby
bash "install mongodb" do
  code <<-EOC
    apt-key adv --keyserver keyserver.ubuntu.com --recv 7F0CEB10
    touch /etc/apt/sources.list.d/10gen.list
    echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' > /etc/apt/sources.list.d/10gen.list
    apt-get update
    apt-get install mongodb-10gen
  EOC

  creates "/etc/apt/sources.list.d/10gen.list"
end
```

### Install Memcached ###

```ruby
execute "install nodejs" do                                         
    action :run
    command "apt-get -y install memcached"      
end
```

### Install Monit ###

```ruby
execute "install monit" do                                             
    action :run
    command "apt-get -y install monit"      
end

directory "/etc/monit.d" do
  owner "root"
  group "root"
  mode 0755
end
```

### Set up Firewall Rules ###

```ruby
# open standard ssh port, enable firewall
firewall_rule "ssh" do
  port node['ssh_port']
  protocol :tcp
  action :allow
  notifies :enable, "firewall[ufw]"
end

# open standard http port to tcp traffic only; insert as first rule
firewall_rule "http" do
  port 80
  protocol :tcp
  position 1
  action :allow
end

firewall "ufw" do
  action :nothing
end

execute "limit ssh retries" do
  command "ufw limit ssh/tcp"
  action :run
end
```

### Fail2Ban Setup ###

```ruby
template "/etc/fail2ban/jail.local" do
  source "fail2ban_jail_local"
  owner "root"
  group "root"
  mode 0644
  notifies :restart, "service[fail2ban]"
end
```

### Set NGinx Configuration ###

```ruby
service "nginx" do
  action :stop
end

execute "move default nginx conf" do
  action :run
  command "mv /etc/nginx/nginx.conf /etc/nginx/nginx_old.conf"
end

template "/etc/nginx/nginx.conf" do
  source "nginx_conf"
  owner "root"
  group "root"
  mode 0644
  notifies :start, "service[nginx]"
end
```

### Set Redis Log Permissions ###

```ruby
execute "open up redis log" do
  action :run
  command "chmod 0766 /var/log/redis-server.log"
end
```

### Get Upstart Doing the Hard Work for You! ###

Because the Upstart configuration files include a `start on runlevel[2345]` for any services you specify in the node JSON file, we don't want the old services getting in the way.

```ruby
service "memcached" do
  action :stop
end

service "nginx" do
  action :stop
end

service "mongodb" do
  action :stop
end

service "redis" do
  action :stop
end

service "redis-server" do
  action :stop
end

service "postgresql" do
  action :stop
end

service "monit" do
  action :stop
end

execute "Stop use of services" do
  command <<-EOC
    sudo update-rc.d -f memcached remove
    sudo update-rc.d -f nginx remove
    sudo update-rc.d -f mongodb remove
    sudo update-rc.d -f redis remove
    sudo update-rc.d -f redis-server remove
    sudo update-rc.d -f postgresql remove
    sudo update-rc.d -f monit remove
  EOC
  action :run
end
```

## Drop in Upstart config files

```ruby
template "/etc/init/memcached.conf" do
  source "upstart_memcached.erb"
  owner "root"
  group "root"
  mode "0755"
end

template "/etc/init/nginx.conf" do
  source "upstart_nginx.erb"
  owner "root"
  group "root"
  mode "0755"
end

execute "move mongo conf" do
    action :run
    command "mv /etc/init/mongodb.conf /etc/init/mongodb_old"
end

template "/etc/init/mongodb.conf" do
  source "upstart_mongo.erb"
  owner "root"
  group "root"
  mode "0755"
end

template "/etc/init/redis.conf" do
  source "upstart_redis.erb"
  owner "root"
  group "root"
  mode "0755"
end

template "/etc/init/postgres.conf" do
  source "upstart_postgres.erb"
  owner "root"
  group "root"
  mode "0755"
end

template "/etc/init/monit.conf" do
  source "upstart_monit.erb"
  owner "root"
  group "root"
  mode "0755"
end
```


### Last, but not least - LogRotate ###

This sets up just a single logrotate entry.  But that's because I do the rest of this from Capistrano (in case this server doesn't have both a web server and database, for example)

```ruby
logrotate_app "fail2ban" do
  cookbook "logrotate"
  path "/var/log/fail2ban.log"
  frequency "weekly"
  rotate 4
  create "644 root admin"
end
```

## Templates ##

A bunch of these recipes copy templated ERB files over onto your server.  The contents of these files goes into `site_cookbooks/templates/default`.

### Fail2Ban Jail Local ###

```
[ssh-ddos]
enabled = true
```

### gemrc ###

```
gem: --no-ri --no-rdoc
install: --no-ri --no-rdoc
update: --no-ri --no-rdoc
```

### GitIgnore Global ###

```
# If you find yourself ignoring temporary files generated by your text editor
# or operating system, you probably want to add a global ignore instead:
#   git config --global core.excludesfile ~/.gitignore_global
#
# Here are some files you may want to ignore globally:

# scm revert files
**.orig

# Mac finder artifacts
.DS_Store

# Netbeans project directory
/nbproject/

# RubyMine project files
.idea

# Textmate project files
/*.tmproj

# vim artifacts
**.swp
```

### Irbrc ###

```
require 'irb/completion'
require 'ap'
require 'hirb'
require 'pp'

IRB.conf[:AUTO_INDENT] = true
IRB.conf[:USE_READLINE] = true

Hirb.enable

Hirb::Formatter.dynamic_config['ActiveRecord::Base']

# http://blog.nicksieger.com/articles/2006/04/23/tweaking-irb
ARGV.concat ["--readline", "--prompt-mode", "simple"]

require 'irb/ext/save-history'
IRB.conf[:SAVE_HISTORY] = 500
IRB.conf[:HISTORY_FILE] = File.expand_path('~/.irb_history')

# http://ozmm.org/posts/time_in_irb.html
def time(times = 1)
  require 'benchmark'
  ret = nil
  Benchmark.bm { |x| x.report { times.times { ret = yield } } }
  ret
end

# list object methods
def local_methods(obj=self)
  (obj.methods - obj.class.superclass.instance_methods).sort
end

def ls(obj=self)
  width = `stty size 2>/dev/null`.split(/\s+/, 2).last.to_i
  width = 80 if width == 0
  local_methods(obj).each_slice(3) do |meths|
    pattern = "%-#{width / 3}s" * meths.length
    puts pattern % meths
  end
end

# reload this .irbrc
def IRB.reload
  load __FILE__
end

puts "Yes! config loaded"
```

### NGinx Conf ###

This only sets up the general configuration - the app specific configurations are set up in Capistrano.

```
user www-data www-data;
worker_processes  <%= node['nginx']['workers'] || 2 %>;

error_log /var/log/nginx/nginx_error.log;
pid        /var/run/nginx.pid;

events {
  worker_connections  1024;
  accept_mutex on; # because worker processes > 1
}

http {
  include       mime.types;
  default_type  application/octet-stream;

  log_format custom_format [$host] ($time_local): "$request" | Status: $status | URI: $uri;
  
  access_log /var/log/nginx/nginx_catchall.log custom_format;

  sendfile        on;
  tcp_nopush     on;
  tcp_nodelay off;

  # open_file_cache max=1000 inactive=20s;
  # open_file_cache_valid 30s;
  # open_file_cache_min_uses 2;

  keepalive_timeout  5;

  gzip on;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_proxied any;
  gzip_min_length 500;
  gzip_disable "MSIE [1-6]\.";
  gzip_types text/plain text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript application/json;

  include /etc/nginx/sites-enabled/*;
}
```

### Upstart Memcached ###

```
<% if node['upstart']['memcached'] %>
start on runlevel [2345]
stop on runlevel [!2345]
<% end %>

respawn
respawn limit 10 5

exec /usr/bin/memcached -m <%= node['memcached']['memory'] %> -p 11211 -u memcache -l 127.0.0.1
```

### Upstart MongoDB ###

```
limit nofile 20000 20000

# wait 300s between SIGTERM and SIGKILL.
kill timeout 300

pre-start script
    mkdir -p /var/lib/mongodb/
    mkdir -p /var/log/mongodb/
end script

<% if node['upstart']['mongo'] %>
start on runlevel [2345]
stop on runlevel [!2345]
<% end %>

respawn
respawn limit 10 5

script
  ENABLE_MONGODB="yes"
  if [ -f /etc/default/mongodb ]; then . /etc/default/mongodb; fi
  if [ "x$ENABLE_MONGODB" = "xyes" ]; then
        if [ -f /var/lib/mongodb/mongod.lock ]; then
                rm /var/lib/mongodb/mongod.lock
                sudo -u mongodb /usr/bin/mongod --config /etc/mongodb.conf --repair
        fi
        exec start-stop-daemon --start --quiet --chuid mongodb --exec  /usr/bin/mongod -- --config /etc/mongodb.conf
  fi
end script
```

### Upstart Monit ###

```
limit core unlimited unlimited

<% if node['upstart']['monit'] %>
start on runlevel [2345]
stop on runlevel [!2345]
<% end %>

respawn
respawn limit 10 5

exec /usr/bin/monit -Ic /etc/monit/monitrc
```

### Upstart NGinx ###

```
<% if node['upstart']['nginx'] %>
start on runlevel [2345]
stop on runlevel [!2345]
<% end %>

env DAEMON=<%= node['nginx']['source']['prefix'] %>/sbin/nginx
env PID=/var/run/nginx.pid

expect fork
respawn
respawn limit 10 5
#oom never

pre-start script
        $DAEMON -t
        if [ $? -ne 0 ]
                then exit $?
        fi
end script

exec $DAEMON
```

### Upstart PostgreSQL ###

```
description "PostgreSQL 9.1 Server"
author "PostgreSQL"

<% if node['upstart']['postgres'] %>  
start on runlevel [2345]
stop on runlevel [!2345]
<% end %>

respawn
respawn limit 10 5

pre-start script
    if [ -d /var/run/postgresql ]; then
        chmod 2775 /var/run/postgresql
    else
        install -d -m 2775 -o postgres -g postgres /var/run/postgresql
    fi
end script
exec su -c "/usr/lib/postgresql/9.1/bin/postgres -D /var/lib/postgresql/9.1/main -c config_file=/etc/postgresql/9.1/main/postgresql.conf" postgres
```

### Upstart Redis ###

```
<% if node['upstart']['redis'] %>
start on runlevel [2345]
stop on runlevel [!2345]
<% end %>

expect fork
respawn
respawn limit 10 5

exec sudo su - redis -c "/opt/redis/bin/redis-server /etc/redis/redis.conf"
```

## That's It! ##

You should now have a fully provisioned deployment server.  Now you just use Capistrano to:

- Set up your app-specific Nginx configuration file
- Set your Unicorn config
- Create postgres database / user
- Use Foreman to export app-specific upstart files

I hope you found this post useful.  *(I'd appreciate a few +1's if so :D )*

