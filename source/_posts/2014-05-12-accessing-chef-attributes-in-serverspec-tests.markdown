---
layout: post
title: "Accessing chef attributes in serverspec tests"
date: 2014-05-12 01:45
comments: true
categories:
  - chef
  - technical
  - serverspec
  - devops
---

I presume that you are familiar with:

* Chef
* test-kitchen
* serverspec

When you write serverspec integration tests, it would be great to have access to chef attributes of cookbook that you're testing.
There's a fast and simple way to do this.

<!-- more -->

# Introduction

```
cat attributes/default.rb
```

```
override['backup']['dependencies'] = [['fog'],['aws-s3']]
```

Let's say we want this attribute in your serverspec tests.

How can we do that?

* Dump chef attributes to JSON file with helper cookbook
* Load this file from serverspec tests

# Dump chef attributes
## Create fixture cookbook

```
emacs test/fixtures/cookbooks/test-helper/metadata.rb
```

```
name             'test-helper'
maintainer       'John Smith'
maintainer_email 'john@example.com'
license          'Apache 2.0'
description      'Dumps chef node data to json file'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '0.0.1'

recipe 'default', 'Dumps chef node data to json file'

%w{ ubuntu debian }.each do |os|
  supports os
end
```

```
emacs test/fixtures/cookbooks/test-helper/recipes/default.rb
```

```
chef_gem 'activesupport'

require 'pathname'
require 'active_support/core_ext/hash/deep_merge'

directory '/tmp/serverspec' do
  recursive true
end

file '/tmp/serverspec/node.json' do
  owner "root"
  mode "0400"
end

log "Dumping attributes to '/tmp/serverspec/node.json."

ruby_block "dump_node_attributes" do
  block do
    require 'json'

    attrs = {}

    attrs = attrs.deep_merge(node.default_attrs) unless node.default_attrs.empty?
    attrs = attrs.deep_merge(node.normal_attrs) unless node.normal_attrs.empty?
    attrs = attrs.deep_merge(node.override_attrs) unless node.override_attrs.empty?

    recipe_json = "{ \"run_list\": \[ "
    recipe_json << node.run_list.expand(node.chef_environment).recipes.map! { |k| "\"#{k}\"" }.join(",")
    recipe_json << " \] }"
    attrs = attrs.deep_merge(JSON.parse(recipe_json))

    File.open('/tmp/serverspec/node.json', 'w') { |file| file.write(JSON.pretty_generate(attrs)) }
  end
end
```

```
echo "This a cookbook for dumping chef node attributes to specific location to json formated file." > test/fixtures/cookbooks/test-helper/README.md
```

## Add it to .kitchen.yml

```
emacs .kitchen.yml
```

```
---
driver:
  name: digitalocean
  region: amsterdam 2
  flavor: 512MB

provisioner:
  name: chef_solo

platforms:
  - name: ubuntu-12.04

suites:
  - name: default
    run_list:
      - recipe[my_cookbook::default]
      - recipe[test-helper::default]
    attributes:

```

## Add cookbook helper to Berksfile

```
emacs Berksfile
```

```
source "http://api.berkshelf.com"

metadata

group :integration do
  cookbook 'test-helper', path: 'test/fixtures/cookbooks/test-helper'
end
```

```
berks update
```

# Load chef attributes

```
emacs test/integration/default/serverspec/spec_helper.rb
```

```
require 'serverspec'
require 'pathname'
require 'net/http'
require 'net/smtp'
require 'json'

set :backend, :exec

$node = ::JSON.parse(File.read('/tmp/serverspec/node.json'))
```

# Use chef attributes in tests

```
require 'spec_helper'

describe 'my_cookbook' do

  context 'dependencies recipe. It' do
    $node['backup']['dependencies'].each do |bgem|
      it "installs backup gem dependency: #{bgem[0]}" do
        expect(package bgem[0]).to be_installed.by('gem')
      end
    end
  end

end
```

# Conclusion

if you execute:

```
kitchen verify
```

you should see:
```
my_cookbook
  dependencies recipe. It       
    installs backup gem dependency: fog       
    installs backup gem dependency: aws-s3       
```