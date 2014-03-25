# vagrant-lxc

[![Build Status](https://travis-ci.org/fgrehm/vagrant-lxc.png?branch=master)](https://travis-ci.org/fgrehm/vagrant-lxc) [![Gem Version](https://badge.fury.io/rb/vagrant-lxc.png)](http://badge.fury.io/rb/vagrant-lxc) [![Code Climate](https://codeclimate.com/github/fgrehm/vagrant-lxc.png)](https://codeclimate.com/github/fgrehm/vagrant-lxc) [![Coverage Status](https://coveralls.io/repos/fgrehm/vagrant-lxc/badge.png?branch=master)](https://coveralls.io/r/fgrehm/vagrant-lxc) [![Gittip](http://img.shields.io/gittip/fgrehm.svg)](https://www.gittip.com/fgrehm/)

[LXC](http://lxc.sourceforge.net/) provider for [Vagrant](http://www.vagrantup.com/) 1.1+

This is a Vagrant plugin that allows it to control and provision Linux Containers
as an alternative to the built in VirtualBox provider for Linux hosts. Check out
[this blog post](http://fabiorehm.com/blog/2013/04/28/lxc-provider-for-vagrant/)
to see it in action.

**NOTICE:** The master branch is targetting an initial beta for 1.0.0, for the
latest stable version of the plugin, please check the [0.8-stable](https://github.com/fgrehm/vagrant-lxc/tree/0.8-stable)
branch.

## Usage with Vagrant 1.5

Usage with the recently released Vagrant 1.5 is only possible by [building the
plugin from sources](https://github.com/fgrehm/vagrant-lxc/wiki/Development#wiki-installing-the-plugin-from-source).
The 1.0.0.beta1 version of the plugin that will ship with the changes required
is expected to be released by April, 2014.


## Features

* Provides the same workflow as the Vagrant VirtualBox provider
* Port forwarding via [`redir`](http://linux.die.net/man/1/redir)

As of now, it does not support public / private networks, but [private networks](https://github.com/fgrehm/vagrant-lxc/issues/120)
will be coming along _soon_.

## Requirements

* [Vagrant 1.1+](http://www.vagrantup.com/downloads.html)
* lxc 0.7.5+
* `redir` (if you are planning to use port forwarding)
* A [kernel != 3.5.0-17.28](https://github.com/fgrehm/vagrant-lxc/wiki/Troubleshooting#wiki-im-unable-to-restart-containers)

The plugin is known to work better and pretty much out of the box on Ubuntu 12.04+
hosts and installing the dependencies on it basically means a `apt-get install lxc lxc-templates cgroup-lite redir`
and a `apt-get update && apt-get dist-upgrade` to upgrade the kernel. For Debian
hosts you'll need to follow the instructions described on the [Wiki](https://github.com/fgrehm/vagrant-lxc/wiki/Usage-on-debian-hosts).

Some manual steps are required to set up a Linode machine prior to using this
plugin, please check [the wiki](https://github.com/fgrehm/vagrant-lxc/wiki/Usage-on-Linode)
for more information. Documentation on how to set things up for other distros
[are welcome](https://github.com/fgrehm/vagrant-lxc/wiki) :)

If you are on a Mac or Windows machine, you might want to have a look at [this](http://the.taoofmac.com/space/HOWTO/Vagrant)
blog post for some ideas on how to set things up or check out [this other repo](https://github.com/fgrehm/vagrant-lxc-vbox-hosts)
for a set of Vagrant VirtualBox machines ready for vagrant-lxc usage.

**NOTE: Some users have been experiencing networking issues and right now you might need to
disable checksum offloading as described on [this comment](https://github.com/fgrehm/vagrant-lxc/issues/153#issuecomment-26441273)**


## Installation

```
vagrant plugin install vagrant-lxc
```


## Usage

After installing, add a [base box](#base-boxes) using any name you want, for example:

```
vagrant box add quantal64 http://bit.ly/vagrant-lxc-quantal64-2013-10-23
```

Then create a Vagrantfile that looks like the following, changing the box name
to the one you've just added:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "quantal64"
end
```

And finally run `vagrant up --provider=lxc`.

If you are using Vagrant 1.2+ you can also set `VAGRANT_DEFAULT_PROVIDER`
environmental variable to `lxc` in order to avoid typing `--provider=lxc` all
the time.


### Advanced configuration

If you want, you can modify container configurations from within your Vagrantfile
using the [provider block](http://docs.vagrantup.com/v2/providers/configuration.html):

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "quantal64"
  config.vm.provider :lxc do |lxc|
    # Same effect as 'customize ["modifyvm", :id, "--memory", "1024"]' for VirtualBox
    lxc.customize 'cgroup.memory.limit_in_bytes', '1024M'
  end
end
```

vagrant-lxc will then write out `lxc.cgroup.memory.limit_in_bytes='1024M'` to the
container config file (usually kept under `/var/lib/lxc/<container>/config`)
prior to starting it.

For other configuration options, please check the [lxc.conf manpages](http://manpages.ubuntu.com/manpages/precise/man5/lxc.conf.5.html).

### Container naming

By default vagrant-lxc will attempt to generate a unique container name
for you. However, if the container name is important to you, you may use the
`container_name` attribute to set it explicitly from the `provider` block:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "quantal64"

  config.vm.define "db" do |node|
    node.vm.provider :lxc do |lxc|
      lxc.container_name = :machine # Sets the container name to 'db'
      lxc.container_name = 'mysql'  # Sets the container name to 'mysql'
    end
  end
end
```

### Avoiding `sudo` passwords

This plugin requires **a lot** of `sudo`ing since [user namespaces](https://wiki.ubuntu.com/UserNamespace)
are not supported on mainstream kernels. To work around that, you can use the
`vagrant lxc sudoers` command which will create a file under `/etc/sudoers.d/vagrant-lxc-<USER>`
whitelisting all commands required by `vagrant-lxc` to run.

If you are interested on what will be generated by that command, please check
[this code](lib/vagrant-lxc/commands/sudoers.rb#L58-L81).

_vagrant-lxc < 1.0.0 users, please check this [Wiki page](https://github.com/fgrehm/vagrant-lxc/wiki/Avoiding-%27sudo%27-passwords)_

### Base boxes

Scripts for building Ubuntu and Debian base boxes are available at
[fgrehm/vagrant-lxc-base-boxes](https://github.com/fgrehm/vagrant-lxc-base-boxes)
and the link to pre built Ubuntu boxes can be found at [this section](https://github.com/fgrehm/vagrant-lxc-base-boxes#pre-built-base-boxes).

If you want to build your own boxes, please have a look at [`BOXES.md`](https://github.com/fgrehm/vagrant-lxc/tree/master/BOXES.md)
for more information.

_vagrant-lxc < 1.0.0 users, please check this [Wiki page](https://github.com/fgrehm/vagrant-lxc/wiki/Base-boxes)_


## More information

Please refer the [wiki](https://github.com/fgrehm/vagrant-lxc/wiki).


## Problems / ideas?

Please review the [Troubleshooting](https://github.com/fgrehm/vagrant-lxc/wiki/Troubleshooting)
wiki page + [known bugs](https://github.com/fgrehm/vagrant-lxc/issues?labels=bug&page=1&state=open)
list if you have a problem and feel free to use the [issue tracker](https://github.com/fgrehm/vagrant-lxc/issues)
propose new functionality and / or report bugs.


## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
