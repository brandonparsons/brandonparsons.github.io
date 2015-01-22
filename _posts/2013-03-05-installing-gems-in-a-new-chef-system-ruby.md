---
title: Installing gems in a new Chef system Ruby during the Chef Run
shortname: Installing gems in a new Chef System Ruby
description: "If you don't correctly load the Ruby binary during installation of a gem, Chef will install to the incorrect Ruby, and your recipes could fail."
tags:
  - tools
  - web development
  - ruby
---

If you are using [Chef][chef] to manage your server deployments (and you *should* be!), then you'll likely have run into the situation where you install a new system [Ruby][ruby] and need to install a few system-wide gems to bootstrap your install (think Bundler/Rake).

However, when you install the system ruby, and use the standard [Chef][chef] gem install resource:

```ruby
gem_package "bundler"
```

then you will run into trouble as Chef will try to use the original system ruby binary when installing the gem for you.

## The Fix ##

After some Googling, and time on StackOverflow, I've put together a workable solution.  All it does it reload the `Ohai` resource (updating information about your system/Ruby binary), and performs the gem installation in a `ruby_block` so that it does not run at compile-time.

```ruby

#=========================
# Install system Ruby
#=========================

## install ruby here....

#=========================
# Install gems in new ruby
#=========================

# Need to reload OHAI to ensure the newest ruby is loaded up
ohai "reload" do
  action :reload
end

["bundler", "rake", "...."].each do |gem_to_install|
  ruby_block "install #{gem_to_install} in new ruby" do
    block do
      g = Chef::Resource::GemPackage.new(gem_to_install)
      g.gem_binary "#{node['languages']['ruby']['bin_dir']}/gem"
      g.run_action(:install)
    end
    action :create
  end
end

```

That ought to do it! Hope it saves someone some time....

[chef]: http://www.opscode.com/chef/
[ruby]: http://www.ruby-lang.org/en/

