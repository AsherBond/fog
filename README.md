![fog](http://geemus.s3.amazonaws.com/fog.png)

fog is the Ruby cloud services library, top to bottom:

* Collections provide a simplified interface, making clouds easier to work with and switch between.
* Requests allow power users to get the most out of the features of each individual cloud.
* Mocks make testing and integrating a breeze.

[![Build Status](https://github.com/fog/fog/actions/workflows/ruby.yml/badge.svg)](https://github.com/fog/fog/actions/workflows/ruby.yml)
[![Code Climate](https://codeclimate.com/github/fog/fog/badges/gpa.svg)](https://codeclimate.com/github/fog/fog)
[![Gem Version](https://badge.fury.io/rb/fog.svg)](http://badge.fury.io/rb/fog)
[![SemVer](https://api.dependabot.com/badges/compatibility_score?dependency-name=fog&package-manager=bundler&version-scheme=semver)](https://dependabot.com/compatibility-score.html?dependency-name=fog&package-manager=bundler&version-scheme=semver)

## Dependency Notice

Currently all fog providers are getting separated into metagems to lower the
load time and dependency count.

If there's a metagem available for your cloud provider, e.g. `fog-aws`,
you should be using it instead of requiring the full fog collection to avoid
unnecessary dependencies.

'fog' should be required explicitly only if the provider you use doesn't yet
have a metagem available.

## Getting Started

The easiest way to learn fog is to install the gem and use the interactive console.
Here is an example of wading through server creation for Amazon Elastic Compute Cloud:

```
$ sudo gem install fog
[...]

$ fog

  Welcome to fog interactive!
  :default provides [...]

>> server = Compute[:aws].servers.create
ArgumentError: image_id is required for this operation

>> server = Compute[:aws].servers.create(:image_id => 'ami-5ee70037')
<Fog::AWS::EC2::Server [...]>

>> server.destroy # cleanup after yourself or regret it, trust me
true
```

## Ruby version

Fog requires Ruby `2.0.0` or later.

Ruby `1.8` and `1.9` support was dropped in `fog-v2.0.0` as a backwards incompatible
change. Please use the later fog `1.x` versions if you require `1.8.7` or `1.9.x` support.

## Collections

A high level interface to each cloud is provided through collections, such as `images` and `servers`.
You can see a list of available collections by calling `collections` on the connection object.
You can try it out using the `fog` command:

    >> Compute[:aws].collections
    [:addresses, :directories, ..., :volumes, :zones]

Some collections are available across multiple providers:

* compute providers have `flavors`, `images` and `servers`
* dns providers have `zones` and `records`
* storage providers have `directories` and `files`

Collections share basic CRUD type operations, such as:

* `all` - fetch every object of that type from the provider.
* `create` - initialize a new record locally and a remote resource with the provider.
* `get` - fetch a single object by its identity from the provider.
* `new` - initialize a new record locally, but do not create a remote resource with the provider.

As an example, we'll try initializing and persisting a Rackspace Cloud server:

```ruby
require 'fog'

compute = Fog::Compute.new(
  :provider           => 'Rackspace',
  :rackspace_api_key  => key,
  :rackspace_username => username
)

# boot a gentoo server (flavor 1 = 256, image 3 = gentoo 2008.0)
server = compute.servers.create(:flavor_id => 1, :image_id => 3, :name => 'my_server')
server.wait_for { ready? } # give server time to boot

# DO STUFF

server.destroy # cleanup after yourself or regret it, trust me
```

## Models

Many of the collection methods return individual objects, which also provide common methods:

* `destroy` - will destroy the persisted object from the provider
* `save` - persist the object to the provider
* `wait_for` - takes a block and waits for either the block to return true for the object or for a timeout (defaults to 10 minutes)

## Mocks

As you might imagine, testing code using Fog can be slow and expensive, constantly turning on and shutting down instances.
Mocking allows skipping this overhead by providing an in memory representation of resources as you make requests.
Enabling mocking is easy to use: before you run other commands, simply run:

```ruby
Fog.mock!
```

Then proceed as usual, if you run into unimplemented mocks, fog will raise an error and as always contributions are welcome!

## Requests

Requests allow you to dive deeper when the models just can't cut it.
You can see a list of available requests by calling `#requests` on the connection object.

For instance, ec2 provides methods related to reserved instances that don't have any models (yet). Here is how you can lookup your reserved instances:

    $ fog
    >> Compute[:aws].describe_reserved_instances
    #<Excon::Response [...]>

It will return an [excon](http://github.com/geemus/excon) response, which has `body`, `headers` and `status`. Both return nice hashes.

## Go forth and conquer

Play around and use the console to explore or check out [fog.github.io](http://fog.github.io) and the [provider documentation](http://fog.github.io/about/provider_documentation.html)
for more details and examples. Once you are ready to start scripting fog, here is a quick hint on how to make connections without the command line thing to help you.

```ruby
# create a compute connection
compute = Fog::Compute.new(:provider => 'AWS', :aws_access_key_id => ACCESS_KEY_ID, :aws_secret_access_key => SECRET_ACCESS_KEY)
# compute operations go here

# create a storage connection
storage = Fog::Storage.new(:provider => 'AWS', :aws_access_key_id => ACCESS_KEY_ID, :aws_secret_access_key => SECRET_ACCESS_KEY)
# storage operations go here
```

geemus says: "That should give you everything you need to get started, but let me know if there is anything I can do to help!"

## Versioning

Fog library aims to adhere to [Semantic Versioning 2.0.0][semver], although it does not
address challenges of multi-provider libraries. Semantic versioning is only guaranteed for
the common API, not any provider-specific extensions.  You may also need to update your
configuration from time to time (even between Fog releases) as providers update or deprecate
services.

However, we still aim for forwards compatibility within Fog major versions.  As a result of this policy, you can (and
should) specify a dependency on this gem using the [Pessimistic Version
Constraint][pvc] with two digits of precision. For example:

```ruby
spec.add_dependency 'fog', '~> 1.0'
```

This means your project is compatible with Fog 1.0 up until 2.0.  You can also set a higher minimum version:

```ruby
spec.add_dependency 'fog', '~> 1.16'
```

[semver]: http://semver.org/
[pvc]: http://guides.rubygems.org/patterns/

## Getting Help

* [General Documentation](http://fog.github.io).
* [Provider Specific Documentation](http://fog.github.io/about/provider_documentation.html).
* Ask specific questions on [Stack Overflow](http://stackoverflow.com/questions/tagged/fog)
* Report bugs and discuss potential features in [Github issues](https://github.com/fog/fog/issues).

## Contributing

Please refer to [CONTRIBUTING.md](https://github.com/fog/fog/blob/master/CONTRIBUTING.md).

## License

Please refer to [LICENSE.md](https://github.com/fog/fog/blob/master/LICENSE.md).
