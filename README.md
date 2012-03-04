**THIS IS IN PROGRESS. FORKS WELCOME, BUT SUBSTANTIAL WORK REMAINS.**
 - Greg Albrecht 2012-02-12

# Introduction

The intent of this document is to collect the best practices and guidelines
to assist Cookbook authors in creating code that is readable, maintainable, 
and easy to use.


# Authoritative Sources

The inspirations for this document, as well as a fallback sources of
guidance, in order of precedence, are:

* [Andrew Crump's](https://twitter.com/acrmp) [foodcritic](http://acrmp.github.com/foodcritic/)
* [@bbatsov's](https://twitter.com/bbatsov) [Community Driven Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide)
* Google's [Python Style Guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html)
* Python's [PEP-8](http://www.python.org/dev/peps/pep-0008/)

TODO(gba): Integrate Andrew's guide into this one.


# Alternative Sources

Alternative guidelines are available at:

* Opscode's [Cookbook Style Guide Draft](http://wiki.opscode.com/display/chef/Cookbook+Style+Guide+Draft)
* Opscode's [Cookbook Style-Guide Outline](http://wiki.opscode.com/display/chef/Cookbook+Style-Guide+Outline)

TODO(gba): Integrate Opscode's guides into this one.

# Chef Cookbook Style Guide
## Recipe Organization
* Organize a Recipe as you would a Ruby program.
* Ruby `include` and Chef `require_recipe` are always put at the top of a Recipe, just after comments, and before globals and constants.


    ```Ruby
    # Good
    include 'right_aws'
  
    require_recipe 'apache2::mod_ssl'
  
  
    tacos = 'delicious'
  
    package 'mysql' do
      action :upgrade
    end
  
  
    # Bad
    tacos = 'delicious'
    package 'mysql' do
      action :upgrade
    end
    require_recipe 'apache2::mod_ssl'
    ```


## LWRPs
* If a Recipe contains more than a few lines of pure Ruby, you might fare better with a LWRP.
* Better yet, more than a few lines of Ruby in an LWRP might fare better as a Library.
  1. See also: [FC014: Consider extracting long ruby_block to library](http://acrmp.github.com/foodcritic/#FC014)

* Treat an LWRP's Resource definition as an analog of a method envelope.

    ```ruby
    # Pure-ruby example of a method envelope:
    def my_method(food='taco', price=2):
      puts "Here's a delicious #{food} for $#{price}!"
  
  
    # Pseudo-equivalent Resource definition in Chef:
    attribute :food, :kind_of => String, :required => false, :default => 'taco', :regex => /\w+/
    attribute :price, :kind_of => [Integer, Float], :required => false, :default => 1
    ```
  

## Recipes
* Use constants for Resource parameters.
  1. Imagine that traceback!
  2. `nil` will get passed to the `apache2_site` Resource? TK
  3. Flow of Resource processing will break here? TK
  4. It's easier to test for the existence of a key.


    ```ruby
    # Good
    fqdn_items = data_bag_item('servers', 'fqdn')
    web_fqdn = fqdn_items['web_fqdn']
    
    apache2_site 'main website' do
      action :enable
      server_name web_fqdn
    end
  
  
    # Bad
    fqdn_items = data_bag_item('servers', 'fqdn')
    
    apache2_site 'main website' do
      action :enable
      server_name fqdn_items['web_fqdn']
    end
    ```

* Organize a Resources parameters for easy program flow interpretation.
  1. Readers can quickly determine whether the Resource will run, and on what conditions it will run.

    ```ruby
    # Good
    some_resource 'some_name' do
      action :some_action
      not_if{ some_condition }
      param1 some_paramater1
      param2 some_paramater2
    end
  
  
    # Bad
    some_resource 'some_name' do
      param1 some_paramater1
      param2 some_paramater2
      action :some_action
      not_if{ some_condition}
    end
    ```


* Include conditionals within a Resource.
  1. See also: [FC023: Prefer conditional attributes](http://acrmp.github.com/foodcritic/#FC023)
  2. See also: [Recipe Resources: Conditional Execution](http://wiki.opscode.com/display/chef/Resources#Resources-ConditionalExecution)
  3. What gets logged TK
  4. DRY approach

    ```Ruby
    # Good
    service 'apach2' do
      action [:enable, :start]
      only_if{ node['webserver'] }
    end
  
  
    # Bad
    if node['webserver']
      service 'apache2' do
      action [:enable, :start]
      end
    end
    ```

* Prefer `Chef::Log` over `log()`.
  1. Line shows up twice in the log TK
  2. `log()` is not usable within LWRPs.

    ```ruby
    # Good
    Chef::Log.info("Hey look, I'm a webserver!")
  
  
    # Bad
    log "Hey look, I'm a webserver!"
    ```


* Do not use `log` or `Chef::Log` within a Resource.
  1. Chef will already log when it's collecting a processing a Resource.

    ```ruby
    # Good
    service 'apache2' do
      action [:enable, :start]
    end
  
  
    # Bad
    service 'apache2' do
      action [:enable, :start]
      Chef::Log.info('Enabling apache2 service.')
    end
    ```

* Don't use static Unix-style paths.
  1. Where's `/etc/ssl` on NTFS? :)

    ```ruby
    # Good
    ETC_SSL = ::File.join(::File::SEPARATOR, 'etc', 'ssl')
  
  
    # Bad
    ETC_SSL = '/etc/ssl'
    ```


## Hints
* You can use `$globals` to pass constants around Recipes. TK
* Why put on 5 lines what you can put on one, for under 80 characters!
  1. File this under personal preference. TK

    ```ruby
    # Good
    gem_package('right_aws'){ action :nothing }.run_action(:install)
  
  
    # Bad
    g = gem_package 'right_aws' do
      action :nothing
    end
    g.run_action(:install)
    ```


* If logging a Hash, use it's built-in `inspect()` method. This works for Node Mashes also.

    ```ruby
    # Good
    Chef::Log.debug("This Node's EC2 information is #{node['ec2'].inspect}")
  
  
    # Bad
    Chef::Log.debug("This Node's EC2 information is #{node['ec2']")
    ```
