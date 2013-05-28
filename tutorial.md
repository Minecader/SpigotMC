Prerequisites
--

* a `vswap` OpenVZ VPS or a KVM/Xen/VMware VPS with Ubuntu Server 12.04 LTS installed (architecture doesn't matter)
* patience and a willingness to learn

Accessing your server
--

Your host should have provided details for accessing your server via SSH.

* On Windows, you can use the free [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) SSH client to access your server.
* On a Unix-based OS (e.g. Mac OS X), you can use the SSH client in the terminal as such:
  `ssh <server address> -l <username, should be root for clean setups> -p <port>`

Fill in the details on your respective client, and start the SSH session. You might see something about "the authenticity of the server can't be established", that's normal, and just input "y" or "yes". Now, input the password your host gave you (you won't see it in the client, that's convention). You can paste using the **right mouse button** on Putty, and otherwise, it depends on your terminal.

Once you're inside, you should see the Ubuntu message of the day, and a line on the bottom saying:

    root@<machine hostname>:~$ 
    
That is the shell you will be using, and let's get this party started.

Securing your server
--

The `root` account is the superuser of the system, and as such, you should not run non-administrative tasks using this account.

The first things we will be doing are:
* updating your server to the latest packages
* setting up a separate user account on your system for running your server
* disabling the root account, and enabling escalation via `sudo`

To update your server, run the following two commands (press enter at each newline):

    apt-get update
    apt-get dist-upgrade

See, that wasn't too difficult. Next, we'll add an administrative user:

    adduser <user name here>
    
For example, `adduser minecraft`. After you've set and verified the password, you don't need to put any more information. Just keep on hitting enter.

Next, we'll be adding the new user to the administrative users list:

    apt-get install nano
    visudo

`nano` is the easiest text editor for newbies, and so if `visudo` asks you for a text editor, use that.

Find the line near the center of the page that says `# User privilege specification`, and add

    <user name here>    ALL=(ALL:ALL) ALL
    
underneath `root    ALL=(ALL:ALL) ALL`. Press Ctrl+X and input Y and enter to save.

Now, exit your SSH session by running `exit`, and follow "accessing your server", this time using the account you just created instead of root.

Once you've gotten into your new account, we need to test if it has root privileges. Run a `sudo apt-get update`, input your password, and see if it runs. If it runs, you're on the right track, otherwise, go back a few steps and check your work.

Now, let's remove some annoyances of nano:

    sudo nano /etc/nanorc
    
Change `set historylog` to `#set historylog`, and save the file. Next:

    cd ~
    sudo rm .nano_history

Now, we need to disable the root account, as we don't need it anymore, and it presents a security risk. 
Run `sudo passwd -l root`, and now, we'll edit the SSHd configuration so that `root` can't log in either:

    sudo nano /etc/ssh/sshd_config

Find the line that says "PermitRootLogin", and change that to no. Save the file like you did in `visudo`, and then run:

    sudo service ssh restart

More security-conscious admins may point out that the SSH port has remained the same, and that I have introduced private key authentication, but those are outside the scope of the tutorial, and should not be necessary.

Now, in case you don't like the spammy Ubuntu message of the day when you log in, you can do:

    cd ~
    touch .hushlogin

For future reference, `~` is your home directory (a.k.a. folder).

Setting up mark2
--

[mark2](http://github.com/mcdevs/mark2/) is a *server wrapper* with advanced monitoring, scripting, and multiuser capabilities that's point-and-click through the SSH session, and is easy to install on Ubuntu. Let's get started.

First of all, we need to install some dependencies:

    sudo apt-get install python-software-properties software-properties-common

Next, we'll add a repository for access to the latest and greatest version of `twisted`, which mark2 relies on:
    
    sudo add-apt-repository ppa:twisted-dev/ppa
    sudo apt-get update
    
Press enter, and now we install the rest of the python dependencies:

    sudo apt-get install python-dev python-pip python-twisted-core python-twisted-web python-twisted-words
    sudo pip install psutil urwid feedparser
    
We are done with the dependencies. Now, it's time to:

* Download the mark2 archive and unpack it
* Link mark2 to an executable so you can easily run it

This is pretty straightforward, and don't worry if you don't understand the commands:

    cd /usr
    sudo wget https://github.com/mcdevs/mark2/archive/master.tar.gz
    sudo tar zxvf master.tar.gz
    sudo rm master.tar.gz
    sudo mv mark2-master mark2
    sudo ln -s /usr/mark2/mark2 /usr/bin/mark2
    
Congratulations! You have now set up mark2. Finally, let's download and install our Minecraft server running [Spigot](http://spigotmc.org/).

Setting up Spigot
--

First, let's get back to your home directory, and make a folder called `spigot`:

    cd ~
    mkdir spigot
    cd spigot

Now, let's get the latest Spigot JAR, using mark2:

    mark2 jar-get spigot-latest
    
Wait a few seconds, and the Spigot JAR should be downloaded.

Now, let's set up a blank mark2 configuration, so we can start the server:

    sudo mark2 config
    
Save the file without making any changes, as it's easier to use our own `mark2.properties` instead of trawling thru the config.

We need to tell mark2 what to do, so:

    nano mark2.properties
    
Paste this into `nano`:

    java.cli.X.ms=1024M
    java.cli.X.mx=1024M
    # Use whatever mark2 saved the JAR as here
    mark2.jar-path=spigot-1.5.2-R0.2-SNAPSHOT.jar
    # Saving notifications aren't really useful
    plugin.save.warn-message=
    plugin.save.message=
    
Save the file, and now, let's start the server:

    mark2 start
    
You should see mark2 starting the server, and congratulations, you have set up a server with mark2. Connect to your Minecraft server using your VPS' address, and bask in the glory of having set up a Minecraft server.

Accessing your Minecraft console
--

To access your Minecraft console, run `mark2 attach`. This is the standard mark2 console, and note that you can click on most things, and control the server that way. Let's run some sample commands:

    version
    
That gives you the Spigot version you're running. Now, run the following **with the tilde** to restart the MC server:

    ~restart
    
Now, exit mark2 by pressing Ctrl+C. You can reattach using `mark2 attach`. You can run **either** `~stop` in the console or `mark2 stop` to stop the MC server.