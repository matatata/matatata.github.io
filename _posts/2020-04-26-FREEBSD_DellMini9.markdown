---
layout: post
title:  "FreeBSD 12.1 on Dell Mini 9"
date:   2020-04-26 12:03:15 +0200
categories: jekyll FreeBSD
---


# FreeBSD

## Dell Mini9

To get a higher resolution in textmode install a kernel module for the internal Intel GMA GPU
	
	pkg install drm-kmod
	
and **load** it in /etc/rc.conf

	kld_list="/boot/modules/i915kms.ko"


### Admin
To add an existing user to the group *wheel* in order to allow him top use *sudo* and/or to become root

	pw group mod wheel -m johndoe
	

	


### Unfortunately sleep does not work! So it's almost useless for me.
