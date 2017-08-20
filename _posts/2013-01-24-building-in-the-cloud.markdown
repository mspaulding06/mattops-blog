---
layout: post
title: Building in the Cloud
date: '2013-01-24 03:48:00'
tags:
- python
- fedora
- devops
---

Recently a colleague of mine wrote a blog about <a href="http://neilsoman.blogspot.com/2013/01/this-is-article-about-using-eucalyptus.html">how we do builds</a> here at Eucalyptus. It&rsquo;s an accurate overview of the life-cycle of a continuous integration build in the cloud and an excellent read. Leveraging the power of the Eucalyptus cloud we can now horizontally scale our build farm. But while we have been able to do that, there are still places where we need to optimize the build process.

One of those places involves the use of Mock. Mock is the RPM chroot build environment used to build packages for <a href="http://fedoraproject.org/">Fedora</a> and also <a href="http://fedoraproject.org/wiki/EPEL">EPEL</a> projects. When building Eucalyptus we use a pristine Mock environment to ensure that all builds are cruft free. A Python wrapper script called <a href="https://github.com/gholms/rpmfab">rpmfab</a> is used to simplify the process.

Here&rsquo;s an example snippet from Jenkins that would build a Eucalyptus RPM package:

```
$ rpmfab/build-arch.py -r $platform-$arch -o results
    --mock-options "--uniqueext $BUILD_TAG" *.src.rpm
```

In this example, the value of `$platform-$arch` gets passed along to Mock as the argument to the `--root` cli option. So what does Mock do with that information? Well, it constructs a configuration file name using the default configuration directory location so that it comes up with something like this:

```
/etc/mock/$platform-$arch.cfg
```

While this works just fine in most basic cases, things start to get weird if you try to perform more advanced builds. What if you need to build something that depends on custom RPMs that you&rsquo;ve built? That&rsquo;s not hard, you say. Just go ahead and add a new configuration to `/etc/mock` that includes your custom repository. But what if the repository you want to include is different depending on what branch of the code you&rsquo;re building? Maybe one branch uses bleeding edge dependencies, while the more stable build does not. That&rsquo;s okay, right? Just add a couple more configurations to your `/etc/mock` and off you go.

But there&rsquo;s a problem here. What if you add another branch? Or another project? You have to keep adding configuration files to `/etc/mock` and that doesn&rsquo;t really scale well. True, it doesn&rsquo;t seem like a lot of overhead. But we&rsquo;re building in the cloud, remember? So now every time you want to add another configuration file you have a not-so-fun list of things to slog through:

* Grab a machine image file for your cloud builder (assuming you have one on hand)
* Mount it on loopback
* Copy your configuration file into `/etc/mock`
* Now bundle, upload and register your image remembering to name it something different than your last one (because you do want to make sure that you remember you&rsquo;ve changed your image, dont you?)
* Stop all your running build instances
* Deregister and delete your old build image
* Start up some fresh instances using your newly registered builder image
* Hope that you didn&rsquo;t break anything when you made the change otherwise you have to do the whole process over again

That&rsquo;s a lot of work and also error prone. And it&rsquo;s not a very &ldquo;cloudy&rdquo; way of doing things. It would be a lot better if we could access this configuration information from somewhere else. Sure, Mock doesn&rsquo;t really lend itself to supplying dynamic configurations, but there are ways around that. What it will let you do is change the configuration directory using the `--configdir` cli option. So how about if we pointed it at a temporary configuration directory and then downloaded a remote configuration file and dropped it in there? That&rsquo;s exactly what I decided to do. Now instead of supplying a configuration name to rpmfab we can instead supply the URL of a Mock configuration like this:

```bash
rpmfab/build-arch.py
    -c http://myjenkins:8080/userContent/mock/$platform-$arch.cfg
    -o results
    --mock-options "--uniqueext $BUILD_TAG" *.src.rpm
```

Now there&rsquo;s almost no need to ever touch the builder image. Simply changing the URL in the Jenkins build job to a different Mock configuration is all that needs to be done. Rpmfab will download the remote file (or a local file for that matter), drop it into a temporary directory, generate other necessary files, and point Mock at the configuration when it runs.

Building in the cloud just got a little easier.