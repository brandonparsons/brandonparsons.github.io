---
title: "The Ultimate in Dev/Prod Parity: Standing up an Ubuntu Server with Chef and Vagrant"
shortname: "Dev/Prod Parity - Server with Chef and Vagrant"
tags:
  - tools
  - web development
  - deployment 
description: "Using Vagrant to emulate your exact production environment locally on your computer. Great for Rails development boxes."
---

## Intro

The convenience of [VirtualBox][] & [Vagrant][] makes dev/prod parity possible for even the newest developers.  No longer do you have to deploy to production and cross your fingers... You can have an environment exactly matching your Production environment running on your local computer, whether you develop on a Mac or Linux *(or even a P.C. - probably... I'm sure there's a way to get it working)*.

Combining [Vagrant][] with [Chef][] allows the provisioning process to be completely repeatable.  Not only does this allow for completely throwaway environments, but it also ensures your production and staging boxes are completely identical.

[Virtualbox]: https://www.virtualbox.org/
[Vagrant]: http://vagrantup.com/
[Chef]: http://wiki.opscode.com/display/chef/Home

## Installation

1. Install the appropriate [VirtualBox][] and [Vagrant][] packages from the downloads page on their website.  *(I have used the [Vagrant][] package rather than installing from a Gem.)*
2. Install the Knife-solo gem to automate Chef cookbook creation, server bootstrapping and cookbook usage: `gem install knife-solo`
3. Set up your local chef repo:
  - `knife kitchen chef_repo`
  - (It would be smart to use version control for this repo - i.e. Git with Github/Bitbucket)
  - Copy cookbooks to your `chef_repo/cookbooks` directory.  Consider directly cloning these cookbooks from their respective GitHub repos to ensure you can keep them up to date.

## Configuration of your Vagrant Box

What we are going to do is use Vagrant to load up a simple Ubuntu 12.04LTS box, saved as "Ubuntu1204LTS".  This box can be created and destroyed at will.  Then Knife-Solo will be used to apply a Chef server setup remotely via SSH.  We can test this server on our local machine, and then use the exact same Chef provisioning recipe on the production server for precise dev/prod parity.

### Set up your VagrantFile

This goes at the root of your `chef_repo` directory.  Mine looks like this:

```ruby
Vagrant::Config.run do |config|
  config.vm.box = "Ubuntu1204LTS"
  config.vm.box_url = "http://dl.dropbox.com/u/1537815/precise64.box"
  config.vm.network :hostonly, "192.168.0.10"
  config.ssh.forward_agent = true
end
```

The `forward_agent` option will allow you to use local SSH keys to log in to remote services directly from the Vagrant/remote host.  The network option will set up the Vagrant box using your local network at the IP address of your choice.  I've provided the `box_url` so that the correct file will automatically download the first time you set it up.

If you want to test your Vagrant box:

```ruby
vagrant up # This will load the base Ubuntu 12.04 box, set up any port forwarding you specify, configure SSH etc
vagrant ssh # This will get you into your box
```

Once you are finished, simply `vagrant halt` (or `vagrant suspend`).  To wipe the slate clean (and get rid of the box) `vagrant destroy`.

### Set up your Chef provisioning `run_list`

This goes in `chef_repo/nodes/default.json`:

```javascript
{
  "run_list": [
    "recipe[apt]",
    "recipe[ohai]",
    "recipe[build-essential]",
    "recipe[openssl]"
    .....
}
```

This is the key to setup of your Ubuntu server.  There are infinite resources on the internet that will help configure the server - don't reinvent the wheel!  I'll provide an overview of the recipes I use at the end of this post.

You can reference any recipe here that is located in `chef_repo/cookbooks`.  They will run in the exact order you specify. That directory is intended for cookbooks that come directly off the internet (the equivalent of a `vendor` directory).  It is intended that you put your own cookbooks in the `site_cookbooks` directory.

If you create a `main` cookbook, just place your instructions in the `recipes/default.rb` file.  You can reference other recipes in the same folder from this file (via `include_recipe`).  Alternatively you can include other recipes from this cookbook in your run list directly: `"recipe[main::my_other_recipe]"`.

### Deploy Locally

