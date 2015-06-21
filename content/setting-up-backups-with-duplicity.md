+++
date = "2014-03-19T23:05:25-07:00"
title = "Setting up backups with duplicity"

+++

There are a myriad of ways to get backups of your Linux box.  There's [tar](https://www.gnu.org/software/tar/), there's [rsync](http://rsync.samba.org), there's even good old [dd](http://en.wikipedia.org/wiki/Dd_(Unix)) if you like that sort of thing.  I personally like [duplicity](http://duplicity.nongnu.org), for a couple of reasons:
<!--more-->

* It does a great job of keeping track of multiple "chains" of full/incremental backups
* It uses rsync "under the hood", so it's rock-solid and efficient.
* Although I tend not to use it, you can encrypt your backups with [GnuPG](http://www.gnupg.org).

This is one of the few packages that I generally build from source on my Debian servers, as I prefer to use a later version than can be found in the Debian repositories.  This post describes how I go about setting up backups for my servers.

Installing Duplicity
---------------

First, install the prerequisites:

    sudo apt-get install python-dev librsync-dev python-paramiko python-lockfile
    
This works for me, as I'm tunneling rsync over SSH to another server.  If you were planning to use Amazon S3, you will need to install `python-boto`.

Then, download the source for duplicity.  At the time of writing, 0.6.23 is the latest stable version:

    mkdir ~/src
    cd ~/src
    wget http://code.launchpad.net/duplicity/0.6-series/0.6.23/+download/duplicity-0.6.23.tar.gz

Building and installing duplicity is as simple as:

    sudo python setup.py install

Connecting to the Backup Server via SSH
----------------------

On your main server (where you just installed duplicity), you'll create an SSH key to use for connecting to the backup server:

    sudo -s
    ssh-keygen -t rsa
    
Leave the password blank, so that you can make the connection unattended (say, in a cron job).

Copy your public key:

    cat ~/.ssh/id_rsa.pub

Log into your backup server, and set up the key - in this example, I'll use the user "bob" as the backup user.  If you don't already have key-based authentication set up on the backup server, you'll do the following:

    ssh bob@backup.server.com
    mkdir ~/.ssh
    cat > ~/.ssh/authorized_keys

... and paste in the public key you copied from the other server.  Type `ctrl-D` on a blank line to exit `cat`.  If you've already got an `authorized_keys` file, you will instead want to:

    chmod +w ~/.ssh/authorized_keys
    cat >> ~/.ssh/authorized_keys
    
... so as not to overwrite any existing keys.  I generally `chmod 400 ~/.ssh/authorized_keys` to keep everything nice and "tight".

To test your connection, log into your main server, and try the following:

    sudo -s
    ssh bob@backup.server.com
    
You should be able to connect without a password.

Configuring Duplicity backups
---------------------

Last but not least, you'll need to decide on a backup policy.  For my purposes, I settled on:

* weekly full backups
* daily incremental backups
* retain backups for 1 month

Duplicity makes it very straightforward to create a policy like this.  I create a simple bash script like so:
`sudo vim /etc/cron.daily/file-system-backup`

    test -x $(which duplicity) || exit 0

    $(which duplicity) --no-encryption --full-if-older-than 1W --exclude /dev --exclude **/lost+found --exclude /media --exclude /mnt --exclude /proc --exclude /sys --exclude /tmp --exclude /root/.cache --exclude /var/lib/mysql --exclude /var/lib/postgresql --exclude /var/cache / scp://bob@backup.server.com//home/bob/$(hostname)

    $(which duplicity) --no-encryption remove-older-than 1M --force scp://bob@backup.server.com//home/bob/$(hostname)

The first line checks to ensure that it can find duplicity (since I often copy and paste this script to new servers, it's worth checking!).

The second line does all the work.  `--no-encryption` turns off GnuPG.  `--full-if-older-than 1W` does what it sounds like.  Then I `--exclude` a whole bunch of directories, like `/dev`, `/mnt`, `/proc` and so on.  

Note that I also exclude database directories for [mysql](https://mariadb.org) and [postgresql](http://www.postgresql.org).  Generally speaking, databases don't backup well via the filesystem unless they're shut down first.  Since duplicity is going to take a while, I'm not willing to leave the database servers down while I back up the rest of the filesystem.  The good news - there's great ways to back up these databases, including [xtrabackup](http://www.percona.com/doc/percona-xtrabackup/2.1/) for mysql, and [barman](http://www.pgbarman.org) for postgresql.  Those are topics for another article.

The final line cleans up any backups older than 1 month.

The backups will be found on the backup server, at `/home/bob/main.server.com`, if `main.server.com` were your server's hostname.

That wasn't too bad, now, was it?

