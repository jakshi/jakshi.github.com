---
layout: post
title: "Chef cookbook templating with cookiecutter"
date: 2014-05-03 01:21
comments: true
categories: 
- chef
- cookiecutter
- python
- template
- technical
---

One of the challenges in chef cookbook development - create a comprehensive cookbook template.

Good cookbook template might:

* standartize your cookbooks
* fill them with convenient predefinitions
* save your time. 

If you are not python averse, you could find that [cookiecutter](https://github.com/audreyr/cookiecutter) pretty useful tool for creating your cookbooks templates. This article describes my experience in creating chef cookbook template with [cookiecutter](https://github.com/audreyr/cookiecutter).

<!-- more -->

# Cookiecutter in nutshell

* [cookiecutter](https://github.com/audreyr/cookiecutter) is a python based software.
* it uses [Jinja](http://jinja.pocoo.org/) templating engine
* To install cookiecutter read: [Installing Cookiecutter](http://cookiecutter.readthedocs.org/en/latest/installation.html).
* On my Ubuntu 12.04 workstation I used `pip cookiecutter` to install it.
* variables defined in `cookiecutter.json` file in the root of template directories hierarchy
* these variables might be used inside files by specifing {% codeblock %} {% raw %}{{cookiecutter.variable_name}}{% endraw %} {% endcodeblock %}
* these variables might be used as a file or directory names, if you call file or directory like: {% codeblock %} {% raw %}{{cookiecutter.variable_name}}{% endraw %} {% endcodeblock %}
* cookiecutter supports pre and post generate hooks.
* hooks might be shell or python scripts, have names `pre_gen_project.[sh,py]` or `post_gen_project.[sh,py]`, and should be in hooks dir.

# Chef cookbook templating with cookiecutter
## Create a directory for template

```
mkdir chef-cookbook-template
cd chef-cookbook-template
```

## Create a file with template variables

```
emacs cookiecutter.json
{
     "author": "John Smith",
     "email": "cookbooks@example.com",
     "cookbook_name": "",
     "company_name": "Example Ltd",
     "release_date": "2014-04-17",
     "year": "2014",
     "version": "0.0.1"
}
```

Value that specified in this file, will be suggested as a default one. 

As you see there's no value for "cookbook_name". So there will be no default value, you need to specify it every time you create new cookbook from this template.

## Create a templated directory for cookbook

{% codeblock %} {% raw %}
mkdir "chef-{{cookiecutter.cookbook_name}}"
{% endraw %} {% endcodeblock %}

## Create files for your cookbook.
On this stage you should fill your cookbooks with files that you need in your cookbook template. Use variables when it's suitable.

In my cookbook template I added variables to files:

* attributes/default.rb
* README.md
* spec/default_spec.rb
* metadata.rb
* {% raw %}{{cookiecutter.cookbook_name}}.packer{% endraw %}
* LICENSE
* Vagrantfile
* .kitchen.yml
* test/integration/default/serverspec/localhost/default_spec.rb
* recipes/default.rb

## Post-create hooks
It could be anything you want. For example I initialize my cookbook template with git.

In root directory of cookiecutter template add hooks directory.

```
mkdir hooks
cd hooks
emacs post_gen_project.sh
```

```
#!/bin/bash
command -v git >/dev/null 2>&1 && git init || { echo >&2 "git init is failed. Probably git is not installed. Install git if it's not installed."; exit 1; }
```

# Usage
I presume that:

* you finished your template
* and uploaded it to github

## Use template from github git repository

```
cookiecutter https://github.com/example/my-chef-cookbook-template.git
```

## Use it again.

  * After first usage template will be copied to ~/.cookiecutter directory.
  * So to use it again you need to specify template name only:

```
cookiecutter my-chef-cookbook-template
```

## Update template

```
cd ~/.cookiecutter/my-chef-cookbook-template
git pull
```

## Use template from local dir

* You can just store template in current directory and use it. 
* Let's presume that you have template in `my-chef-cookbook-template` dir. 
* Then to use template you just need to execute command:

```
cookiecutter my-chef-cookbook-template
```

# Tips and tricks

* If you want to include in resulting file {% raw %}{{ or/and }}{% endraw %}, you can specify them like {% codeblock %} {% raw %}{{'{{'}}{% endraw %} {% endcodeblock %} or/and {% codeblock %} {% raw %}{{'}}'}}{% endraw %} {% endcodeblock %}

# References

* [Cookiecutter](https://github.com/audreyr/cookiecutter)
* [Installing Cookiecutter](http://cookiecutter.readthedocs.org/en/latest/installation.html)
* [Jinja](http://jinja.pocoo.org/)
