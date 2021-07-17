---
title: "The journey to controlling external monitors on M1 Macs"
subtitle: "How the transition to Apple Silicon made all monitor-controlling apps useless overnight, and how Lunar got past that"
excerpt: |-
  When M1 Macs where launched, a new GPU with iOS-like architecture started driving the external monitors.

  This meant that the old method for controlling monitors using DDC wouldn't work anymore.

  Apps like [Lunar](https://lunar.fyi) and [MonitorControl](https://github.com/MonitorControl/MonitorControl) had to find other ways to change the brightness of the monitors.
date: 2021-07-16T18:39:37+03:00
draft: true
author: Alin Panaitiu
image: m1-monitors.png
pageStyles: 
  - file: blog.sass
    media: "(prefers-color-scheme: light), (prefers-color-scheme: no-preference)"
  - file: blog-dark.sass
    media: "(prefers-color-scheme: dark)"
---

