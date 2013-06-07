---
layout: post
title: "Manage AWS Elastic IPs with AWS Ruby SDK"
date: 2013-06-07 13:48
comments: true
categories: 
- technical
- aws
- amazon
- AWS SDK
- ruby
---
I tried three ruby libs for managing Elastic IPs for AWS EC2.

* fog
* right_aws
* Official Ruby AWS SDK.

I wholeheartedly recommend Official AWS Ruby SDK, as most simple and easy to use.
I also insert several examples of assigning Elastic IPs with AWS Ruby SDK:

<!-- more -->

# Connect to AWS API with AWS Ruby SDK.

    c = AWS::EC2.new(:access_key_id => "AWS_access_key", :secret_access_key => "AWS_secret_key", :region => "us-east-1")

# Associate Elastic IP
**x.x.x.x** in this example is your Elastic IP address

    eip = c.elastic_ips["x.x.x.x"]
    eip.associate :instance => "Your instance id"

# Disassociate Elastic IP
**x.x.x.x** in this example is your Elastic IP address

    eip = c.elastic_ips["x.x.x.x"]
    eip.disassociate

# Further reading

  * [AWS Documentation: Instance Metadata and User Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AESDG-chapter-instancedata.html)
  * [AWS Elastic IP remapper](https://github.com/madebymade/aws-remap-elastic-ip)

## AWS Ruby SDK

  * [GitHub: AWS Ruby SDK Examples](https://github.com/aws/aws-sdk-ruby/tree/master/samples)
  * [AWS Docs: AWS Ruby SDK Getting Started](http://docs.aws.amazon.com/AWSSdkDocsRuby/latest
/DeveloperGuide/ruby-dg-setup.html)
  * [AWS Docs: AWS Ruby SDK](http://docs.aws.amazon.com/AWSRubySDK/latest/frames.html)
  * [AWS Docs: AWS Ruby SDK Class: AWS::EC2::ElasticIpCollection](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/EC2/ElasticIpCollection.html)
  * [AWS Docs: AWS Ruby SDK Class: AWS::EC2::ElasticIp](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/EC2/ElasticIp.html)
  * [AWS Docs: AWS Ruby SDK Class: AWS::Core::Configuration.initialize](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/Core/Configuration.html#initialize-instance_method)
  * [AWS Docs: AWS Ruby SDK Class: AWS::EC2](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/EC2.html)
  * [RubyGems: AWS Ruby SDK](http://rubygems.org/gems/aws-sdk)
  