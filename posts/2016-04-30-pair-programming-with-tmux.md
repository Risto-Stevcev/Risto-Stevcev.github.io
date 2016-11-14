---
title: Pair programming with Tmux (part 1)
description: How to set up simple pair programming with Tmux
---

As my first blog post, I thought I'd share a cool way to do pair programming with terminal mulitplexing. More and more developers are working for home, and so it's nice to know some quick and easy ways to set up a remote pair programming session.

Terminal multiplexers provide a way of sharing multiple virtual consoles. There are a lot of different ones out there, but the two most popular one's are Tmux and GNU Screen. Both are good, but for this blog I'll be using Tmux because it's slightly easier to see what's going on.

### Initial Test Run

Install Tmux, and then fire up two different terminal windows. In the first terminal, type in:

```bash
$ tmux new -s mysession
```

You'll see it fire up Tmux, which will look like the shell you just had, except at the bottom it will say the name you chose for your session (mysession in this case) and the shell program you're using (for me it's bash).

Now attach your second terminal to the session you created:

```bash
$ tmux attach -t mysession
```

You'll see the same thing happen in your second terminal window. Now try typing something in one of the windows. You'll see it start happening in the other window!

This way, you can share a screen, fire up Vim or Emacs, split a console, and do some pair programming easily.

Let's get a sense of what it's doing:

```bash
$ ps au | grep tmux
risto   11492  0.0  0.0  19552  2888 pts/10   S+   11:54   0:00 tmux new -s mysession
risto   11499  0.0  0.0  19552  2960 pts/7    S+   11:55   0:00 tmux attach -t mysession
```

From just this process info, we can gather a lot about what it's doing under the hood. We can see that our two separate pseudoterminal instances, `pts/10` and `pts/7`. We can also see that the two tmux instances are not the same process, but belong the same process group, so it's doing IPC under the hood.

But it's one thing to do a simple demo like this, and it's another to set it up in an actual real-life scenario, so I'm going to talk about how to do that next.

### Testing Out A Real-Life Scenario

It's good to know how you would set this up in a real-world scenario. So in this section I'll talk about how to simulate that with a virtual machine.

First off, install virtualbox

For this post, I've decided to do it with [Alpine Linux](http://alpinelinux.org), because the image is small and so trying this out makes it less problematic if you're worried about the disk space a full linux install would take up on your harddrive. But feel free to use whatever you want. Alpine Linux uses Busy Box, which is like a stripped down version of a lot of main utility toolchain for linux.

Download the 32-bit Alpine Linux instance from their website. You can check that you're download completed correctly by checking the sha256sum:

```bash
$ sha256sum -c ~/downloads/alpine-3.3.3-x86.iso.sha256sum
```

### Setup Alpine Linux

Create a new virtualbox instance. Set it to 32-bit Linux. The rest could just be the defaults.

Mount the iso of Alpine Linux you downloaded to your new image, and then run the image. It should load Alpine Linux. Once it's done loading, it should give you a shell prompt.

Run setup-alpine, and just use all of the defaults. Set a root password. For which disk you want to use, type "sda", and for how you want to use it, type "sys".

Once it's done installing, unmount the cd and reboot.

### Configure The System

Alpine should load up, and you can login as root with the password you set in the install. You should be connected to the internet out of the box, and you can check with ping.

First off, you'll notice that it's preinstalled with ssh and has a server already up and running:

```bash
$ rc-status
sshd               [started]
```

Lets add two users to simulate a real ssh environment:

```bash
$ adduser -g -S -s /bin/ash foo
$ adduser -g -S -s /bun/ash bar
```

The -g flag (GECOS field) will just allow you to set a password without typing (`passwrd [user]`). The -S flag sets it as a system user, and -s is the shell to run at login (Alpine uses ash).

The next part is optional, but it's nice to have sudo permission. Lets add sudo:

```bash
$ apk add sudo
$ visudo
```

Add an entry for foo:

```
foobar ALL=(ALL) ALL
```

Then, lets update the system. Make sure that the `3.3/main` repository is uncommented under `/etc/apk/repositories`, since Tmux lives in that repo, and then run:

```bash
$ sudo apk update
$ sudo apk upgrade
$ sudo apk tmux
```

Try launching Tmux to make sure it works. Now, we need a way to ssh into the guest virtual machine from the host computer. Click Settings for the virtual machine from the virtualbox main window, and go to the Network section. You should see tabs such as Adapter 1, Adapter 2, etc, and for Adapter 1, it should say that it's Attached to: NAT. Click the Advanced dropdown, and click Port Forwarding. Add a new rule where the protocol is TCP, the Host port is `3022` and the Guest port is `22`, and the rest blank.

You should now be able to ssh into your box via port `3022`:

```bash
$ ssh -p 3022 foo@localhost
```

If ssh complains that there's a mismatch or security vulnerability, it might be that you already have `@localhost` in your `~/.ssh/known_hosts` file. If so, you would need to remove that entry, or follow part two of this tutorial which details a more powerful way of setting it up but requires more configuration.

You can now ssh as foo in two different places, and then use Tmux like it was described earlier. And if that's fine (both users using the same account), then that's all you need. But if you need two different users to be able to pair program (more likely), then we need to create a user group for them:

```bash
$ addgroup tmuxers
$ addgroup foo tmuxers
$ addgroup bar tmuxers
```

And we need to create a directory to house the tmux socket that both users will share. Both users need to have write permissions on the folder, so we can do that by making tmuxers the group owner and adding write permission for the group:

```bash
$ mkdir /tmux
$ chown foo:tmuxers /tmux
$ chmod g+w /tmux
```

If you're ssh-ed in, you might need to logout/exit for the permissions to update. This time we need to run Tmux with a shared socket. Since there might be more users on our ssh server that are part of the `tmuxers` group, we should give the socket a descriptive name, like `foo-bar-socket`. From your host computer, ssh into the server as `foo@localhost` and type:

```bash
$ tmux -S /tmux/foo-bar-socket new -s mysession
```

And then in another terminal window from the host computer, ssh into the server as `bar@localhost` and type:

```bash
$ tmux -S /tmux/foo-bar-socket attach -t mysession
```

You'll now see that both users are attached to the same Tmux instance!

### Getting Fancy

You can set up your remote dev environment in a multitude of ways. Developers can ssh into the development environment, which would have their git instances that they push to the CI environment. They could use Docker to run the dev instance, which would be identical to the CI instance and identical to the production instance. Which would make dealing with Dev Ops much nicer.

And finally, you might be thinking, **what if I want to run GUI apps?**

For Vim and Emacs users, you can run your browser tests headlessly using an x-video-framebuffer (or xvfb), or with PhantomJS. But if you and your pair developer are IDE users, or if you need an IDE for some other reason, then you'll definitely love `Xpra`.

Stay tuned for part two of this tutorial, where I'll explain how you can achieve that same kind of behavior with GUI apps! And the nice thing with Xpra is that it doesn't use X11 forwarding, so it's not vulnerable to keystroke sniffing, and it's not invasive, slow and vulnerable like other screen-sharing techniques. And you you're not forced to set up your screen like your peer.
