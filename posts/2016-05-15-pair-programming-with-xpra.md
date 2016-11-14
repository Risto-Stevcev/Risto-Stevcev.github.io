---
title: Pair programming with Xpra (part 2)
description: How to set up graphical pair programming with Xpra
---

In the first part of this tutorial, I showed how you how two remote developers can SSH into a development environment and use `Tmux` to share their screens in a memory-efficient way. One drawback of this basic approach is that they are not sharing an `X server`, so they cannot share graphical environments like the browser, or an IDE.

In this second part, I'll show you how you can share a graphical environment in a memory-efficient way using `Xpra`. Xpra is a best approach to remote pair programming compared to X11-Forwarding or screen-sharing programs, because it's less invasive and substantially more secure.

In the first part of the tutorial I set up the environment using `Alpine Linux`, but in this part I'll use `Lubuntu` for brevity's sake, because manually setting up `X` properly can be tedious at the best of times. If you want to manually set up `X` on Alpine, go for it, the vesa driver is widely supported. The Alpine repos have Xpra, so you should be able to do all of the same stuff.


### Create Two VMs

Create two VirtualBox VMs and install `Lubuntu` on both. The installation process is self-explanatory. One of the instances will host the SSH server and the other will connect to it. If you want to be consistent with this guide, I'm using `foobar` as the name for the initial user for the box that will host the SSH server. Install SSH, start the server with `systemctl start sshd`,  and make sure that it's up and running: `systemctl status sshd`

The reason we need to have two instances for this second part of the tutorial is because we need to have two X servers running to show the screen sharing, and a machine can only have one X server running at any time.


### Using Host-Only Networking

Using the port-forwarding trick from part 1 is easy, but it can be limiting if we want to use the default SSH port and to provide real IPs to test the setup out. In this section, I'll use host-only networking to give the SSH server a real IP to connect to. If you don't want to use this method or you're having trouble setting it up, just use the trick from part 1 and make sure you're giving `Xpra` the correct `host:port`, which was `localhost:3022` in part 1.

First off, make sure that you disable the port forwarding for the VM if you left it on from part 1. 

From VirtualBox, go to `File > Preferences > Network > Host-only Networks`, and add a new host. Set the IPv4 to something like `192.168.57.1` with a netmask of `255.255.255.0`. Leave IPv6 stuff as is, and don't enable DHCP. 

Then, under the VM's settings, go to `Network > Adapter 2` (make sure the VM isn't running or this tab will be disabled), and select `Host-only Adapter` and select the adapter you made under name (it should be `vboxnet0` by default).

Start up your VM. Once it's up and running, type `ifconfig`. You should see two ethernet interfaces, one of them is the direct interface to your localhost, and the other is the host-only adapter. The one with the higher number (ie. `enp0s8` > `enp0s3`) should be your host-only adapter.

If `ifconfig` is showing that it has an IPv4 address, you don't need to do anything from here except SSH to that IP from the host. If it's not showing an IP, give it a static one. The high-level approach to this is to create an `ethernet-static` `netctl` profile that you can `start`/`stop`, and the low-level approach is to use `ip addr`/`ip route`. I'll take the middle path and just use `ifconfig`: 

```bash
$ ifconfig enp0s8 192.168.57.10 netmask 255.255.255.0 up
```

Then check `ifconfig` to see that the new IP has been assigned, and then you should be able to SSH to that IP from the host.



### Basic Xpra Session

First try `Xpra` in the simplest way, using the same account. First, go to the VM that's running the ssh server and type:

```bash
$ xpra start :100 --sharing --start-child=xterm
```

This creates a server instance of Xpra for DISPLAY `100` (choosing a higher display number is generally safer), `--sharing` to share the socket, and `--start-child` to tell it to start a program, `xterm`, on connecting. Alternatively, you can start the child manually like this:

```bash
$ DISPLAY=:100 xterm
```

Then, from the other VM and the host machine, connect to the `Xpra` server instance as the user that spawned it, `foobar`:

```bash
$ xpra attach ssh:foobar@192.168.57.10:100 --sharing
```

Now in both your host machine, and in the client VM, you should see that `xterm` opened up. If you type something in one of them, it will show up in the other machine! Just like `Tmux`. Since Xpra provides X, you can launch and multiplex graphical environments such as browsers or IDEs!

