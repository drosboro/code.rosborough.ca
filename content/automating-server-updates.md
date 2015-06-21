+++
date = "2015-04-12T21:32:18-07:00"
title = "Automating server updates"
+++

As the number of virtual servers I've run has increased, it has become an increasing nuisance to log into each and run a `sudo aptitude full-upgrade` on each of them.  I use [apticron] to email me every time an update is available for each of my servers, and 99% of the time I am good to just `full-upgrade` them.

I use my `~/.ssh/config` file to keep aliases to all of my servers, including port numbers, user names, etc., so it is usually simply a matter of typing `ssh server1` to connect to a server.  This means that it's dead-simple to write a shell script to log into a list of servers, one at a time, and run `sudo aptitude full-upgrade` on them.

Here's my script - complete with nice colored headers for each server:

    #!/bin/bash
    
    servers=(server1 server2 server3 server4)
    
    textreset=$(tput sgr0)
    green=$(tput setaf 2)
    bold=$(tput bold)
    
    for s in ${servers[@]}; do
        echo -e "\n${green}${bold}Updating  $s\n=========================\n${textreset}"
        ssh $s "sudo aptitude full-upgrade"
    done
    exit 0

I simply save the script somewhere in my path (for me, I saved it to `~/bin/up`, because I've added that to my user's path), and `chmod +x ~/bin/up` to make it executable.

Now, I just type `up` from the command line, and all my servers update automatically.

**NOTE**: The clever among you will point out that this will still require some user intervention (hitting 'Y' if there's packages to update).  That's the way I wanted it.  If you want it to "assume yes", simply add it to the command:

    ssh $s "sudo aptitude -y full-upgrade"

[apticron]:(https://www.debian-administration.org/article/491/Automatic_package_update_nagging_with_apticron)