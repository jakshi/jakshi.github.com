---
layout: post
title: "Resolving slow wifi issue with Centrino N 1030 BGN in Ubuntu 12.04 LTS"
date: 2012-11-26 12:03
comments: true
categories: 
  - technical
  - workaround
  - quickfix
  - linux
  - ubuntu
  - ubuntu 12.04
---

# Introduction

I've bumped into issue with Intel Centrino Wireless-N 1030 BGN WiFi adapter under Ubuntu Linux 12.04 LTS.
It works extremely slow with some WiFi AP.


# How to fix
After some investigation I clarify that you need to switch off 802.11n for Centrino Wireless-B 1030 BGN WiFi adapter, then adapter will work good with all AP.
I think that some of AP realize 802.11n protocol in not correct way, And this confuses Centrino WiFi adapter.

## Switch off 802.11n

    sudo rmmod iwlwifi
    sudo modprobe iwlwifi 11n_disable=1

## Switch on 802.11n

    sudo rmmod iwlwifi
    sudo modprobe iwlwifi 11n_disable=0

## Permanently switch off 802.11n

    emacs /etc/modprobe.d/iwlwifi-disable11n.conf
    options iwlwifi 11n_disable=1

# Further reading

  * [How do I fix slow wireless on Intel wireless cards?](http://askubuntu.com/questions/130499/how-do-i-fix-slow-wireless-on-intel-wireless-cards)