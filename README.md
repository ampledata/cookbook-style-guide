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


# Alternative Sources

Alternative guidelines are available at:

* Opscode's [Cookbook Style Guide Draft](http://wiki.opscode.com/display/chef/Cookbook+Style+Guide+Draft)
* Opscode's [Cookbook Style-Guide Outline](http://wiki.opscode.com/display/chef/Cookbook+Style-Guide+Outline)

**TODO**: Integrate Opscode's guides into this one.

# Chef Cookbook Style Guide
## Recipe Organization
* Layout a Recipe as you would a [Ruby program](https://github.com/bbatsov/ruby-style-guide)
* Ruby `require` statements should be placed at the top of a Recipe,
  just after comments.
* Ruby constant, global and local variable declarations should be placed 
  immediately following `require` statements, separated by a empty line.
* Chef statements & Resources should be placed below all other Ruby
  statements.

    ```Ruby
    # Recipe:: my_good_example
    # Cookbook Name:: my_cookbook
  
    # Ruby require statement.
    require 'right_aws'

    # Ruby variable declaration:
    apache_config = '/etc/apache.conf'

    # Chef Resource
    file apache_conf do
      action :create
    end
    ```


## LWRPs
* If a Recipe contains more than a few lines of pure Ruby, you might fare 
better with a [Lightweight Resource Provider
(LWRP)](http://wiki.opscode.com/display/chef/Lightweight+Resources+and+Providers+%28LWRP%29).
* Better yet, more than a few lines of Ruby in an LWRP might fare better as a 
Library.
  1. See also: [FC014: Consider extracting long ruby_block to library](http://acrmp.github.com/foodcritic/#FC014)

* Treat an LWRP's Resource definition as an analog of a Ruby method.

    ```ruby
    # Example of a Ruby method:
    def my_method(food='taco', price=2):
      puts "Here's a delicious #{food} for $#{price}!"
  
  
    # Example of a Resource definition in Chef:
    attribute :food, :kind_of => String, :default => 'taco'
    attribute :price, :kind_of => [Integer, Float], :default => 1
    ```
  

## Recipes
* Pass one-dimensional Ruby structures to Resources.
  1. Avoid passing in complex structures, such as Hashes or Arrays.
  2. It's easier to test for the existence of a key from the Recipe
     before instantiating a Resource Provider.

    ```ruby
    # Good, defensible example:
    fqdn_items = data_bag_item('servers', 'fqdn')
    web_fqdn = fqdn_items['web_fqdn']
    
    apache2_site 'main website' do
      action :enable
      server_name web_fqdn
    end
  
  
    # Bad, ambiguous example:
    fqdn_items = data_bag_item('servers', 'fqdn')
    
    apache2_site 'main website' do
      action :enable
      server_name fqdn_items['web_fqdn']
    end
    ```

* Organize a Resources parameters for easy program flow interpretation.
  1. Readers can quickly determine whether the Resource will run, and on what 
  conditions it will run.

    ```ruby
    # Good, easy to grok example:
    some_resource 'some_name' do
      action :some_action
      not_if{ some_condition }
      param1 some_paramater1
      param2 some_paramater2
    end
  
  
    # Bad, several eye scans required:
    some_resource 'some_name' do
      param1 some_paramater1
      action :some_action
      param2 some_paramater2
      not_if{ some_condition }
    end
    ```


* Include conditionals within a Resource.
  1. See also: [FC023: Prefer conditional attributes](http://acrmp.github.com/foodcritic/#FC023)
  2. See also: [Recipe Resources: Conditional Execution](http://wiki.opscode.com/display/chef/Resources#Resources-ConditionalExecution)
  3. [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) approach.


    ```Ruby
    # Good
    service 'apache2' do
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

* Prefer `Chef::Log` over `log`.
  1. Avoids ambiguity of `log` Resource getting collected and executed
     (e.g. two log lines).
  2. `log` is not usable within LWRPs.
  3. `log` writes to the log during convergence, not compilation; use `log` in a Recipe when you want to declare a log entry during compilation but trigger the log entry during convergence; use `Chef::Log` when you want to log normally.


    ```ruby
    # Good
    Chef::Log.info("Hey look, I'm a webserver!")
  
  
    # Bad
    log "Hey look, I'm a webserver!"
    ```

Log of 'Bad' usage:

    ```
    INFO: *** Chef 10.12.0 ***
    INFO: Processing log[Hey look, I'm a webserver!] action write (style-guide::default line 9)
    INFO: Hey look, I'm a webserver!
    INFO: Chef Run complete in 0.004116 seconds
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
* You can use `$globals` to pass constants around Recipes.

    ```Ruby
    # Example, where configure_web will use the $web_url global we've defined in a prior Recipe.

    $web_url = 'http://example.com'

    include_recipe 'www::configure_web'
    ```

* Why put on 5 lines what you can put on one, for under 80 characters!
  1. File this under personal preference.

    ```ruby
    # Good, one line concise Resource:
    gem_package('right_aws'){ action :nothing }.run_action(:install)
  
  
    # Bad, multiple lines when could easily condense to one:
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

# Cookbook Style Guide Metadata

## Authors
* [Greg Albrecht](https://ampledata.org) [@ampledata](http://twitter.com/ampledata)

## Copyright
Copyright 2012 Greg Albrecht

## License
[GNU Free Documentation License version
1.3](http://www.gnu.org/copyleft/fdl.html)

## Version
1.1.0
