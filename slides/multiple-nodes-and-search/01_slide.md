# Multiple Nodes And Search

Section Objectives:

* Chef search indexes
* Search using knife
* Search in a recipe
* Knife bootstrap and ssh
* Integrating systems with search

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Search

The Chef Server provides a powerful full text search engine.

The search engine is based on Apache SOLR.

The search query language is modified SOLR Lucene.

# Default Search Indexes

Four search indexes are created on the Chef Server by default.

* node
* client
* role
* environment

# Search Indexes

When data bags are created, a search index is also created, and the
index is the same name as the bag.

Data Bags are covered in detail later.

# Knife Search

Search returns the entire object found on the server, but the output
is abbreviated.

* -m will display "normal" node attributes in the output.
* -l will display all node attributes in the output.
* -a ATTRIBUTE will display only the selected attribute.
* -i will display only the ID of the matching items.
* For nodes, -r will display the run list of the node.

# Knife Search

    knife search node "platform:centos"
    knife search node "platform:redhat" -r
    knife search node "role:cheftrain"

# Query Syntax

Queries follow a basic pattern, with Knife:

    knife search INDEX "field:pattern"

In a recipe:

    search(:INDEX, "field:pattern")

Search patterns can be an exact match, range match or a wildcard match.

# Query Syntax: Exact Match

    knife search node "role:cheftrain"
    knife search node "ipaddress:10.1.1.26"
    knife search node "kernel_machine:x86_64"

Search for a node with an attribute set to a particular value.

Sub-key attributes (`node["kernel"]["machine"]`) are flattened with underscores.

# Query Syntax: Ranges

    knife search node "cpu_total:[4 TO 8]"
    knife search node "cpu_total:{4 TO 8}"

Range searches are inclusive with square brackets [].

Range searches are exclusive with curly braces {}.

# Query Syntax: Wildcards

    knife search node "hostname:mu*"
    knife search node "fqdn:mu*.marshall.edu"
    knife search node "hostname:*"
    knife search node "*:*"

Search for all nodes with hostname starting with "mu".

Search for all nodes with fqdn that starts with mu and ends with .marshall.edu

Search for all nodes that have a hostname at all.

Search for all nodes.

# Query Syntax: Boolean

    knife search node "(NOT cpu_total:1)"
    knife search node "role:cheftrain AND chef_environment:test"
    knife search node "cpu_total:2 OR kernel_machine:x86_64"

Search for all nodes except those with 1 CPU.

Search for all production web servers using the chef_environment.

Search for all nodes that have 2 CPUs or are 64bit.

# Recipe Search

You can run a search and retrieve the results, assigning them to a
local variable.

Or, you can run a search and iterate over the results dynamically.

# Assign Results to Variable

    pool = search(:node, "role:cheftrain")

`pool` will be an array of the JSON representation of all the node
objects that match the search.

We can then use this node data in the recipe to configure resources
dynamically.

.notes that the entire object is returned in the results can be a
significant use of system memory

# Assign Results to Variable

Sometimes we only want specific attributes of nodes that match the
search query. As the `search` results are an array, we can generate
another array based on a specific condition using the `#map` method
(from Enumerable).

    ip_addrs = search(:node, "role:cheftrain").map {|n| n["ipaddress"]}

`ip_addrs` will be an array of the `ipaddress` attribute from all the
node objects that match the search. Contrast to `pool` from before which
contained the entire node objects.

# Iterating Over a Search

We can iterate over the search results in real time because search
takes a block as a parameter.

    @@@ruby
    search(:node, "role:cheftrain") do |match|
      puts match["ipaddress"]
    end

Provide a Ruby block to process the results directly. We could do
other things such as creating a resource for each result and pass the
data from the objects returned into the resource's paramter attributes.

# Common Search Usage

Search is used for a wide variety of purposes in Chef. Common uses
are:

* load balancers that need to pool a number of web servers.
* application servers that need to find the master database server.
* monitoring tools that need to find all the nodes of a particular
  type for custom monitoring.

# Knife Sub-commands

Knife has two built in subcommands that use search.

* knife status
* knife ssh

# Knife Status

Knife's status subcommand performs a search and prints out information
about all the nodes, e.g.:

    > knife status
    1 hour ago, www1, www1.example.com, 10.1.1.20, centos 6.2.
    1 hour ago, www2, www2.example.com, 10.1.1.21, centos 5.7.

By default it searches for all the nodes, but you can pass it a
query like with knife search.

    > knife status "role:cheftrain"

# Knife SSH

`knife ssh` is a handy method for running a single command on several servers at once, in parallel.
This knife subcommand uses a SOLR search string to select nodes on which to operate.

For example, to check the cheftrain servers for updates, run the following.  Substituting your own
ssh username with the `-x` switch.

    @@@sh
    > knife ssh -x wolfe21-a "role:cheftrain" "sudo yum check-update"

# Knife SSH

This comes in handy to update all servers by chef_environment at once.

    @@@sh
    > knife ssh -x wolfe21-a \
    "chef_environment:test AND fqdn:cheftrain01.marshall.edu" \
    "sudo yum -y upgrade"

# Summary

* Chef search indexes
* Search using knife
* Search in a recipe
* Knife bootstrap and ssh
* Integrating systems with search

# Questions

* What is the search query language used by Chef?
* What are two search indexes are created by default?
* How can node searches display only the run list of the results?
* What are two other knife subcommands that use search?

# Additional Resources

* [http://wiki.opscode.com/display/chef/Search](http://wiki.opscode.com/display/chef/Search)
* [http://wiki.opscode.com/display/chef/Knife+Built+In+Subcommands](http://wiki.opscode.com/display/chef/Knife+Built+In+Subcommands)

# Lab Exercise

Multiple Nodes And Search

* Use knife search to find all servers with a particular Ohai attribute like platform:centos.
* Use knife ssh to rerun chef client.
* Use knife ssh to check for, and apply, updates on your assigned machine.
