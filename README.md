Digital Ocean Vagrant Provider
==============================
`vagrant-digitalocean` is a provider plugin for Vagrant that supports the
management of [Digital Ocean](https://www.digitalocean.com/) droplets
(instances).

**NOTE:** The Chef provisioner is no longer supported by default (as of 0.2.0).
Please use the `vagrant-omnibus` plugin to install Chef on Vagrant-managed
machines. This plugin provides control over the specific version of Chef
to install.

Current features include:
- create and destroy droplets
- power on and off droplets
- rebuild a droplet
- provision a droplet with the shell or Chef provisioners
- setup a SSH public key for authentication
- create a new user account during droplet creation

The provider has been tested with Vagrant 1.1.5+ using Ubuntu 12.04 and
CentOS 6.3 guest operating systems.

Install
-------
Installation of the provider requires two steps:

1. Install the provider plugin using the Vagrant command-line interface:

        $ vagrant plugin install vagrant-digitalocean


**NOTE:** If you are using a Mac, and this plugin would not work caused by SSL certificate problem,
You may need to specify certificate path explicitly.  
You can verify actual certificate path by running:

```bash
ruby -ropenssl -e "p OpenSSL::X509::DEFAULT_CERT_FILE"
```

Then, add the following environment variable to your
`.bash_profile` script and `source` it:

```bash
export SSL_CERT_FILE=/usr/local/etc/openssl/cert.pem
```

Configure
---------
Once the provider has been installed, you will need to configure your project
to use it. The most basic `Vagrantfile` to create a droplet on Digital Ocean
is shown below:

```ruby
Vagrant.configure('2') do |config|

  config.vm.provider :digital_ocean do |provider, override|
    override.ssh.private_key_path = '~/.ssh/id_rsa'
    override.vm.box = 'digital_ocean'
    override.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"

    provider.token = 'YOUR TOKEN'
    provider.image = 'ubuntu-14-04-x64'
    provider.region = 'nyc2'
    provider.size = '512mb'
  end
end
```

Please note the following:
- You *must* specify the `override.ssh.private_key_path` to enable authentication
  with the droplet. The provider will create a new Digital Ocean SSH key using
  your public key which is assumed to be the `private_key_path` with a *.pub*
  extension.
- You *must* specify your Digital Ocean Personal Access Token. This may be
  found on the control panel within the *Apps &amp; API* section.

**Supported Configuration Attributes**

The following attributes are available to further configure the provider:
- `provider.image` - A string representing the image to use when creating a
   new droplet. It defaults to `ubuntu-14-04-x64`.
- `provider.ipv6` - A boolean flag indicating whether to enable IPv6
- `provider.region` - A string representing the region to create the new
   droplet in. It defaults to `nyc2`.
- `provider.size` - A string representing the size to use when creating a
  new droplet (e.g. `1gb`). It defaults to `512mb`.
- `provider.private_networking` - A boolean flag indicating whether to enable
  a private network interface (if the region supports private networking). It
  defaults to `false`.
- `provider.backups_enabled` - A boolean flag indicating whether to enable backups for
   the droplet. It defaults to `false`.
- `provider.ssh_key_name` - A string representing the name to use when creating
  a Digital Ocean SSH key for droplet authentication. It defaults to `Vagrant`.
- `provider.setup` - A boolean flag indicating whether to setup a new user
  account and modify sudo to disable tty requirement. It defaults to `true`.
  If you are using a tool like [packer](https://packer.io) to create
  reusable snapshots with user accounts already provisioned, set to `false`.

The provider will create a new user account with the specified SSH key for
authorization if `config.ssh.username` is set and the `provider.setup`
attribute is `true`.

### provider.image slug

Each image has to be specified with the slug name.

| slug                | image name                |
|:--------------------|:--------------------------|
| ubuntu-14-10-x32    | Ubuntu 14.10 x32          |
| coreos-beta         | CoreOS 444.5.0 (beta)     |
| coreos-stable       | CoreOS 444.5.0 (stable)   |
| coreos-alpha        | CoreOS 472.0.0 (alpha)    |
| fedora-20-x64       | Fedora 0 x64              |
| fedora-20-x32       | Fedora 20 x32             |
| fedora-19-x64       | Fedora 19 x64             |
| fedora-19-x32       | Fedora 19 x32             |
| centos-6-5-x32      | CentOS 6.5 x32            |
| centos-6-5-x64      | CentOS 6.5 x64            |
| centos-5-8-x64      | CentOS 5.10 x64           |
| centos-5-8-x32      | CentOS 5.10 x32           |
| debian-7-0-x64      | Debian 7.0 x64            |
| debian-7-0-x32      | Debian 7.0 x32            |
| debian-6-0-x64      | Debian 6.0 x64            |
| debian-6-0-x32      | Debian 6.0 x32            |
| ubuntu-10-04-x64    | Ubuntu 10.04 x64          |
| ubuntu-10-04-x32    | Ubuntu 10.04 x32          |
| ubuntu-12-04-x64    | Ubuntu 12.04.5 x64        |
| ubuntu-12-04-x32    | Ubuntu 12.04.5 x32        |

You can find latest image slug name using DigitalOcean API V2 call. Example call:

```
curl -s https://api.digitalocean.com/v2/images \
      -H 'Authorization:Bearer $DIGITAL_OCEAN_TOKEN' \
      | jq '.images[].slug'
```

More details: [DigitalOcean API - Images](https://developers.digitalocean.com/#images)

### provider.region slug

Each region has to be specified with the slug name.
Current Region-slug table is:

| slug | Region Name     |
|:---- |:----------------|
| nyc1 | New York 1      |
| ams1 | Amsterdam 1     |
| sfo1 | San Francisco 1 |
| nyc2 | New York 2      |
| ams2 | Amsterdam 2     |
| sgp1 | Singapore 1     |
| lon1 | London 1        |
| nyc3 | New York 3      |

You can find latest region slug name using DigitalOcean API V2 call. Example call:

```
curl -X GET "https://api.digitalocean.com/v2/regions" \
     -H "Authorization: Bearer $DIGITAL_OCEAN_TOKEN" \
     2>/dev/null | jq '.regions [] | .slug,.name'
```

More details: [DigitalOcean API - Regions](https://developers.digitalocean.com/#regions)

Run
---
After creating your project's `Vagrantfile` with the required configuration
attributes described above, you may create a new droplet with the following
command:

    $ vagrant up --provider=digital_ocean

This command will create a new droplet, setup your SSH key for authentication,
create a new user account, and run the provisioners you have configured.

**Supported Commands**

The provider supports the following Vagrant sub-commands:
- `vagrant destroy` - Destroys the droplet instance.
- `vagrant ssh` - Logs into the droplet instance using the configured user
  account.
- `vagrant halt` - Powers off the droplet instance.
- `vagrant provision` - Runs the configured provisioners and rsyncs any
  specified `config.vm.synced_folder`.
- `vagrant reload` - Reboots the droplet instance.
- `vagrant rebuild` - Destroys the droplet instance and recreates it with the
  same IP address which was previously assigned.
- `vagrant status` - Outputs the status (active, off, not created) for the
  droplet instance.

Contribute
----------
To contribute, clone the repository, and use [Bundler](http://gembundler.com)
to install dependencies:

    $ bundle

To run the provider's tests:

    $ bundle exec rake test

You can now make modifications. Running `vagrant` within the Bundler
environment will ensure that plugins installed in your Vagrant
environment are not loaded.
