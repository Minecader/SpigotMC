Prerequisites
--

* a `vswap` OpenVZ VPS or a KVM/Xen/VMware VPS with Ubuntu Server 12.04 LTS installed (architecture doesn't matter), or a dedicated server
* patience and a willingness to learn
* approximately 10 minutes of time

Why Ubuntu LTS?
--

You can skip this section, but here's some reasoning as for why I chose Ubuntu LTS for this guide:
* It combines the stability of Debian with support from the larger Ubuntu community
* The "milestone" release cycle helps maintain stability and support
* It has updated repositories and easy ways to install software not in them
* It has enterprise support and is easy for newbies to understand
* It has a sensible package manager (for installing software) and startup daemon
* It's fast, has sensible defaults, and is easy to set up Minecraft on

**Note: This guide should work for Debian 7. However, instead of using the PPA for Oracle Java 7, follow [these instructions](http://www.webupd8.org/2012/06/how-to-install-oracle-java-7-in-debian.html) instead.**

**Note 2: For securing CentOS, follow [this guide](http://lowendbox.com/blog/securing-your-server-ssh-and-sudo/). You need to install `python-pip`, which can be found in EPEL, and `python-dev`, which should be in the standard repos.**

Accessing your server
--

Your host should have provided details for accessing your server via SSH.

* On Windows, you can use the free [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) SSH client to access your server.
* On a Unix-based OS (e.g. Mac OS X), you can use the SSH client in the terminal as such:
  `ssh <server address> -l <username, should be root for clean setups> -p <port>`

Fill in the details on your respective client, and start the SSH session. You might see something about "the authenticity of the server can't be established", that's normal, and just input "y" or "yes". Now, input the password your host gave you (you won't see it in the client, that's convention). You can paste using the **right mouse button** on Putty, and otherwise, it depends on your terminal.

**If your host didn't give you a root account, (OVH sometimes does this), you can skip this and "securing your server".**

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

    usermod -a -G sudo <user name here>


Now, exit your SSH session by running `exit`, and follow "accessing your server", this time using the account you just created instead of root.

Once you've gotten into your new account, we need to test if it has root privileges. Run a `sudo apt-get update`, input your password, and see if it runs. If it runs, you're on the right track, otherwise, go back a few steps and check your work.

Let's change our SSH login settings. First, let's install nano, the text editor (if it isn't already installed):

    sudo apt-get install nano

Run `nano`, and take a look. `Ctrl+X`, then `Y + Enter` closes and saves a file. A blank file isn't saved. Close `nano`, and read on.

Now, let's remove some annoyances of nano:

    sudo nano /etc/nanorc
    
Change `set historylog` to `#set historylog`, and save the file: `Ctrl + X`, `Y`, and `Enter`. Next:

    cd ~
    sudo rm .nano_history

Now, we need to disable the root account, as we don't need it anymore, and it presents a security risk. 
Run `sudo passwd -dl root`, and now, we'll edit the SSHd configuration so that `root` can't log in either:

    sudo nano /etc/ssh/sshd_config

Find the line that says "PermitRootLogin", and change that to `no`. Save the file, and then run:

    sudo service ssh restart

More security-conscious admins may point out that the SSH port has remained the same, and that I should have introduced private key authentication, but those are outside the scope of the tutorial, and should not be necessary.

Now, in case you don't like the spammy Ubuntu message of the day when you log in, you can do:

    cd ~
    touch .hushlogin

For future reference, `~` is your home directory (a.k.a. folder).

Setting up mark2
--

[mark2](http://github.com/mcdevs/mark2/) is a *server wrapper* with advanced monitoring, scripting, and multiuser capabilities that's point-and-click through the SSH session, and is easy to install on Ubuntu. Let's get started.

First of all, we need to install some dependencies:

    sudo apt-get install python-software-properties software-properties-common

Press enter, and now we install the rest of the python dependencies (this allows mark2 installation through pip):

    sudo apt-get install python-dev python-pip

We are done with the dependencies. Now, it's time to install mark2.

    sudo pip install mark2
    
Congratulations! You have now set up mark2. Finally, let's download and install our Minecraft server running [Spigot](http://spigotmc.org/).

Setting up Spigot
--

Obviously, we need to install Java first (we'll be installing Oracle Java 7):

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java7-installer oracle-java7-set-default

Next, let's get back to your home directory, and make a folder called `spigot`:

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
    # This helps stop GC hangs
    java.cli.XX.UseConcMarkSweepGC=true
    
Save the file, and now, let's start the server:

    mark2 start
    
You should see mark2 starting the server, and congratulations, you have set up a server with mark2. Connect to your Minecraft server using your VPS' address, and bask in the glory of having set up a Minecraft server.

Accessing your Minecraft console
--

To access your Minecraft console, run `mark2 attach`. This is the standard mark2 console, and note that you can click on most things, and control the server that way. 

**If you're having "UnicodeEncodeError" after running that, run this, and run `mark2 attach` again:**

    sudo update-locale LANG=en_US.UTF-8 LC_MESSAGES=en_US.UTF-8

Let's run some sample commands:

    version
    
That gives you the Spigot version you're running. Now, run the following **with the tilde** to restart the MC server:

    ~restart
    
Now, exit mark2 by pressing Ctrl+C. You can reattach using `mark2 attach`. You can run **either** `~stop` in the console or `mark2 stop` to stop the MC server. This works for multiple logged on users without any additional configuration, unlike `screen` or `tmux`.

If you have multiple servers, you can click the server names at the top of the mark2 screen to switch to one of them.

**If you're getting special accented characters for mark2, you need to force your SSH client to use UTF-8 encoding.**

For Putty, you can go to *Window > Translation*, and set "received data" to `UTF-8`. Restart Putty to see the changes.

Scripting mark2, and maintaining uptime
--

This will be the final "tutorial" component, and will cover:
* Automatic restarts
* Periodic saves
* Starting mark2 automatically if your server restarts

mark2 has a powerful scripting mechanism. I recommend that you [review it first](https://raw.github.com/mcdevs/mark2/master/samples/scripts.txt) before continuing, so that you gain a better understanding of the `cron` syntax used in mark2 and Linux in general.

Now, let's create the `scripts.txt` file:

    cd ~
    cd spigot
    nano scripts.txt
    
Paste this in `nano`, and adjust as necessary:

         # Saves the map every 15 minutes
         */15 *    *    *    *    ~save
         # Restarts Spigot every 24 hours
         0    12   *    *    *    ~restart
         # Warns 5 minutes before restart
         55   11   *    *    *    /say Server restarting in 5 minutes
         
Save the file, and stop mark2 by running either `mark2 stop` or `~stop` in the console. Start mark2 by running `mark2 start`.

Now, in case you need to restart your VPS, let's set it up so that mark2 starts your server automatically: 

    sudo nano /etc/crontab

Add this to the end, obviously customizing your server location and username:

    @reboot <username> mark2 start /home/<username>/spigot

Save the file, and that should be covered. **If you really need to test it**, you can attempt to restart the server:

    sudo reboot

This covers the general server setup using mark2. Sit back, relax, eat a sandwich, and play some Minecraft.

Graphical access
--

In case you're not feeling like using `wget` and `nano` to download plugins and edit configs, or want to easily transfer your existing server, any SCP/SFTP client will do.

[WinSCP](http://winscp.net/eng/index.php) is a good choice for Windows, [CyberDuck](http://cyberduck.ch/) works quite well on OS X, and [Filezilla](https://filezilla-project.org/) is a nice cross-platform solution.

For these clients, just select SCP or SFTP, and use your SSH credentials. Use their respective documentations for usage.

MySQL Server
--

Some plugins, such as Logblock, require an SQL server. Setting up a server that can be accessed only locally is as easy as:

    sudo apt-get install mysql-server
    
Input your preferred SQL password (preferably different from your server password), and we can create a database.

First, let's login to the MySQL shell:

    mysql -u root -p
    
Type the password you set earlier, and now, inside the shell:

    create database minecraft;
    exit
    
That created a database called `minecraft`, and exits the shell. SQL queries end with `;`.

Now, to use that database in a plugin, fill in the host as `localhost`, username as `root`, password as the one you set during the install, and database as `minecraft`. 

If you want to set up multiple SQL users, or access the database remotely, that is outside the scope of this tutorial.
Please use Google for guides.

Tips and tricks
--

If you're using a Spigot derivative, set `restart-on-crash` to `false` in `bukkit.yml`. **This conflicts with mark2's crash detection**, and doesn't detect possible connection failures, it only checks for server hangs. If your server is **crashing due to a PermGen related issue**, add `java.cli.XX.PermSize=128M` to your `mark2.properties`.

If you're migrating a server, and you can't connect after the migration, check that `server-ip` in `server.properties` is either blank or the IP you want it to bind to, and check your Votifier settings (this can cause connection failures).

If you really need a specific version of a JAR, or just want to download files directly, `cd` into the directory you want to download to, and then:

    wget <file location> -O <output name>
    
You might need to `sudo apt-get install wget`, depending on your Ubuntu image.

For monitoring resource usage, you can run `top`, and for monitoring memory usage, you can run `free -m`.

Update your system using `sudo apt-get update && sudo apt-get dist-upgrade`. Update mark2 using `sudo pip install --upgrade mark2`.

Have fun!

Additional resources
--

You should review these sometime to learn more about Ubuntu, mark2, and Linux in general (assuming you already know how to administer Minecraft servers)

* Some useful [Linux commands](https://help.ubuntu.com/community/UsingTheTerminal#Commands)
* mark2 [configuration samples](https://github.com/mcdevs/mark2/tree/master/samples)
* mark2's [readme](https://github.com/mcdevs/mark2/blob/master/README.md) and [usage](https://github.com/mcdevs/mark2/blob/master/USAGE.md)
* Why Linux isn't actually [eating your RAM](http://www.linuxatemyram.com/)
* Intro to the Linux [file system](http://www.linux.org/article/view/getting-around-in-linux-directories)

### Some entertainment

    <LaxWasHere> You could have just made a bash script. "Minecraft in 2 seconds."
    <chiisana> 2 seconds* 
    <chiisana> *: actual time may vary depending on internet connection speed, and server installation time
    <vemacs> Coming soon?!