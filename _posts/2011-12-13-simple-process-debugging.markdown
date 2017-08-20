---
layout: post
title: Simple Process Debugging
date: '2011-12-13 04:36:00'
tags:
- bash
---

Ever have an elusive bug that only occurs once in a long while, usually over the weekend when you&rsquo;re not in the office? I&rsquo;ve been having this very problem with my current project, where all of the sudden it will stop closing sockets and exiting threads, resulting in a terminated process. The following is a simple shell script you can use to set maximums on open sockets and threads for a process. It will poll the process at your chosen interval. When your process exceeds the values you&rsquo;ve set, it will immediately attach the GDB debugger, which pauses execution on the process. Brilliant! Now when you come back in the morning after your test infrastructure has been beating up on your app, you&rsquo;ll be right at the point where it was starting to take a nose dive.

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "usage: $(basename $0) [process_name]"
    exit 1
fi

process_pid=$(pidof $1)

socket_max=60
thread_max=150
sleep_interval=5

function attach_gdb()
{
    gdb --pid=$process_pid
    echo "quitting..."
    exit 0
}

while [ 1 ]; do
    echo -n "open sockets: "
    socket_now=$(ls -l /proc/$process_pid/fd | grep socket | wc -l)
    echo $socket_now

    echo -n "running threads: "
    thread_now=$(ls -l /proc/$process_pid/task | wc -l)
    echo $thread_now

    if (( $socket_now > $socket_max )); then
        echo "too many sockets!"
        attach_gdb
    elif (( $thread_now > $thread_max )); then
        echo "too many threads!"
        attach_gdb
    fi

    sleep $sleep_interval
done
```

You&rsquo;ll need to run this script as root, so you could also add the code from my previous post. The code above is not perfect. For instance, sometimes not all files in the file descriptor path cannot be accessed, which can result in ugliness on the screen. What I&rsquo;d really like to do is make this a little fancier; maybe add some bells and whistles. It would be helpful, for instance, to add command line options so you can pick what constraints you want in order to the trigger the debugger. Maybe something for a later post? We&rsquo;ll see.