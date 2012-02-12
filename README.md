THIS WORK IS IN PROGRESS. FORKS WELCOME, BUT SUBSTANTIAL WORK REMAINS.
 - Greg Albrecht 2012-02-12

# Introduction

My intent with this document is to collect the best practices and
guidelines to assist Opscode Chef Cookbook authors create code that is
readable, maintainable, and easy to use.

# Authoritative Sources

The inspirations for this document, as well as a fallback sources of
guidiance, in order of precidence, are:

* [@bbatsov's](https://twitter.com/bbatsov) [Community Driven Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide)
* Google's [Python Style Guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html)
* Python's [PEP-8](http://www.python.org/dev/peps/pep-0008/)

# Alternative Sources

* Opscode's [Cookbook Style Guide Draft](http://wiki.opscode.com/display/chef/Cookbook+Style+Guide+Draft)
* Opscode's [Cookbook Style-Guide Outline](http://wiki.opscode.com/display/chef/Cookbook+Style-Guide+Outline)

# Opscode Chef Cookbook Style Guide

* Organize a Recipe as you would a Ruby program.
  TK
  e.g.

  1. imports
  2. constants
  3. resources

* Use constants for Resource paramaters.

  ```Ruby
  # Yes
  fqdn_items = data_bag_item('servers', 'fqdn')
  web_fqdn = fqdn_items['web_fqdn']
  
  apache2_site 'main website' do
    action :enable
    server_name web_fqdn
  end

  # No
  fqdn_items = data_bag_item('servers', 'fqdn')
  
  apache2_site 'main website' do
    action :enable
    server_name fqdn_items['web_fqdn']
  end
  ```

  Justification:

  1. Imagine that traceback!
  2. `nil` will get passed to the `apache2_site` Resource? TK
  3. Flow of Resource processing will break here? TK
  4. It's easier to test for the existence of a key.

* If a Recipe contains more than a few lines of pure Ruby, you might fare better with a LWRP.
* Better yet, more than a few lines of Ruby in an LWRP might fare better as a Library.
* Organize a Resources paramaters for easy program flow interpretation.
  
  I don't like this example:
  ```Ruby
  # Yes
  some_resource 'some_name' do
    action :some_action
    not_if{ some_condition }
    param1 some_paramater1
    param2 some_paramater2
  end

  # No
  some_resource 'some_name' do
    param1 some_paramater1
    param2 some_paramater2
    action :some_action
    not_if{ some_condition}
  end
  ```

  Justification:

  1. Readers can quickly determine what weather the Resource will run, and on what conidtions it will run.

* Include conditionals within a Resource.

  ```Ruby
  # Yes
  service 'apach2' do
    action [:enable, :start]
    only_if{ node['webserver'] }
  end

  # No
  if node['webserver']
    service 'apache2' do
      action [:enable, :start]
    end
  end
  ```

  Justification:

  1. What gets logged TK
  2. DRY approach
 
* Prefer `Chef::Log` over `log()`.

  ```Ruby
  # Yes
  Chef::Log.info("Hey look, I'm a webserver!")
  
  # No
  log "Hey look, I'm a webserver!"
  ```

  Justification:

  1. Line shows up twice in the log TK
  2. `log()` is not usable within LWRPs.

* No `log` or `Chef::Log` needed within a Resource.
  
  ```Ruby
  # Yes
  service 'apache2' do
    action [:enable, :start]
  end

  # No
  service 'apache2' do
    action [:enable, :start]
    Chef::Log.info('Enabling apache2 service.')
  end
  ```

  Justification:

  1. Chef will already log when it's collecting a processing a Resource.

* Don't use static Unix-style paths.

  ```Ruby
  # Yes
  ETC_SSL = ::File.join(::File::SEPARATOR, 'etc', 'ssl')
  
  # No
  ETC_SSL = '/etc/ssl'
  ```

  Justification:

  1. Where's `/etc/ssl` on NTFS? :)

* You can use `$globals` to pass constants around Recipes. TK
* If logging a Hash, use it's built-in `inspect()` method. This works for Node Mashes also.

  ```Ruby
  # Yes
  Chef::Log.debug("This Node's EC2 information is #{node['ec2'].inspect}")

  # No
  Chef::Log.debug("This Node's EC2 information is #{node['ec2']")
  ```

  Justification:

  1. Just look at the output! TK

# Why put on 5 lines what you can put on one, for under 80 characters!

  ```Ruby
  # Yes
  gem_package('right_aws'){ action :nothing }.run_action(:install)

  # No
  g = gem_package 'right_aws' do
    action :nothing
  end
  g.run_action(:install)
  ```

  Justification:

  1. File this under personal preference. TK

# Treat an LWRP's Resource definition as an analog of a method envelope.

  ```Ruby
  # Here's a pure-ruby example of a method envelope:
  def my_method(food='taco', price=2):
    puts "Here's a delicious #{food} for $#{price}!"

  # Here's the equivalent Resource definition in Chef:
  attribute :food, :kind_of => String, :required => false, :default => 'taco', :regex => /\w+/
  attribute :price, :lind_of => [Integer, Float], :required => false, :default => 1
  ```
