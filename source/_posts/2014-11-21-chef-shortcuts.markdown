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

# Add a recipe to run list on certain environment

Test run, not actually add a recipe:

```
knife exec -E 'nodes.transform ("chef_environment:beta") {|n| puts n.run_list << "recipe[logentries_ng]" }'
```

Add a recipe for real:

```
knife exec -E 'nodes.transform ("chef_environment:beta") {|n| puts n.run_list << "recipe[logentries_ng]"; n.save }'
```

<!-- more -->


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

Want even more shortcuts? Read nice article: [Knife Tricks](http://dougireton.com/blog/2013/02/03/knife-tricks/)