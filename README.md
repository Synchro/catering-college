Install chef-server from scratch
================================

Installing your own Chef server can be a bit hoop-jumpy. This abstracts all of those hoops into one (ugly) script which does all the jumping for you.

What does it do?
----------------

Chef server depends on a truckload of not-necessarily-easy-to-install things. We mostly follow [this](http://wiki.opscode.com/display/chef/Installing+Chef+Server+Manually) with a detour into RVM and Ruby 1.9.2.

The script:

+ Installs CouchDB
+ Installs and configures RabbitMQ (a more recent version than is in the Ubuntu repos)
+ Installs Oracle Java 7
+ Installs OpsCode's Gecode
+ Installs RVM
+ Installs Ruby 1.9.2 (which isn't yet in the Ubuntu repos)
+ Installs the Chef gems
+ Configures the Chef server
+ Creates upstart scripts for each of the Chef components
+ Installs nginx to proxy the WebUI

Regarding Java, the delightful people at Oracle, always the friends of free software, have moved the goalposts and the oab-java6 thing no longer works. I know people have come up with workarounds for this, but no doubt these will get stepped on soon, too. Life is too short to spend it trying to outwit lawyer-driven corporations, so it now tests if Java is already installed, and if not, installs OpenJDK (which seems to work OK for me).

These steps are mostly idempotent.

How to use it
-------------

Dead-simple install:

    sudo apt-get install -y curl ; bash <(curl -s https://raw.github.com/gist/1869395/179c776758d8aae317b63fa87c447b821fb420a4/catering-college-installer)

What the above is actually doing: we create a new user (I've used 'chef') and add them to the 'sudo' group (the shortest path to sudo happiness, at least on Ubuntu Precise) and give them passwordless sudo privileges temporarily. Then, as this new user, checkout and run the script.

    #!/bin/bash
    NEWUSER=chef
    sudo useradd -m ${NEWUSER} -s /bin/bash -G sudo
    U=`umask`
    umask 0226
    sudo echo "${NEWUSER} ALL=(ALL) NOPASSWD: ALL" |sudo tee /etc/sudoers.d/${NEWUSER} > /dev/null
    umask $U
    sudo apt-get update -q
    sudo apt-get install -q -y git-core
    sudo su - ${NEWUSER} -c "git clone https://github.com/pikesley/catering-college ; cd catering-college ; ./install-chef-server"
    sudo rm  /etc/sudoers.d/${NEWUSER}

Caveats
-------

+ I've tested this to death on an Ubuntu Precise VM (Vagrant's stock precise64 box). I suspect it'll probably work on other Debian-ish platforms.
+ The configuration step for chef-solr assumes there's no data in there that we care about (which is obviously true for a new installation), and nukes the whole thing. There may be ways around this, but I don't know how Solr works and I really have no desire to.

