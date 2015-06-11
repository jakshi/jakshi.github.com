---
layout: post
title: "Chef shortcuts"
date: 2014-11-21 13:16
comments: true
categories: 
  - chef
  - technical
  - knife
  - devops
---

# Add a recipe to the end of run list on certain environment

Test run, not actually add a recipe:

```
knife exec -E 'nodes.transform ("chef_environment:beta") {|n| puts n.run_list << "recipe[logentries_ng]" }'
```

Add a recipe for real:

```
knife exec -E 'nodes.transform ("chef_environment:beta") {|n| puts n.run_list << "recipe[logentries_ng]"; n.save }'
```

<!-- more -->

# Add a recipe to the beginning of run list on certain environment

Test run, not actually add a recipe:

```
knife exec -E 'nodes.transform ("chef_environment:qa01") {|n| old_rl = n.run_list.to_a; puts n.run_list(["recipe[datadog::dd-handler]"] + old_rl) }'
```

Add a recipe for real:

```
knife exec -E 'nodes.transform ("chef_environment:qa01") {|n| old_rl = n.run_list.to_a; puts n.run_list(["recipe[datadog::dd-handler]"] + old_rl); n.save }'
```

# Insert a recipe in the second position of run list on certain environment

Test run, not actually add a recipe:

```
knife exec -E 'nodes.transform ("chef_environment:qa04") {|n| old_rl = n.run_list.to_a; puts n.run_list(old_rl[0..0] + ["recipe[datadog::dd-handler]"] + old_rl[1..-1]) }'
```

Add a recipe for real:

```
knife exec -E 'nodes.transform ("chef_environment:qa04") {|n| old_rl = n.run_list.to_a; puts n.run_list(old_rl[0..0] + ["recipe[datadog::dd-handler]"] + old_rl[1..-1]); n.save }'
```

# Remove a recipe from run list on certain environment

Test run, not actually remove a recipe:

```
knife exec -E 'nodes.transform ("chef_environment:qa02") {|n| puts n.run_list.remove ("recipe[logentries_ng]") }'
```

Remove a recipe for real:

```
knife exec -E 'nodes.transform ("chef_environment:qa02") {|n| puts n.run_list.remove ("recipe[logentries_ng]"); n.save }'
```

# Execute chef-client on all nodes in specific environment

```
knife ssh 'chef_environment:qa02' 'sudo chef-client'
```

# Search for nodes in specific environment and having specific recipe
And show only node names (-i option)


```
knife search node "chef_environment:production AND recipes:web_server" -i
```

# Get list of all uniq non-system usernames that exists in specific environment

```
knife exec -E 'users = []; nodes.find("chef_environment:production") {|n| n[:etc][:passwd].select { |user, options| options[:uid] >= 1000 }.each { |user, options| users << user} }; users.uniq.each { |user| puts user } '
```

# More shotcuts

Do you want even more shortcuts? Read nice article: [Knife Tricks](http://dougireton.com/blog/2013/02/03/knife-tricks/)