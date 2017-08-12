---
layout: post
title: SSH Agents
date: '2012-04-21 20:13:56'
tags:
- ssh
---

If you&rsquo;re an engineer like me, you like to automate things. Typing the same commands over and over can get old. One of those repetitious tasks is using <a href="http://en.wikipedia.org/wiki/Secure_Shell">ssh</a> to connect to other hosts. At my work I have about 10 different physical boxes I need to access, and of course, some virtual machines as well. This becomes a problem for a number of reasons.

Surprisingly, ssh comes with some bells and whistles that should help you get up and running. Once we&rsquo;re done you should be able to access your hosts without having to know the ip address, user name, or password. You can even give each box its own unique name.

#### Generate Keys

The first thing you&rsquo;ll need to do is generate private and public keys to use for authentication. Authenticating with keys will keep us from having to type in the password of the remote host every time. We&rsquo;ll still have to type a password, but it will be the passphrase associated with your key and not the host you&rsquo;re connecting to. That way you can use the same password for all of your hosts.

#### Run SSH Keygen
```
[demouser@athene ~]$ ssh-keygen -t rsa -C demouser@emailservice.com
Generating public/private rsa key pair.
Enter file in which to save the key (/home/demouser/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/demouser/.ssh/id_rsa.
Your public key has been saved in /home/demouser/.ssh/id_rsa.pub.
```

<p>In the above example we have generated an RSA key pair for the user &ldquo;demouser&rdquo;. Make sure to enter a passphrase instead of hitting enter. It is tempting not to provide a passphrase, but not very wise. If someone gained access to your computer they could connect to any server you are authenticated against. Later in the post we&rsquo;ll talk about ssh-agent which will keep you from having to type your password over and over.</p>

#### Authorizing Connections to Remote Hosts
Now that we have our keys generated, we&rsquo;ll have to tell our other hosts about them so that we can use them as login credentials. To do that we&rsquo;ll use `ssh-copy-id` which will insert our public key into the remote host&rsquo;s authorization file (`~/.ssh/authorized_keys`).

```
[demouser@athene ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub remoteuser@remotehost
remoteuser@remotehost's password: 
Now try logging into the machine, with "ssh 'remotehost@remoteuser'", and check in:

  ~/.ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.
```

If everything went as planned, then the next time you login to your remote host, you should be prompted for your key&rsquo;s passphrase instead of the remote user&rsquo;s password.

```
[demouser@athene ~]$ ssh remoteuser@remotehost
Enter passphrase for key '/home/demouser/.ssh/id_rsa': 
Last login: Sat Apr 21 20:19:22 2012 from athene
remoteuser@remotehost ~$ 
```

If this didn&rsquo;t work, chances are that the permissions on your remote host&rsquo;s authorization file are incorrect. Permissions for the file should be set so that the user can read and write, while group and other have no access to the file.

```
$ chmod 600 ~/.ssh/authorized_keys
```

#### Remembering Host Login Information

Host information can be stored in `~/.ssh/config` which we&rsquo;ll have remember our user name and ip address. Lets say that I wasn&rsquo;t using DNS, or the host name of my remote host was too long to remember easily.

In the following example I have a host with the ip address `192.168.1.10`. My ssh config might look something like this.

```
Host host1 192.168.1.10
    HostName 192.168.1.10
    User remoteuser
    Protocol 2

Host *
    ForwardAgent yes
    ForwardX11 yes
```

Now instead of having to connect like this:

```
$ ssh remoteuser@192.168.1.10
```

I can now connect like this:

```
$ ssh host1
```

#### Remembering Your Passphrase with SSH-Agent

A little-known tool that will help remember your passphrase is the `ssh-agent`. Chances are that you already have this daemon running if you are running a Linux desktop like Gnome or KDE.

#### Check if ssh-agent is running

```
pgrep ssh-agent
```

If you see the ssh-agent process, then you&rsquo;re halfway there. If not, then you can run ssh-agent like this:

```
$ eval `ssh-agent`
```

When running the daemon, <em>eval</em> is used because the agent will spit out some shell script that we want to run. Certain environment variables need to be set in order for the agent to work. If we run the agent without doing an `eval` we&rsquo;ll get something like this:

```bash
SSH_AUTH_SOCK=/tmp/ssh-xLOirks18528/agent.18528; export SSH_AUTH_SOCK;
SSH_AGENT_PID=18529; export SSH_AGENT_PID;
echo Agent pid 18529;
```

Once your agent is running you might want to check your environment to make sure that the proper environment variables got exported.

```
$ env | grep SSH_
```

After getting the agent running, we&rsquo;ll need to add our identity which is associated with our public and private keys that we generated earlier. You can list the identities that the agent currently knows about by running:

```
$ ssh-add -L
```

If you don&rsquo;t see any output, that&rsquo;s because the agent doesn&rsquo;t know about any identities. In order to add your identity, just run `ssh-add`> without any arguments.

```
$ ssh-add
```

Now that your agent is running and aware of your identity, you shouldn&rsquo;t have to type your passphrase anymore! Ah, the virtues of laziness.

#### Resources

<ul><li><a href="http://mah.everybody.org/docs/ssh">http://mah.everybody.org/docs/ssh</a></li>
<li><a href="http://ornellas.apanela.com/dokuwiki/pub:ssh_key_auth">http://ornellas.apanela.com/dokuwiki/pub:ssh_key_auth</a></li>
<li><a href="http://help.github.com/ssh-key-passphrases/">http://help.github.com/ssh-key-passphrases/</a></li>
</ul>