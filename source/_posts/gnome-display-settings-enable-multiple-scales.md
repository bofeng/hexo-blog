---
title: 'Gnome display settings enable multiple scales'
date: 2020-04-17 05:18:15
tags: [ubuntu,linux]
published: true
hideInList: false
feature: 
isTop: false
---
I was settings up my new System76 laptop (my current Pop!_OS version is based on ubuntu 19.10), the default 1920x1680 resolution is too small for me in the display settings. So I want to make it bigger. There is a "Scale" settings below, and it only has 2 options: 100%, 200%

100% is too small for me, 200% is too big.

To enable other scales, open terminal run command:
```bash
$ gsettings set org.gnome.mutter experimental-features "['x11-randr-fractional-scaling']"
```

Then back to display settings, you will be able to see other scale values: 125%, 150% and 175%.

![](/post-images/1587158852197.png)


### Reference:
* [https://www.linuxuprising.com/2019/04/how-to-enable-hidpi-fractional-scaling.html](https://www.linuxuprising.com/2019/04/how-to-enable-hidpi-fractional-scaling.html)
