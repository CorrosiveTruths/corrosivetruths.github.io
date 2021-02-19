---
layout: post
title:  "Zram Setup"
date:   2020-11-20 12:58:00 +0000
author: Peter
published: true
categories:
- article
---
People have a habit of over-specifying parameters, zram is particularly bad for it, but really, you don't need to tell it a lot and it will take care of it for you.

The other misunderstanding with zram is that when you set the size you are setting how big the swap partition itself is, rather than how much memory it is allowed to use. You can't really set how much memory to devote to zram as that depends on the compression ratio.

For example, setting up zram like tmpfs with half the size of your memory, will in tmpfs' case mean that it will use up to half your memory. With zram it means you're limiting the swap used to half your memory, if you get a compression ratio of 3:1 then you'll be using 1/6 of your memory for swapping. Which likely isn't what you want. Instead, you should set your zram up so it can compress much more of your memory than that.

I would recommend setting up zram at two to three times your memory size instead as there isn't a lot of point in going bigger than your compression ratio of your memory.

In my case, performance was more important than compressibility, so I'm using the lz4 algorithm - and because that's also not going to net you higher compression ratios, I stick to two times my memory size.

An extreme example of zram usage involves compiling chromium.

An average compression ration of just over 3 was attained with a maximum used swap of 15GiB for an 8-thread Chromium (86.0.4240.193) compile so it works in practice and under extreme conditions. The computer did slow down during the process, but not nearly as badly as with disk-based swap of course.

Personally, I only use a single zram for swap as that will improve the system as a whole. A tmpfs /tmp directory will swap out when pressured and I keep /var/tmp as disk-based because I'd have to write exceptions for big packages and there's only a negligible advantage to putting it into memory over letting the kernel cache it.

You need the module loading at start-up and a service file (you can of course adapt this for other init systems).

```bash
#/etc/modules-load.d/zram.conf
zram
```

```bash
# /etc/systemd/system/zram-setup.service
[Unit]
Description=Setup zram

[Service]
Type=oneshot
RemainAfterExit=true

ExecStart=/sbin/zramctl -a lz4 -s 16G /dev/zram0
ExecStart=/sbin/mkswap -L Swap /dev/zram0
ExecStart=/sbin/swapon -L Swap -p 100
ExecStop=/sbin/swapoff /dev/zram0

[Install]
WantedBy=multi-user.target
```
