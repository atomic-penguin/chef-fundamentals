# More Cookbooks

Section Objectives:

* Download additional cookbooks with knife
* Apply multiple cookbooks with a role
* Override a cookbook attribute from a role
* Create another new cookbook

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Download Cookbooks

Use knife to download additional cookbooks from the Opscode Chef
Community site.

    > knife cookbook site download yum
    > knife cookbook site download yumrepo
    > knife cookbook site download git 
    > knife cookbook site download oracle

# Extract Cookbooks

Knife downloads the cookbook from the community site as a `.tar.gz`
file. The file name will be the cookbook name, with the version. For
example, "`oracle-0.0.9.tar.gz`".

    > knife cookbook site download oracle
    Downloading oracle from the cookbooks site at version
    0.0.9 to $PWD/oracle-0.0.9.tar.gz
    Cookbook saved: $PWD/oracle-0.0.9.tar.gz
    > tar -zxvf oracle-0.0.9.tar.gz -C cookbooks/
    > ls cookbooks/oracle

# Use Your VCS

Knife currently integrates with Git in the "cookbook site install"
command.

Git should alert you to any merge conflicts, which may need
corrected before you can push these back to the MU Chef repository.

# Examine the Cookbooks

Remember, you're probably going to run Chef on the nodes as a
privileged user. After extracting the cookbooks, examine their
contents to see what they're doing.

Most cookbooks have a README file that describe what the purpose of
the cookbook is, what it does and how to use it.

Chef doesn't actually execute anything on the system unless it is
in a recipe. Start by looking at the recipes that are in the
cookbooks so you know what they're going to do.

# Upload cookbooks

Once code has been reviewed, upload the cookbooks to the server. All
cookbooks can be uploaded at one time with the `-a` switch.  However,
the safest and quickest option would be to upload just the cookbooks you
wish to change or update.

    > knife cookbook upload -a
    Uploading apache2                 [1.0.8]
    Uploading apt                     [1.3.0]
    Uploading chef-client             [1.0.4]
    Uploading fail2ban                [1.0.0]
    Uploading webserver               [1.0.0]
    upload complete

# Recipe Ordering

Recipes applied to a node are processed in the order specified.

Recipes are specified in a Role or Node run list.

Run lists are ordered lists (Ruby/JSON arrays).

# Recipe Ordering

Recipes can include other recipes, too. Use "`include_recipe`".

Chef will process the included recipe in place, adding its resources
to the resource collection.

Included recipes from other cookbooks require metadata dependency.

# Cookbook Dependencies

Remember, cookbook dependencies are assumed when using part(s) of one
cookbook in another, such as recipe inclusion.

Cookbook dependencies are explicitly defined in metadata. Use the
"depends" keyword. This will cause Chef to download the dependency
cookbook from the server.

Downloading a cookbook as a dependency from another does not cause it
to be applied on the node. It merely makes the code/contents available
for another cookbook to use it.

# Cookbook Metadata

The metadata defines the additional cookbooks required that might not
appear in the run list explicitly.

The server reads the metadata as JSON, it does not parse the recipes
(which are Ruby).


# Using Roles

We typically apply multiple related recipes to a node via a role.

Our `base5` role is a great place to apply things that need to be done
for all systems.

# Sample base Role

    @@@ruby
    name "base5"
    description "Base role is applied to all systems"
    run_list(
      "recipe[chef-client]",
      "recipe[yumrepo]",
      "recipe[users::itisystems]"
    )

# Order Matters

We put `yumrepo` before other cookbooks in the list to make sure the
system package cache is updated for package resources.

For example, the default action for all of the package resources in the
MU chef-repo is install.  The yumrepo cookbook will ensure we always have
the latest version of Dell, VMWare, or EPEL packages available to Chef.

# Apply Role to Node

Applying the role to the node can be done with knife.

    > knife node run list add NODE 'role[base5]'

However, this appends the role to the end of the node's run list. To
insert it before another item, you need to edit the node directly.

    > knife node edit NODE

