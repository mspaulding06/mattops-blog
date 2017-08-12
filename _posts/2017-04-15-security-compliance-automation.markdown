---
layout: post
title: Security Compliance Automation
date: '2017-04-15 16:31:58'
tags:
- configuration-management
- security
---

Recently I have been working on project to automate security compliance of Linux systems.  There are many options to solve this common problem but I haven't been satisfied with them.  I'll outline the options I considered and then explain the option that I ended up choosing.

### The Options

###### Golden Base Image
One way to handle this is to have security compliance built in to your base image.  That way every time you deploy you know that your system is secure.  Or at least that's the idea.  But what about drift over time?  What if the compliance rules change?  There is no way to manage this except for building a new golden base image and then rebuilding your systems.  If you're working with something like Docker that may not be a problem, and is actually the way that things ought to be done since containers are considered immutable infrastructure.  But if you're working with virtual machines or bare metal (pets, not cattle) then you have yourself a problem.  While the base image may solve part of the problem you still need to run compliance checks and automate remediation.  Not an ideal solution.

###### Custom Scripts
Another possible solution is to create a set of custom scripts that configure your systems for compliance.  As long as you have a scanner, something like [Nexpose](https://www.rapid7.com/products/nexpose/), that might be okay.  The process would be that you check for compliance with your scanner, update your custom scripts to bring your systems into compliance if needed, and then run your custom scripts.  This doesn't scale well and requires to run your scripts either manually or via a cron job.  Configuration management tools do a lot of this for you already so that might be another option to try.

###### Puppet
An obvious choice considering this is the primary configuration management tool that I use is to build compliance into Puppet.  There are a couple different ways to go about this.  Either I can create a new Puppet module that manages these secure configurations, or I can build security into each of the modules that I am already using.  Creating a new Puppet module seemed to be a good idea so I tried that first.  The hope was that I could keep compliance rules isolated from the rest of the Puppet code.  It turned out this wasn't possible.  I'll give an example.

For compliance I have been using the [CIS Benchmarks](https://benchmarks.cisecurity.org/downloads/benchmarks/).  One example of a compliance rule from the benchmark is management of permissions for cron directories and files.  In my security module I would like to be able to write code to manage this similar to the following:

```puppet
file {
  default:
    mode => '0600',
  ;

  '/etc/cron.d':
    ensure       => directory,
    recurse      => true,
    recurselimit => 1,
  ;
}
```

This works great on its own, but what about the cron jobs that are already managed elsewhere in Puppet?  Let's say that I am using a [cron module](https://forge.puppet.com/torrancew/cron) to manage cron jobs for a specific application.

```puppet
include cron

cron::hourly { 'mycronjob':
  minute  => 30,
  command => 'echo "hello world"', 
}
```

So what is the cron module actually doing here?  It's going to create a file resource that manages a file inside the `/etc/cron.d` directory.  Now this is a problem because in Puppet resources can only be managed once.  We'll get an error when we run our Puppet agent on the node.

Then why don't we try checking if the resource already exists?  Great idea.  Our compliance module will check if files are managed before attempting to manage them.  Let's assume that we've written a custom function called `get_files` that will return all the files on disk in a given directory.  Then we can tell Puppet to manage each of these but only if they're not already managed.

```puppet
$files = get_files('/etc/cron.d')

$files.each |String $file| {
  if !defined(File[$file]) {
    file { $file:
      ensure => file,
      mode   => '0600',
    }
  }
}
```

So what about files that are already managed?  They won't get our security permissions set now.  Well, we can do something about that too.  In the case that a file is already managed we can modify the attributes using a collector.

```puppet
$files = get_files('/etc/cron.d')

$files.each |String $file| {
  if !defined(File[$file]) {
    file { $file:
      ensure => file,
      mode   => '0600',
    }
  } else {
    File<| title == $file |> {
      mode => '0600',
    }
  }
}
```

Okay, we did it.  But now wait, what if the file resource is created with an alternate name?  Actually the cron module doesn't use the file name for the `title` of the file resource and so we can't use the collector here.  We could use `path` instead of `title` in the collector, but how would we know which to use in which cases?

There's actually a second problem with the code above.  There is no way to guarantee that this code will get evaluated before the cron module's code.  That's a problem because if this code gets evaluated first then there will be no existing file resource.  If there is no existing file resource then this code will create the file resource.  Since the cron module doesn't check if the file resource is already defined it will fail when it is evaluated second.

###### Puppet without a dedicated module
Alright, so that didn't work.  The next step would be to update our code to be in compliance without creating a separate module.  I'll just go update all the places where we create cron jobs and change the file permissions in each of those locations.  This is certainly possible but tedious.  With the cron module in particular this is possible, but not all modules allow management of the permissions of files that it creates so we need to have another way of doing this.

###### Ansible
How about Ansible?  Maybe I can create compliance playbooks in Ansible to manage security rules separate from Puppet.  That way there will be no resource conflicts.  Sounds great but the problem that occurs here is that Ansible doesn't have any knowledge about what Puppet manages.  So now Ansible can change permissions and settings on Puppet managed files that when Puppet runs again it will change back to the previous insecure settings.  Another dead end.

###### Custom Compliance Automation
Now we come to the final and chosen solution: custom compliance automation.  In the end I decided it best to write a custom tool that can check compliance of settings on a system and do the configuration.  Why is this better?  The problem I have had working with Puppet is that it is not possible to know what resources are already managed at runtime to know how to apply security settings.  It turns out that Puppet managed agents have the ability to download their complied catalog from a Puppet master by running a command.

`puppet catalog download --terminus json`

Using that knowledge I designed a tool that would run periodically on a system and download the Puppet catalog.  It then parses the catalog and gathers a list of all Puppet managed resources.  It has a set of compliance rules that it runs based on the CIS benchmark written in YAML.  The tool then runs compliance checks and remediations based on these rules but uses the Puppet catalog to avoid modifying any resources already managed by Puppet.  Instead the engineer can be alerted through logs or email if a file cannot be brought into compliance because it is already managed.  Then the engineer can make a decision if the Puppet code should be updated to meet the compliance rules.  I decided on this approach because it ensures compliance and provides auditing capability that allows for further improvement of systems compliance.