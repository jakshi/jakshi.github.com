---
layout: post
title: "Manage AWS Elastic IPs with fog"
date: 2013-06-03 19:25
comments: true
categories: 
- technical
- aws
- amazon
- fog
- ruby
---

# Connect to AWS API with fog.

    require 'fog'

    c = Fog::Compute.new(
                     :provider => 'AWS',
                     :aws_access_key_id => 'Your AWS access key',
                     :aws_secret_access_key => 'Your AWS secret key',
                     :region => 'us-east-1' )

<!-- more -->

# Associate Elastic IP

    eip = c.addresses.get('Your elastic IP address')
    c.associate_address("Your instance id",nil,nil,eip.allocation_id)

# Disassociate Elastic IP

    eip = c.describe_addresses('public-ip' => ['Your elastic IP address'])
    disassociate_address(nil,eip[:body]["addressesSet"][0]["associationId"])

# Association Elastic IP script example.

**assign_eip.rb**:

    #!/usr/bin/ruby
    # This is a script that assign Elastic IP to current instance.
    # Usage:
    # You need to fill out config_eip.json config file:
    # {
    # "aws_access_key": "aws_access_key",
    # "aws_secret_key": "aws_secret_key",
    # "eip": "x.x.x.x",
    # }
    #
    # Config file should be in the same dir as a script
    # then, start script:
    # assign_eip.rb
    # This will assign x.x.x.x Elastic IP to current instance.
    
    require "rubygems"
    require "json"
    require "net/http"
    require "fog"
    
    # Read config
    CONFIG = JSON.parse(IO.read(File.join(File.dirname(__FILE__), 'assign_eip.json')))
    
    # Here we need to get server.id
    INSTANCE_HOST = '169.254.169.254'
    INSTANCE_ID_URL = '/latest/meta-data/instance-id'
    INSTANCE_REGION_URL = '/latest/meta-data/placement/availability-zone'
    
    httpcall = Net::HTTP.new(INSTANCE_HOST)
    resp, instance_id = httpcall.get2(INSTANCE_ID_URL)
    resp, region = httpcall.get2(INSTANCE_REGION_URL)
    
    # Cut out availability zone marker.
    # For example if region == "us-east-1c" after cutting out it will be
    # "us-east-1"
    
    region = region[0..-2]
    
    # First we get a connection object from amazon, region is
    # required if your instances are in other zone than the
    # gem's default one (us-east-1).
    
    c = Fog::Compute.new(
                     :provider => 'AWS',
                     :aws_access_key_id => CONFIG['aws_access_key'],
                     :aws_secret_access_key => CONFIG['aws_secret_key'],
                     :region => region )
    
    # Then we get Fog::Compute::AWS::Address to get allocation_id of Elastic IP.
    # For some reason I failed to make it work with IP address directly.
    # if I use Elastic IP instead of allocation id it always returns 400
    # Bad Request.
    
    eip = c.addresses.get(CONFIG['eip'])
    
    # Then we accociate Elastic IP with current node.
    
    c.associate_address(instance_id,nil,nil,eip.allocation_id)

**assign_eip.json**:

    {
    "aws_access_key": "AWS_access_key",
    "aws_secret_key": "Your_AWS_secret_key",
    "eip": "Elastic IP to assign"
    }

# Further reading

  * [RubyDoc: fog](http://rubydoc.info/gems/fog/frames)
  * [RubyDoc: Fog::Compute::AWS](http://rubydoc.info/github/fog/fog/Fog/Compute/AWS)
  * [RubyDoc: Fog::AWS::EC2.associate_address](http://rubydoc.info/github/stesla/fog/Fog/AWS/EC2#associate_address-instance_method)
  * [RubyDoc: Fog::Compute::AWS::Addresses](http://rubydoc.info/github/fog/fog/Fog/Compute/AWS/Addresses)
  * [RubyDoc: Fog Alphabetic Index](http://rubydoc.info/github/fog/fog/index)
  * [GitHub: fog - associate_address.rb](https://github.com/fog/fog/blob/master/lib/fog/aws/requests/compute/associate_address.rb)
  * [Using fog for attaching AWS elastic ip addresses to servers](http://opsrobot.com/post/9631423001/using-fog-for-attaching-aws-elastic-ip-addresses-to)
  * [Fog home site](http://fog.io/)
  * [AWS Documentation: Instance Metadata and User Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AESDG-chapter-instancedata.html)
  * [AWS Elastic IP remapper](https://github.com/madebymade/aws-remap-elastic-ip)