The `$EDITOR` environment variable must be set, or specified with
`knife node edit`'s `-e` or `--editor` option.

    > knife node edit NODE -e vi

# Apply role to Node

    @@@javascript
    {
      "run_list": [
        "role[base5]",
        "role[java]"
      ]
    }

# Run Chef

Running Chef on the target node will cause the cookbooks in the role's
run list to be downloaded just like when only a recipe is in the
node's run list.

    > sudo chef-client
    INFO: *** Chef 0.10.8 ***
    INFO: Run List is [role[base5], role[java]]
    INFO: Run List expands to [inittab, users::itisystems, ...]

# Common Patterns

Many things automated with Chef follow a pattern:

* Install a software package.
* Write configuration files.
* Enable and start a service.

Let's walk through another example of this pattern.

# SNMP Cookbook

Download the snmp cookbook.

    > knife cookbook site download snmp
    > tar -zxvf haproxy-0.2.1.tar.gz -C cookbooks

We will explore the snmp cookbook for this pattern because we'll
revisit it in the next section on search.

# snmp default recipe

The default recipe follows this simple pattern, first install the packages.

    @@@ruby
    node['snmp']['packages'].each do |snmppkg|
      package snmppkg
    end    

# snmp default recipe

This part of the recipe drops off any plugins or supplemental
config files, if any are defined.

    @@@ruby
    if not node['snmp']['cookbook_files'].empty?
      node['snmp']['cookbook_files'].each do |snmpfile|
        cookbook_file snmpfile do
          mode 0644
          owner "root"
          group "root"
        end
      end
    end

# snmp default recipe

This part of the recipe enables the service to start at boot,
and starts the snmp service at this time.

    @@@ruby
    service node['snmp']['service'] do
      action [ :start, :enable ]
    end

# snmp default recipe

Finally, the recipe renders the main configuration file for snmpd.
When doing so, the template resource notifies the service to restart,
loading the new configuration file.

    @@@ruby  
    template "/etc/snmp/snmpd.conf" do
      mode 0644
      owner "root"
      group "root"
      notifies :restart, resources(:service => node['snmp']['service'])
    end

# snmp default attributes

The default attributes file sets platform-specific attributes for
the recipe code, and template.

    @@@ruby
    case node['platform']
    when "redhat","centos","fedora","scientific"
      set['snmp']['packages'] = ["net-snmp", "net-snmp-utils"]
      set['snmp']['cookbook_files'] = Array.new
    end

    default['snmp']['service'] = "snmpd"

# snmp template

The default template makes use of node attributes, and ohai attributes
to render the correct snmpd.conf

    @@@ruby
    com2sec notConfigUser  default       <%= node[:snmp][:community] %> 
    group   notConfigGroup v1            notConfigUser
    group   notConfigGroup v2c           notConfigUser
    <% if node[:snmp][:full_systemview] %>
    view    systemview    included   .1
    <% end %>
    view    systemview    included   .1.3.6.1.2.1.1
    view    systemview    included   .1.3.6.1.2.1.25.1.1
    access  notConfigGroup ""      any       noauth    exact  systemview none none
    <% if node[:virtualization][:role] == "guest" %>
    syslocation <%= node[:snmp][:syslocationVirtual] %>
    <% else %>
    syslocation <%= node[:snmp][:syslocationPhysical] %>
    <% end %>

# Summary

* Download additional cookbooks with knife
* Apply multiple cookbooks with a role
* Override a cookbook attribute from a role
* Create another new cookbook

# Questions

* What knife command is used to download cookbooks from the Chef
  Community site?
* What is the first thing one should do after downloading a cookbook?
* How can all cookbooks be uploaded at one time?
* How does Chef determine what recipes to apply on a node?
* How does Chef determine what order to apply recipes on a node?
* How does Chef determine what cookbooks to download?
* Can Chef download cookbooks for a node that aren't in its run list?
* What is a common recipe pattern used in many cookbooks?

# Lab Exercise

More Cookbooks

* Apply snmp, yumrepo::default, and any other recipe via cheftrain role
* Download and examine the haproxy cookbook
