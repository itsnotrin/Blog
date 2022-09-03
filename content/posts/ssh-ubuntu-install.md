---
title: "Installing Ubuntu... over SSH?"
date: 2022-09-03T11:45:39+01:00
description: "Deploying my new Jellyfin and general services server!"
draft: false
---

I recently purchased a little shitbox PC off of Facebook Marketplace for just £50!

It comes with an older i3 (2c 4t) and 8GB of RAM. My initial plan was just to throw arch on it and have it as a portable Arch machine I can remote into from my laptop- this didn't work out though as it was a lot larger than I thought (I thought it was one of those very very small ones 😂)

My next idea was to just sell it on and get myself a small PC like I originally intended. This wouldn't work out either as the PC is VGA only (it has USB 3 though??) and I don't have a VGA cable.

This brought me to now.
As I'm sure you're all aware, the energy prices are shooting up everywhere, it's a madness! 
The server I was currently using at my home (16c 32t, 32GB RAM, 2ish TB of raw storage) chews through electricity like it doesn't matter at all, pulling nearly 100 watts at idle and a lot more than that under load.

Lately, I've only really been using it as a jellyfin server though- which it is certainly overkill for- that's where this idea came from. I can replace my Raspberry Pi Zero First Generation (yikes) and my server with this PC - which only pulls 15 watts under full load!

However, the VGA problem was still very prominent.
I did some research and stumbled across [FAI](https://fai-project.org/FAIme/) - Build your own customised ISO! This was perfect for me as it allowed me to automate the install and enable SSH etc, like everything I need! I entered the info I wanted, tested it in a VM and it shot straight up in the VM!

This seemed perfect- I threw it in the PC and I waited... and waited... and waited. Nothing.

Reboot, nothing. It refused to boot and I couldn't plug into a monitor to see the issue.

This annoyed me a lot so I took a little break- now we're onto today :)

Another option I stumbled across was [this blog from 2021](https://zameermanji.com/blog/2021/9/9/installing-ubuntu-20-04-over-ssh/). They too had tried other methods and gotten annoyed however they posted something I'm certain is going to work- nocloud cloud-init. You provide the username and password you'd like to use and you can SSH into the installer from another PC.

I gave this a try and entirely wrote it out in this blog- just for it to not work. What a waste of my time!

After a lot of googling and stack overflowing, I found an [official guide from Ubuntu themselves](https://ubuntu.com/server/docs/install/autoinstall-quickstart)!

Perfect!

Or so I thought. It doesn't go into detail regarding how to do it outside of KVM - I don't want to run it in KVM so this didn't work for me either.

So queue another while of StackOverflow searching- I stumbled across [this](https://www.pugetsystems.com/labs/hpc/How-To-Make-Ubuntu-Autoinstall-ISO-with-Cloud-init-2213/)!

I downloaded the Ubuntu 20.04 Server ISO from my collection [screenshot](https://i.imgur.com/thP72NK.png)

For the prerequisites, I just had to install `cloud-init`, which I did with the command `sudo apt-get install cloud-init`.

Now, we must do the user-data and meta-data files.
meta-data is simple. It's just an empty file!
For user-data however, we must add some information.
```
#cloud-config
chpasswd:
    expire: false
    list:
        - installer:$6$exDY1mhS4KUYCE/2$zmn9ToZwTKLhCw.b4/b.ZRTIZM30JZ4QrOQ2aOXJ8yk96xpcCof0kxKwuX1kqLG/ygbJ1f8wxED22bTL4F46P0
```
The crypted password is just `ubuntu`

Now, we use [this handy script from GitHub](https://github.com/covertsh/ubuntu-autoinstall-generator) in order to build the ISO

We run the command `./ubuntu-autoinstall-generator.sh -a -u user-data -d test.iso -s ubuntu-server_20.04.iso -k`

Out of this, you get an ISO!

I threw it in a VM, got the IP from my router's DHCP reservations, SSHd in with the username installer and password `ubuntu` and it worked!
[image](https://i.imgur.com/O42gJk7.png)

Finally!

Now... to throw it in the PC.
Booted straight up and I've now configured the PC- now to set it up!

I may post a blog regarding setting it up (Nginx Proxy Manager, Migrating Old Jellyfin etc etc) so keep an eye out!

Keep cool :)