---
title: "Automating Cisco IOS Upgrades: A Step-by-Step Ansible Guide (INSTALL MODE)"
seoTitle: "Cisco IOS Automation with Ansible Guide"
seoDescription: "Automate Cisco IOS upgrades with Ansible using Install Mode for modern routers. Step-by-step guide included"
datePublished: Wed Dec 31 2025 07:04:07 GMT+0000 (Coordinated Universal Time)
cuid: cmjto7cnb000l02js8xufcg8u
slug: automating-cisco-os-upgrade-ansible-bundle-mode-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1766971368080/768f8d50-557c-49f0-bad1-db8ce6dfa0b9.png
tags: ansible, cisco, os-upgrades

---

## Introduction

If you’ve been following my blog, you might have seen my earlier guide on [**Bundle Mode**](https://blogs.hestinetlab.com/automating-cisco-ios-xe-upgrade-ansible) upgrades. That method is classic-simple and familiar. But let’s be honest, if you are managing modern Catalyst 9000s or ISR routers, Install Mode is the standard we should all be aiming for.

Unlike the legacy method where we simply pointed the boot variable to a **.bin** file, Install Module extracts the packages into the flash…

Here is how I do it use workflow in Ansible Tower.

## Diagram

I use the same diagram as [**Bundle Mode**](https://blogs.hestinetlab.com/automating-cisco-ios-xe-upgrade-ansible) as well

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767164027716/43f59a8c-9358-4555-8692-4a2b25721b8c.png align="center")