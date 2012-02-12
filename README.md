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

TK

## Resource Usage

* Include conditionals within a Resource.

  ```Ruby
  # Yes
  service 'apach2' do
    action [:enable, :start]
    not_if{ node['webserver'] }
  end

  # No
  if node['webserver']
    service 'apache2' do
      action [:enable, :start]
    end
  end
  ```

  Why:

  # what gets logged TK
  # DRY approach
 
* Prefer ```Chef::Log``` over ```log()```.

  ```Ruby
  # Yes
  Chef::Log.info("Hey look, I'm a webserver!")
  
  # No
  log "Hey look, I'm a webserver!"
  ```

  Why:

  # Line shows up twice in the log TK
  # ```log()``` is not usable within LWRPs.


  

