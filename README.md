# SSH Tunnel Remote Access

## Introduction
Some instructions on how to remotely access a server by tunneling through your cloud server via SSH.

**The motivation**: I have a Mac mini in the office which I would like to remotely access at home using screen sharing, without having to buy or install any additional software.

## Prerequisites
I am assuming that you are using a Mac (local server) to connect to another Mac (remote server) via your Linux-based cloud server. You need to have

- Cloud server with IP address and SSH access (I have a simple DigitalOcean setup)
- Password protection for the above, otherwise anyone can open a tunnel into your remote machine and you would not want that.

## Architecture
What is going on here? We're going to set up an SSH tunnel from my MacBook Pro (local server) into my cloud server. And another SSH tunnel from my Mac mini (remote server) into my cloud server. My cloud server will serve as the link between the two. There are numerous benefits to using SSH tunneling to facilitate this connection, especially when your remote IP is non-static, or you just don't have access to the remote network or don't want to expose it to the internet (or like me, too much work)

The context here is based on screen share so I will be using VNC port `5900` in examples. You can of course open any port you want.

I use ssh `config` files to get the job done. If you look at tutorials online they'd use mumbo jumbo Terminal commands which don't quite make sense. Basically, you would need to define a config file each for your remote and local servers.

## SSH Config

Rename the files in the directory to simply `config`, and place them in your `~/.ssh` directory.

### Remote

```ssh
Host hostname
  User username
  HostName 0.0.0.0
  RemoteForward 5900 127.0.0.1:5900
```

In the remote server, we will use `RemoteForward`. What this does is that it forms a tunnel from port `5900` (VNC) on my remote server to port `5900` of my cloud server at `0.0.0.0`. If you have a domain name service (DNS) mapped to this IP you can use it in place of the cryptic IP address.

### Local

```ssh
Host hostname
  User username
  HostName 0.0.0.0
  LocalForward 5900 127.0.0.1:5900
```

In the local server, we will use `LocalForwarding`. Similar to remote forwarding, but we just want to link our port `5900` to the same port in the cloud. You have to explicitly state that local forwarding is going on here otherwise the cloud server will not be paying attention to your local port `5900`.

### SSH Keys

These are completely optional and save you the step of having to enter your password each time you open the tunnel. I have added support for them in the `config` files but feel free to glaze over them if you don't need them.

Basically, all you need to do is to pipe your `id_rsa.pub` files on your remote and/or local servers (or whatever key you want, I used `id_rsa` because it's default) into `~/.ssh/authorized_keys` on your cloud server.

My `authorized_keys` file looks like this

```ssh
ssh-rsa ry8nq3c7wyt743478wnyy...

ssh-rsa 3247y5hq8cinwaosaxnfj...

```

### Why?
Why use `LocalForward` on my local server, and `RemoteForward` on my remote server? Because the VNC client you are going to be using is found in your local server. These are within the same firewall, so to speak, so the port mapping is done **locally**. You could technically switch the two up and VNC from your remote server back into your local server.

~~TL;DR~~ `RemoteFoward` sends the connection **out**. `LocalForward` maps the connection **within**.

## Remote Access
Start talking by running

```shell
ssh hostname
```

on both remote and local machines. Of course, replace `hostname` with whatever you called it in your config file.

Once you have both servers talking to each other successfully, you should be able to fire up Mac's secret *Screen Sharing* app by hitting <kbd>⌘</kbd>+<kbd>K</kbd> in Finder on your local server.

Assuming you are using port `5900`, type in `vnc://localhost:5900` and hit <kbd>Connect</kbd>. You may be prompted to enter the Username and Password of your remote server.

Note that the SSH tunnel must be open on both servers for the VNC to continue running. I would suggest keeping it running on your remote server, in non-interactive mode.

```shell
ssh -nNT hostname
```

## Conclusion

That is it, you are done with quite a secure and free way (besides hosting your cloud server) to remotely access your Mac from some place else.
