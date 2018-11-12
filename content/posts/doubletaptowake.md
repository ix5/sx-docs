---
title: "Implementing double-tap to wake"
date: 2018-11-07T01:18:00+01:00
draft: true
author: Felix
---

## The journey

In kernel 3.x, double-tap to wake works. It even works in TWRP!
So why doesn't it work on Pie anymore?

## Sleuthing
Display is: `<display-model>`, incell-hybrid
New SDE(Snapdragon Display Engine)
Drivers: incell, clearpad, headers

Kernel got upgraded to 4.9

Also: Kernel stuff, we need to ditch the prebuilt dts files and start building
our own!

Then, up the stack, an xml file
