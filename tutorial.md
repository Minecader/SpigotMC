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

Fill in the details on your respective client, and start the SSH session. Now, input the password your host gave you (you won't see it in the client, that's convention). You can paste using the **right mouse button** on Putty, and otherwise, it depends on your terminal.

Once you're inside, you should see the Ubuntu message of the day, and a line on the bottom saying:
    root@<machine hostname>:~$ 
    
That is the shell you will be using, and let's get this party started.

