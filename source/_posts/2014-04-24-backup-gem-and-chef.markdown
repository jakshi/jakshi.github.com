---
layout: post
title: "Backup gem and Chef"
date: 2014-04-24 12:48
comments: true
categories: 
  - technical
  - aws
  - amazon
  - chef
  - devops
  - ruby
  - backup
---

# Intro

* There's a nice tool for doing fast and easy backup to AWS S3: [backup gem](http://meskyanichi.github.io/backup/)
* There should be done a lot of steps for setup a backup though.
* So it could be automated with chef.
* In this article I'll write log of creation backup gem's cookbook.
* Essentially this cookbook will install backup gem, that will backup /var/www folder to AWS S3 every day in 01:00. It will store last 14 backups.

<!-- more -->

# Chef application cookbook for backup gem
## Create a cookbook

   mkdir example-backup
   cd example-backup

## Create metadata

    emacs metadata.rb

```
name             'example-backup'
maintainer       'John Smith'
maintainer_email 'john@example.com'
license          'Apache 2.0'
description      'Installs/Configures example-backup'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '0.0.1'

%w{ubuntu}.each do |os|
  supports os
end

depends 'backup', '~> 0.4.0'
depends 'build-essential'
depends 'cron', '= 1.3.8'
```

* We will use 'backup' gem for setupping backup.
* build-essential is necessary for installing gem's native extensions
* cron is restricted to 1.3.8 version, because last version of cron (1.3.10) was broken on the moment of writing this article.
* This example is for ubuntu 12.04 LTS only

## Attributes

    emacs attributes/default.rb

```
default['example']['backup']['s3-bucket'] = 'example.com.backup'
override['build-essential']['compile_time'] = true
override['backup']['dependencies'] = [['fog'],['aws-s3']]
override['backup']['version'] = '4.0.1'
override['backup']['path'] = '/opt/backup'
```

## Templates

    emacs templates/default/config.rb.erb

```
# encoding: utf-8

##
# Backup v4.x Configuration
#
# Documentation: http://meskyanichi.github.io/backup
# Issue Tracker: https://github.com/meskyanichi/backup/issues
root_path '<%= @root_path %>'
tmp_path '<%= @tmp_path %>'
data_path '<%= @data_path %>'
```

## Recipes
### default

    emacs recipes/default.rb

```
include_recipe 'example-backup::dependencies'
include_recipe 'example-backup::backup'
```

### dependencies
This recipe is to install dependenices that neccessary for backup gem to functioning.

    emacs recipes/dependencies.rb

```
include_recipe 'build-essential'

%w{ruby1.9.1-full libopenssl-ruby1.9.1 libssl-dev zlib1g-dev}.each do |pkg|
  package(pkg).run_action(:install)
end

# Make ruby19 default ruby interpreter
# Ruby 1.8.x on Ubuntu 12.04 LTS have priority 50
# Let's setup priority 400 for ruby 1.9.x
bash 'ruby19_by_default' do
  cwd '/tmp'
  code <<-EOH
  update-alternatives --install /usr/bin/ruby ruby /usr/bin/ruby1.9.1 400 --slave /usr/share/man/man1/ruby.1.gz ruby.1.gz /usr/share/man/man1/ruby1.9.1.1.gz --slave /usr/bin/erb erb /usr/bin/erb1.9.1 --slave /usr/share/man/man1/erb.1.gz erb.1.gz /usr/share/man/man1/erb1.9.1.1.gz --slave /usr/bin/irb irb /usr/bin/irb1.9.1 --slave /usr/share/man/man1/irb.1.gz irb.1.gz /usr/share/man/man1/irb1.9.1.1.gz --slave /usr/bin/rdoc rdoc /usr/bin/rdoc1.9.1 --slave /usr/share/man/man1/rdoc.1.gz rdoc.1.gz /usr/share/man/man1/rdoc1.9.1.1.gz --slave /usr/bin/ri ri /usr/bin/ri1.9.1 --slave /usr/share/man/man1/ri.1.gz ri.1.gz /usr/share/man/man1/ri1.9.1.1.gz --slave /usr/bin/testrb testrb /usr/bin/testrb1.9.1 --slave /usr/share/man/man1/testrb.1.gz testrb.1.gz /usr/share/man/man1/testrb1.9.1.1.gz
EOH
end

chef_gem "chef-rewind"
require 'chef/rewind'

include_recipe 'backup'

%w{keys .data logs}.each do |dir|
  directory "#{node['backup']['path']}/#{dir}" do
    action :create
    recursive true
  end
end

rewind :template => "Backup config file" do
  source 'config.rb.erb'
  cookbook_name 'example-backup'
  variables({
    :root_path => node['backup']['path'],
    :tmp_path => '/tmp',
    :data_path => "#{node['backup']['path']}/.data"
  })
end
```

