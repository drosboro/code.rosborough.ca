+++
date = "2014-03-19T11:28:36-07:00"
title = "DNS server monitoring with dnstop"
+++

I was recently decommissioning a DNS server (running PowerDNS), and was temporarily "stumped" trying to figure out if I was still seeing traffic to the server.  With a web server, I simply `tail -f /var/log/_my_webserver_access_.log` and watch to see who's still accessing the server.  However, I had minimal logging set up for PowerDNS, and I didn't really feel like wading through config files to set up a bunch of logging for a server I was about to kill anyways.

Happily, I found `dnstop`.  (That's "DNS-top", not "DN-stop", by the way!)  On a Debian/Ubuntu server, it's installed simply by:

    sudo apt-get install dnstop

On CentOS or other yum-my systems:

    sudo yum install dnstop

You'll need to figure out which network interface to listen to.  Simply run `ifconfig` or `sudo ifconfig` and look for the name of the interface that has your public IP address listed.  On my VPSs, it's usually `eth0` or `venet0:0`.

Once you know the name of the relevant interface, run dnstop as root:

    sudo dnstop venet0:0

While you sit there and watch, dnstop will start gathering any DNS requests made via the specified interface.  This will include incoming and outgoing requests.  The default view shows the source IP address of the request, so it's easy to see which ones are outgoing (ie, the source is the server's IP address) and incoming (all the others).

You can also change the view.  Typing `?` will bring up the list of available views.  I've reproduced some of the more useful ones here:

    s - Sources list
    d - Destinations list
    t - Query types
    o - Opcodes
    r - Return codes
    1 - 1st level Query Names (TLDs)
    2 - 2nd level Query Names
    ^R - Reset counters
    ^X - Exit

    ? - help

This turns out to be a super-simple and useful way to tell what DNS traffic is coming into and out of your machine.