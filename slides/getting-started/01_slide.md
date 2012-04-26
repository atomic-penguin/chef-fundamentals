# Getting Started

Section Objectives

* Install Ruby and Chef
* Get familiar with the tools that come with Chef
* Set up connectivity to a Chef Server
* Create an initial Chef Repository

.notes These course materials are Copyright 2012 Eric G. Wolfe and 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Supported Platforms

MU Systems staff have tested and ran chef-client, or knife, on:

* Ubuntu (10.04, 10.10, 11.04, 11.10)
* RHEL & CentOS (5.x, 6.x)
* Windows 7 / 2008

.notes chef-client is the config management component, knife is the command and control.

# Ruby and Chef

Chef is written in Ruby, an interpreted object oriented programming language.

Much of the code you'll work with for Chef is Ruby, albeit domain-specific language(s) suited for the task.

In order to install and use Chef, we need to have Ruby installed and available. It is important to understand some nuances about Ruby installation, first.

# Ruby Versions

Ruby is an interpreted language and has several interpreter VMs available. The most common is "MRI" or the "Matz Ruby Interpreter"

The latest stable version of MRI is [Ruby 1.9.3](http://ruby-lang.org).

Chef requires at least Ruby 1.8.7.

.notes Versions are current as of the latest update to these materials.

# Platform Packages

Ruby and parts of the Ruby ecosystem have a reputation for backwards incompatibility. As such, not all platforms have the latest version available as the default package for "ruby".

Most Linux/Unix variants package version 1.8.7; it is an API-compatible transition version between 1.8 and 1.9.

Some make 1.9 version(s) available, but few use it as the default version.

.notes Debian (wheezy/testing) and Ubuntu (oneiric+) use their alternatives system to make ruby1.9.1 available to be used as a default ruby installed as an OS package.

# RubyGems

Ruby software and libraries are usually published as Gems, using the Ruby packaging system *RubyGems*. Gems are published to http://rubygems.org/

Prior to Ruby 1.9, RubyGems was separately installed. It has been added to Ruby 1.9 in the standard library, at RubyGems version 1.3.7.

Chef requires at least RubyGems 1.3.7.

.notes The `gem_package` provider uses library/API calls that were introduced in RubyGems 1.3.7. Some instructions or automated installations of Chef install newer versions of RubyGems by default.

# Installing Ruby and RubyGems

Chef is distributed primarily as a RubyGem, therefore Ruby and RubyGems need to be installed. Assuming the version requirements are met, we can simply install the Chef gem:

    % sudo gem install chef

# Were it that simple...

For some platforms, getting the required versions of Ruby and RubyGems for Chef is not trivial.

The current version of Ruby not being packaged by default has lead to other solutions, primarily centered around installing Ruby from source, to emerge.

Compiling from source is time consuming, and many users prefer to install all software using packages.

# Omnibus Build System

Opscode created the package build system "omnibus" to generate full-stack binary installers for Chef in a variety of formats.

* Native packages (rpm, deb)
* Self-extracting executable Tar.gz
* Microsoft Software Installer (MSI)

The idea is that all the software required above the standard C library for the OS is included. In Chef's case this is autoconf, openssl, zlib, Ruby, RubyGems and more.

.notes Omnibus is the name of the project for the build tool. These packages are often called "omnibus" packages. It is currently used for Chef, but can build other software stacks, too.

# Installing Chef Full

Omnibus is the build tool. The installation packages are called "chef-full".

Instructions for installation are at:

* [http://opscode.com/chef/install](http://opscode.com/chef/install)

# Linux/Unix Installation

Installation on Linux and Unix is done with a shell script:

    % curl -L http://opscode.com/chef/install.sh | sudo bash

The script detects the platform of the system to determine what installation file to download.

# Windows Installation

Installation on Windows is done by downloading the MSI and installing it.

* [http://opscode.com/chef/install.msi](http://opscode.com/chef/install.msi)

Server versions are directly tested, but the MSI is known to install and work fine on desktop versions of Windows such as Vista and 7.

# What You Get: Linux/Unix

Chef binaries are in `/opt/opscode/bin`. The package installation symlinks them in `/usr/bin` so they are in the default `$PATH`.

Other binaries are in `/opt/opscode/embedded/bin`. This includes `ruby`, `gem`, and binaries for other included software like autoconf, openssl and more.

# What You Get: Windows

Chef binaries are in `C:\opscode\bin`. The other binaries are in `C:\opscode\embedded\bin`.

These directories are both added to the system `%PATH%`.

# Chef Toolbox

Chef comes with several tools of its own.

* ohai
* chef-client and chef-solo
* knife
* shef

# Other Tools

In working with infrastructure as code, we may need several tools in our toolbox.

Non-Chef tools:

* Shell (Bash, Powershell, Cmd.exe)
* Text editors (Vim, Joe, Notepad++)
* Version control system (Git)
* Ruby programming language

# Chef Tools

Each of the tools bundled with the Chef Full package share some common traits.

* Built-in help
* Configuration

# Built-in Help

Each command has a `--help` or `-h` command-line option that displays options and contextual help output.

Each command also has a corresponding Unix `man(1)` page, which is included in the installed Chef library.

Since Windows does not have a `man(1)` help system, the recommended form of documentation on this platform
is the Opscode Wiki at [http://wiki.opscode.com](http://wiki.opscode.com).

# Configuration

Each tool that Chef comes with has its own configuration file.

Configuration files populate values in the `Chef::Config` object.

Chef comes with sane default values for all configuration options.

Context of the configuration file to the appropriate tool is important.

# Chef Configuration

`Chef::Config` uses a simple domain specific language where the setting and its value are specified.

    @@@ruby
    log_level :info
    log_location STDOUT
    chef_server_url "http://muchef.marshall.edu:4000"

# Ohai

`Ohai` is a standalone library that is installed with Chef as a dependency.

`Ohai` uses plugins to profile the local system when Chef runs to gather
information.

When `chef-client` runs, this data gets stored on the Chef Server as `node attributes`.

# Ohai Configuration

Ohai is usually configured via the chef-client configuration file
(`/etc/chef/client.rb`). It has no other configuration file.

The ohai configuration must be modified with `Ohai::Config` object. It is a Ruby hash that uses symbols for key names.

    @@@ruby
    Ohai::Config[:disabled_plugins] = 'passwd'

.notes Disabling plugins is the most common configuration to do for Ohai. We won't detail more configuration at this time.

# Knife

Knife is the "swiss army knife" of infrastructure management tools.

* manage the local Chef repository
* interact with the Chef Server API
* interact with cloud computing APIs
* extensible with custom plugins/libraries

# Knife Sub-commands

Knife plugins are used as sub-commands. General format of knife sub-commands:

    knife COMMAND verb noun (options)

This is consistent for Chef API, but some differences across other uses.

# Knife Command Examples

    knife node show NODENAME
    knife cookbook upload fail2ban
    knife role edit webserver

# Knife Contextual Help

    knife --help
    knife sub-command --help
    knife sub-command verb --help

# Knife Man Pages

Knife has built-in man pages.

    knife help
    knife help list
    knife help knife
    knife help TOPIC
    knife help node

.notes These probably won't work on Windows, which doesn't have a "man" system. The files are also rendered as html and can be found in the Chef MSI.

# Knife Configuration

The default configuration file for Knife is `.chef/knife.rb`; knife looks for it automatically, similar to `git`:

    $PWD/.chef/knife.rb
    $PWD/".."/.chef/knife.rb
    ~/.chef/knife.rb

The MU Chef repo provides a pregenerated `knife.rb` you can use.

# Knife Configuration Options

Knife configuration uses `Chef::Config`.

Knife also has its own specific configuration for various plugins to
use. These are in `Chef::Config[:knife]`, which is a hash of
configuration options.

.notes Configuration options in knife are like other parts of Chef,
the data is a hierarchy of key/value pairs, and where we are in the
hierarchy is context specific.

# Knife Configuration

To work with the Chef Server API, Knife must be configured with:

* The Chef Server's URL (chef\_server\_url).
* The user to authenticate to the API (node\_name).
* The private key for the authenticating user (client\_key).

.notes In the API, users (as knife) and API clients (like chef-client)
are "actors"

# Knife Configuration

Minimal `knife.rb` configured to use MU Chef server with MUNET username.

    @@@ruby
    node_name       user 
    client_key      "#{ENV['HOME']}/.chef/#{user}.pem" 
    chef_server_url "http://muchef.marshall.edu:4000"

.notes We will talk more about how the authentication system works in Chef later in Anatomy of a Chef Run

# Knife User

The configuration value `node_name` in the `knife.rb` refers to a Chef user.

Users are global to the entire Chef service.

# Knife Configuration

We work with a local Chef Repository that stores the various files, including cookbooks, that should be sent to the Chef Server. Knife is written to automatically use these locations.

One directory it does need to be told is the path where cookbooks live.

    @@@ruby
    cookbook_path "./cookbooks"

This configuration file is Ruby, not bash, so we need to be careful with the path usage. The default `knife.rb` provided within the MU Chef repo will have this already set for you.

# Knife Configuration

The cookbook path in the MU Chef repo Knife configuration file uses a relative location based on the location of the `knife.rb` file.

    @@@ruby
    current_dir = File.dirname(__FILE__)
    cookbook_path ["#{current_dir}/../cookbooks"]

# Knife Command-line Options

Knife has a base set of command-line options that correspond to general options in the `knife.rb`.

Each sub-command may have specific command-line options that are different.

# Knife Command-line Options

The following command-line options correspond to the config file settings seen above:

    | Command-line Option  | knife.rb        |
    |----------------------|-----------------|
    | -s, --server-url URL | chef_server_url |
    | -k, --key KEY        | client_key      |
    | -u, --user USER      | node_name       |

# Knife Command-line Options

Other common command-line options used with knife.

    | Command-line Option  | Purpose             |
    |----------------------|---------------------|
    | -c, --config CONFIG  | Configuration file  |
    |                      | to use.             |
    | -V, --verbose        | Verbose output, can |
    |                      | be specified twice. |
    | -F, --format         | Output format, can  |
    |                      | be json, yaml, text |

.notes We'll discuss other options as we progress through the course.

# Chef Client & Chef Solo

The programs `chef-client` and `chef-solo` load the Chef library and make it available to apply configuration management with Chef.

Both programs know how to configure the system given the appropriate recipes found in cookbooks.

# Chef Client

chef-client talks to a Chef Server API endpoint, authenticating with an RSA key pair. It retrieves data and code from the server to configure the node per the defined policy.

List of recipes can be predefined, assigned to a node on the Chef
Server, and retrieved when chef-client runs.

The default configuration file is `/etc/chef/client.rb`.

# Chef Client Configuration

Like Knife, `chef-client` must be configured with the proper authentication information to connect to the Chef Server.

Unlike Knife, the `node_name` is not a user, but the actual system's name for itself. Unless otherwise specified in `/etc/chef/client.rb`, the `node_name` is the value detected by Ohai as the `fqdn` (fully qualified domain name).

.notes We will talk more about authentication in Anatomy of a Chef run.

# Chef Client Configuration

The minimal configuration for `chef-client` in `/etc/chef/client.rb` to talk to the Chef Server:

    @@@ruby
    chef_server_url  "http://muchef.marshall.edu:4000"
    validation_client_name "chef-validator"

All other options will use default values, which are meant to be sane defaults.

# Chef Client Configuration

Other common configuration options (default values are used below):

    @@@ruby
    log_level        :info
    log_location     STDOUT
    verbose_logging  true
    file_cache_path  "/var/chef/cache"
    file_backup_path "/var/chef/backup"
    json_attribs     nil

# Client Command-line Options

The following command-line options correspond to the specified config file settings.

    | Command-line Option  | client.rb       |
    |----------------------|-----------------|
    | -S, --server URL     | chef_server_url |
    | -k, --key KEY        | client_key      |
    | -u, --user USER      | node_name       |

# Client Command-line Options

Other common command-line options used with `chef-client`:

    | Command-line Option   | client.rb    |
    |-----------------------|--------------|
    | -l, --log_level LEVEL | log_level    |
    | -L, --logfile LOGFILE | log_location |
    | -j JSON_ATTRIBUTES    | json_attribs |
    | -N, --node-name NAME  | node_name    |

# Client Command-line Options

The following options control how the `chef-client` process behaves.

    | Command-line Option   | Purpose                        |
    |-----------------------|--------------------------------|
    | -d, --daemonize       | Daemonize the process          |
    | -i, --interval INT    | Run every INT seconds          |
    | -s, --splay SECONDS   | Random splay added to interval |

# Full example client.rb

    @@@ruby
    log_level        :info
    log_location     STDOUT
    chef_server_url  "http://muchef.marshall.edu:4000"
    validation_client_name "chef-validator"
    # Using default node name

# Command-line Options Override

Options passed on the command-line override values in the configuration file.

    @@@sh
    chef-client -l debug
    chef-client -N node-name
    chef-client -S http://muchef.marshall.edu:4000
    chef-client --help

# Chef Solo

chef-solo operates without a Chef Server. It requires that all the
recipes it needs are available, and that it be told what to run on the
node.

A JSON file is passed to chef-solo to give it these instructions in a
`run_list` for the node.

The default configuration file is `/etc/chef/solo.rb`.

# Solo Command-line Options

`chef-solo` does not connect to a server, so it doesn't have options for interacting with a server. Common command-line options:

    | Command-line Option   | solo.rb       |
    |-----------------------|---------------|
    | -l, --log_level LEVEL | log_level     |
    | -L, --logfile LOGFILE | log_location  |
    | -j JSON_ATTRIBUTES    | json_attribs  |
    | -N, --node-name NAME  | node_name     |
    | -r, --recipe-url URL  | recipe_url    |

These are similar to `chef-client`, with the addition of `-r`.

# Shef

Shef is an interactive Ruby console that supports attribute and recipe contexts, as well as interactive debugging features.

Shef can be configured to talk to a Chef Server to interact with the API directly.

In depth use of Shef is beyond the scope of this class, but we may explore it for examples later.

The default configuration file is `~/.chef/shef.rb` but an alternate can be passed with `-c`.

# Chef Server

The Chef Server is a centralized publishing system for infrastructure data and code.

* Stores node, role and user-entered data
* Data is indexed for search
* Stores cookbooks
* Provides an API for management and discovery


# Chef Server Components

<center><img src="../images/chef-server-arch.png"></center>

# Chef Server API Services

API Server (HTTP & JSON, Authentication)

Data storage (JSON documents, Cookbooks)

Message queue (Search indexing, other services)

Search Engine (Full text search)

Web management console (API client)

# Chef Server Security

All communication is initiated by API clients, and never from the Chef Server directly.

Communication is over HTTP.

All API requests are authenticated using digital signatures.

Custom data ("data bags") can be encrypted with user-supplied keys.

# Chef management workstation preparation

  1. Install chef-full packages
  2. Install git
  3. Install editor (optional Windows step)
  4. Generate keys for Git and Chef
  5. Edit configuration files in `$HOME` folder *(V:/ on Win)*

# Install chef-full packages Linux/Mac OS X

    @@@sh
    curl -L http://www.opscode.com/chef/install.sh | sudo bash

# Install chef-full packages Windows

 1. Go to [http://www.opscode.com/chef/install](http://www.opscode.com/chef/install)
 2. Select the `Windows Server` package, which also works on Vista and 7.
 3. Click through the install wizard accepting the default options

# Install Git Linux

    @@@sh
    apt-get install git # Debian / Ubuntu
    yum install git # RHEL / CentOS

# Install Git Windows / Mac OS X

 1. Go to [http://git-scm.com](http://git-scm.com)
 2. Follow Mac OSX or Windows* link to Google code for appropriate DMG or EXE package.

 * The latest `Full installer` msysGit package is recommended for Windows

# Install editor (optional for Windows)

You can use the basic notepad.exe as a source editor on Windows.

If you want a syntax highlighting source editor, then install something like Notepad++.

* [http://notepad-plus.sf.net](http://notepad-plus.sf.net)

# Configure Windows to show file extensions

 1. Open `Control Panel`
 2. Select `Appearance and Personalization`
 3. Click `Show hidden files and folders`
 4. Uncheck the box next to `Hide extensions for known file types`

<center><img src="../images/ctrlpanel-hide-extensions.png"></center>

# Log in to the MU Chef server 

Go to [http://muchef.marshall.edu:4040](http://muchef.marshall.edu:4040)

<center><img src="../images/muchef-signup.png"></center>

A **default training password** will be provided to users with new accounts.

# Test your login on the MU Chef dashboard

You should see the `Nodes` tab when you first login to the Chef dashboard.

<center><img src="../images/muchef-login-test.png"></center>

Reset your default password by clicking the `Edit account` link.

# Create your client API key

 1. Click `Clients` tab.
 2. Click `Create` tab.
 3. Enter your username in the `Name` textbox.
 4. Check the `Admin` box.
 5. Finally click the submit button labeled `Create client`.

<center><img src="../images/create-a-knife-client.png"></center>

# Copy private key to $HOME/.chef folder

 1. Copy your `private key` to the clipboard.
 2. Create a `.chef` folder in your `home` directory.
 3. Paste your `private key` into a file named after your username, with a .pem extension *(would be myusername-a.pem in this case)*.

<center><img src="../images/client-private-key.png"></center>

.notes Windows home directory will be V:/ in most cases

# SSH keys for Git access Linux / Mac OS X

    @@@sh
    ssh-keygen -t rsa

 * E-mail a copy of your id_rsa.pub to the Instructor <wolfe21@marshall.edu>

# SSH keys for Git access Windows

 1. Open the Git GUI
 2. Click Help -> Show SSH Keys -> Generate Key
 3. E-mail a copy of your id_rsa.pub to the Instructor <wolfe21@marshall.edu>

<center><img src="../images/git-generate-sshkey.png"></center>

 * Or copy existing OpenSSH keys to `V:/.ssh`

# Clone MU Chef repo

Open `git bash` on Windows, or your preferred terminal emulator on Linux / Mac.

    @@@sh
    git clone git@git.marshall.edu:/var/git/chef.git chef-repo

 * This should clone the central MU Chef repo to a chef-repo folder in your home directory.

.notes Windows may default to V:/, be cognizant of available space on your V: folder.

# Bash .profile script in $HOME *(V:/ on Win)*

    @@@sh
    #!/bin/bash
    OPSCODE_USER=myusername-a
    ORGNAME=chef
    PATH=$PATH:/c/Program\ Files\ \(x86\)/Notepad++
    EDITOR=notepad++
    COOKBOOK_EMAIL='myemail@marshall.edu'
    COOKBOOK_COPYRIGHT='Full Name'
    export ORGNAME OPSCODE_USER EDITOR COOKBOOK_COPYRIGHT COOKBOOK_EMAIL PATH

# Test knife API client

 1. Open a new shell, or `git bash` window.
 2. Change directories to your `chef-repo` which you previously checked out.
 3. Type `knife status`.  This should return the status of managed nodes.

# Test knife API client

    $ knife status
    30 minutes ago, machine1, machine1, 192.0.2.3, redhat 5.8.
    29 minutes ago, machine2, machine2, 192.0.2.5, redhat 5.8.
    29 minutes ago, machine3, machine3, 192.0.2.2, centos 5.8.

# Workstation setup Results

* MU Chef client API key
* SSH private key
* Knife Configuration file, included in chef-repo
* Bash .profile script
* Chef-repo directory with Chef code 

# Chef Repository

Very simply, the Chef Repository is a version controlled directory that contains cookbooks and other components relevant to Chef.

It contains your "Infrastructure as Code".

Knife already knows how to interact with many parts of the repository.

We'll look at each part of the repository in greater detail when we get to the relevant section.

# Chef Repository

Example Chef Repository directory tree:

    chef-repo
    ├── .chef
    ├── cookbooks
    ├── data_bags
    ├── environments
    ├── README.md
    └── roles

# Working with Chef

<center><img src="../images/working-with-chef.png"></center>

# Summary

* Ruby and Chef Installation
* Tools and commands that come with Chef
* Connectivity to the Chef Server
* Components of a Chef Repository

# Questions

* In what language is Chef written?
* What are commands / tools that come with Chef?
* What do Chef's commands have in common?
* What configuration is required to connect to a Chef Server?
* Student questions?

# Additional Resources

* [http://wiki.opscode.com/display/chef/Resources](http://wiki.opscode.com/display/chef/Resources)
* [http://wiki.opscode.com/display/chef/Recipes](http://wiki.opscode.com/display/chef/Recipes)
* [http://wiki.opscode.com/display/chef/Chef+Repository](http://wiki.opscode.com/display/chef/Repository)
* [http://wiki.opscode.com/display/chef/Chef+Configuration+Settings](http://wiki.opscode.com/display/chef/Chef+Configuration+Settings)
* [http://wiki.opscode.com/display/chef/Server+API](http://wiki.opscode.com/display/chef/Server+API)
* [http://community.opscode.com/cookbooks](http://community.opscode.com/cookbooks)

# Lab Exercise

Getting Started

* Install Ruby and Chef
* Get familiar with the tools that come with Chef
* Set up connectivity to a Chef Server
* Clone a Chef Repository with Git