But this basic approach is limiting, because when we want to pair program remotely, we'll probably be SSH-ing into a dev environment as **different** users. In the next sections I'll show you how you can do that.



### Multiuser Part 1 (Quick and Dirty)

We'll eventually get to a solution that works for real-world remote dev environments, but it's good to keep things simple first and incrementally get to the goal because it eliminates more variables. In this case, we're eliminating permissions variables from the equation.

First let's create two more users. The VM that has the SSH server running has a `foobar` user, but I want to create two more users `quxnorf` and `worble` to simulate a real dev environment:

```bash
$ useradd --create-home --shell /bin/bash quxnorf
$ passwd quxnorf
$ useradd --create-home --shell /bin/bash worble 
$ passwd worble
```

Add them to sudoers if you wish, and **reboot the system** so that the changes are in effect. Try SSH-ing into the new users from the host machine to confirm that they work as expected.

Then, create a folder with full access permissions for simplicity's sake:

```bash
$ mkdir /xpra
$ chmod 777 /xpra
```

And then start up an Xpra server instance, this time telling Xpra to store the socket in the new `/xpra` folder:

```bash
$ xpra start :100 --sharing --start-child=xterm --socket-dir=/xpra
```

Assuming you set the machine name to `foobar` and display to `:100`, the socket that was created should have a name like `foobar-VirtualBox-100`. The other users need to have read/write permissions on the socket also.

During the time of this post, the Xpra version that comes with Ubuntu Trusty (0.14.25) doesn't have the ability to set the permissions on the socket as a flag, but that option is available on the bleeding-edge versions. For obvious reasons, a lot of Linux power users are having a nerdgasm about Xpra, so it's rapidly being developed.

In any case, that's not going to stop me from hacking Xpra to get it to do what I want. So lets update the permissions of the socket:

```bash
$ chmod a+r /xpra/foobar-VirtualBox-100
$ chmod a+w /xpra/foobar-VirtualBox-100
```

Finally, log into to the Xpra instance from the other VM as `quxnorf`:

```bash
$ xpra attach ssh:quxnorf@192.168.57.10:100 --sharing --socket-dir=/xpra
```

And then login from your host as `worble`:

```bash
$ xpra attach ssh:worble@192.168.57.10:100 --sharing --socket-dir=/xpra
```

You should see that both the VM and the host machine are sharing and multiplexing the same thing!


### Multiuser (The Better Way)

For a real dev environment, we'll want to set the permissions in a better way. So in this section I'll create those kind of permissions.

First, add the current users on the VM to a new group called `xpraers`: 

```bash
$ sudo groupadd xpraers
$ usermod -a -G xpraers foobar
$ usermod -a -G xpraers quxnorf
$ usermod -a -G xpraers worble
```

Restart so the changes come into affect. Then, give execute permissions for the folder to the new group:

```bash
$ chown foobar:xpraers /xpra
$ chmod 775 /xpra
```

Test that you can `touch` a file in the `/xpra` folder as one of the users above. Then, start the Xpra instance like the previous section:

```bash
$ xpra start :100 --sharing --start-child=xterm --socket-dir=/xpra
```

And then give the socket read/write permissions for the `xpraers` group:

```bash
$ chmod 660 /xpra/foobar-VirtualBox-100
$ chown :xpraers /xpra/foobar-VirtualBox-100
```


Finally, log into to the Xpra instance from the other VM as `quxnorf`:

```bash
$ xpra attach ssh:quxnorf@192.168.57.10:100 --sharing --socket-dir=/xpra
```

And then login from your host as `worble`:

```bash
$ xpra attach ssh:worble@192.168.57.10:100 --sharing --socket-dir=/xpra
```

That's it! Now you can go ahead and do remote pair programming the way it should be. At this point it should be straightforward how you can set this up with Docker too. I showed this with three users to show separation of concerns, but this can be done with just `quxnorf` and `worble` remote users, where for example `quxnorf` is SSHed and does what `foobar` did, and then follows the instructions as usual.

I hope you enjoyed this tutorial as much as I did. Xpra is awesome, and since it's still currently at major version 0, I can't wait to see what other changes come in the future! 