* Recipe installs ruby 1.9 and make it default.
* I include backup::default recipe, that will install backup, fog and aws-s3 gems, create backup config and some necessary dirs.
* Little trick. Last version of backup recipe doesn't support backup gem 4.x version. 
* Difference is in the config file template. 
* So I substitute backup's cookbook config file template (for 3.x version of backup gem) with my own config template for 4.x version of backup gem.
* I use chef-rewind gem for that

### backup
Action part of recipe. It will configure backup.

    emacs recipes/backup.rb

```
aws = data_bag_item('aws', 'backup')

backup_model :www do
  description "backup of /var/www folder"
  definition <<-EOS
before do
  require 'aws/s3'
  AWS::S3::Base.establish_connection!(:access_key_id => '#{aws['aws_access_key_id']}', :secret_access_key => '#{aws['aws_secret_access_key']}')
  AWS::S3::Bucket.create('#{node['example']['backup']['s3-bucket']}')
  AWS::S3::Base.disconnect!
end

split_into_chunks_of 50
  
store_with S3 do |s3|
  s3.access_key_id = '#{aws['aws_access_key_id']}'
  s3.secret_access_key = '#{aws['aws_secret_access_key']}'
  s3.region = 'us-east-1'
  s3.bucket = '#{node['example']['backup']['s3-bucket']}'
  s3.path = '/'
  s3.keep = 14
  s3.fog_options = {
    :path_style => true
  }
end

##
# Gzip [Compressor]
#
compress_with Gzip

archive :www do |archive|
  archive.add '/var/www/'
  archive.tar_options '-p'
end

EOS
  schedule({
    :minute => '0',
    :hour   => '1',
  })
  cron_options({
    :path => '/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin'
  })
end
```

* I get aws credentials from aws data bag.
* They could be loaded from other source, like attributes, anything you like.
* In backup model I create a before hook that will create amazon bucket for backups in case it doesn't exist.
* Then backup resource just describe backupping /var/www to Amazon S3 bucket, keep 14 backup, start every day on 01:00 AM.

## Usage

* I presume that you use chef-server and you already have some chef repository, like `example-chef-repo`.
* Fill aws data bag with credentials

```
mkdir -p example-chef-repo/data_bags/aws
emacs example-chef-repo/data_bags/aws/backup.json

{
  "id": "backup",
  "aws_access_key_id": "AWS_ACCESS_KEY",
  "aws_secret_access_key": "AWS_SECRET_KEY"
}
```

* It's better to use AWS_ACCESS_KEY and AWS_SECRET_KEY credentials of IAM user that has access to S3 only.
* Upload that data bag to your chef-server

```
knife data bag create aws
knife data bag from file aws backup.json
```

* If you don't have chef-server, rewrite recipe to use attributes, and set AWS credentials through attributes.
* And That's it - just add example-backup to your node run_list.

```
knife node run_list add node_name 'recipe[example-backup]'
```

* And run `chef-client` on the node.

# References
## cookbooks

* [backup cookbook on github](https://github.com/gofullstack/backup-cookbook)
* [backup lwrp cookbook on github](https://github.com/damm/backup)
* [cron cookbook on github](https://github.com/opscode-cookbooks/cron)

## gems
### chef-rewind gem

* [chef-rewind gem on github](https://github.com/bryanwb/chef-rewind)

### backup gem

* [backup gem v4.x documentation](http://meskyanichi.github.io/backup/v4/)

### aws-s3 gem

* [AWS::S3 module documentation](http://rubydoc.info/gems/aws-s3/0.6.3/frames)
* [AWS::S3::Service class documentation](http://rubydoc.info/gems/aws-s3/0.6.3/AWS/S3/Service)
* [AWS::S3::Bucket class documentation](http://rubydoc.info/gems/aws-s3/0.6.3/AWS/S3/Bucket)

### test-kitchen

* [test-kitchen gem on github](https://github.com/test-kitchen/test-kitchen)
* [kitchen-digital ocean driver on github](https://github.com/test-kitchen/kitchen-digitalocean)

### berkshelf

* [berkshelf gem documentation](http://berkshelf.com/)