You will use the knife-solo gem to automate this process.  You will use the same commands to deploy to your production server.  First, create an SSH key on your local machine, and copy the public key over to the vagrant box.  That will allow us to easily SSH into the vagrant box using the same commands as we would on the production machine. *(Vagrant provides a handy `vagrant ssh` command which you can use to quickly SSH into the machine.  I use this when playing around with the system, but I'd like to keep the actual deployment commands identical.)*

`cat ~/.ssh/vagrant_vm.pub | ssh vagrant@192.168.0.10 "cat >> ~/.ssh/authorized_keys; chmod 600 ~/.ssh/authorized_keys"`

*Vagrant boxes all have the `vagrant` user automatically configured.  I use this for basic local deployment, but you'll likely need to set it up for a different user on the final box.*

Next, we use knife prepare to copy the cookbooks to the remote machine.

`knife prepare vagrant@192.168.0.10`

Then instruct chef to provision the server using the relevant node file:

`knife cook vagrant@192.168.0.10 nodes/default.json`

Finally we remove traces of the Chef config from the remote machine:

`knife wash_up vagrant@192.168.0.10`

### Creating a throwaway environment

Once you are happy with the local dev box, you can save tons of time by pre-packaging the system in order so that a fresh system is a simple as `vagrant destroy` `vagrant up`.

We will package up a box with our custom environment as follows:

```bash
vagrant package
vagrant box add my_box_name package.box
rm package.box
vagrant destroy
```

Now in your app project (Ruby on Rails or whatever), just throw in a VagrantFile that looks like the following:

```ruby
Vagrant::Config.run do |config|
  config.vm.box = "my_box_name"
  config.vm.network :hostonly, "192.168.0.10"
  config.ssh.forward_agent = true
end
```

And you can `vagrant up` in your project file and then deploy to 192.168.0.10 with a tool like [Capistrano][].

[Capistrano]: https://github.com/capistrano/capistrano

## Configuration of your Production Box

You'll need to get a VPS or dedicated host with one of the numerous companies on the internet.  The only difference in deploying to your remote VPS is simply adjusting the `knife-solo` commands to reflect your VPS IP.

However, you'll probably want to run slightly different recipes on the remote VPS, especially if you plan to use separate boxes as an app server, database or web server.  All you need to do is create a new run list in `chef_repo/nodes/anything.json`, and point to that file with knife-solo:

`knife cook myname@xx.xx.xx.xx nodes/anything.json`

If you copy the run list directly from `default.json`, your production box will be an exact mirror of your Vagrant box.  You should have precise dev/prod parity.

## Ubuntu Server Recipes

The [Chef][] world is huge - far more than I'd ever be able to cover here.  Not only that, there are far more knowledgeable people than me that you should be listening to!

In any case - I use [Chef][] provisioning to load the following onto the Ubuntu box:

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
    "recipe[brandon::add_deployer_user]",
    "recipe[ruby_build]",
    "recipe[rbenv::vagrant]",
    "recipe[rbenv::user]",
    "recipe[brandon]",
    "recipe[fail2ban]",
    "recipe[brandon::fail2ban_setup]",
    "recipe[postgresql::server]",
    "recipe[postgresql::client]",
    "recipe[nginx::http_gzip_static_module]",
    "recipe[nginx::http_ssl_module]",
    "recipe[nginx::source]",
    "recipe[brandon::set_nginx_conf]",  // Has to come after installation of nginx
    "recipe[redis::server]",
    "recipe[brandon::upstart_recipes]", // Has to come after recipe[brandon] as monit must be installed already
    "recipe[brandon::logrotate]" // Leave at end to ensure logfile in place
  ],
  ....
  // Additional configuration options
```

Most of these packages are self explanatory.  I use `rbenv` to manage Ruby installations on the server in order to keep it simple.  I found it more confusing to run using `rvm` as a result of the shell modifications that were necessary.

The recipes inside the `brandon` cookbook include the following:

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

These are also fairly self-explanatory.  If anyone wants to copy these recipes let me know.

Some of these will not be necessary when you deploy to your production box - e.g. `rbenv::vagrant`.  This recipe puts wrappers around `rbenv` specifically for a Vagrant VM.  Be sure to exclude this recipe from your `nodes/list.json` that deploys to production.

## What's next?

There are a number of setup options that I have purposefully left out of Chef's scope.  These are items that I feel I would want app-specific control over, for example Upstart and Monit configuration.  The idea being that with simple node configuration, you can set up repeatable Ubuntu servers.  Upstart and Monit configuration will depend heavily on what the node is being used for, and may even vary depending on the specific app.  I include these recipes in my server's [Capistrano][] setup - perhaps for my next post!

**P.S.** When running on your Vagrant VM, it may be a good idea to run as a Staging environment to customize how e-mail is sent, etc.

**EDIT**: Check out my next post for specifics on [Chef recipes to bootstrap your Rails development server on Ubuntu.](/2012/chef-recipes-for-rails-deployment/)
