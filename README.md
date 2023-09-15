# Step-by-Step-Guide-on-How-to-Setup-Server-with-Ubuntu-22.04

Once a fresh Ubuntu 22.04 server has been created, there are several configuration steps you can take to enhance security and simplify management. Remember that security is a continuous process, so it's critical to keep your server configurations and software updated as well as to stay current with the most recent security best practices. 

To maintain continuing protection, monitor security logs frequently and carry out security audits. This manual will walk you through a few steps that you must carry out before installing and configuring any software or services on your new server in order to lay a strong foundation for it.

## Step 1: Authenticate as Root

You'll connect to your server for the first time using the root account, which is often the sole account configured on freshly installed servers.

An administrative user with incredibly broad permissions is the root user. You are discouraged from using the root account frequently due to its elevated privileges. This is due to the root account's intrinsic power, which includes the capability to unintentionally make changes that are incredibly harmful. 

Due to this, it is advised to create a regular system user and provide it **sudo** capabilities so that it can execute administrative commands subject to some restrictions. You will create such a user in the following step.

You must log in to your server before you can proceed. Be certain you understand the public IP address of your server. You shall need the root user's SSH private key if you have configured an SSH key for server-side authentication or your account's password to authenticate. 

Enter the following command to log in as the root user if you are not already connected to your server. Use your server's public IP address in the command's highlighted section.

```
$ ssh root@server_ip_address
```

If a message concerning the **host's** legitimacy is displayed, accept it. The display can look like this;

```
The authenticity of host '112.174.192.136 (172.124.152.186)' can't be established.
ECDSA key fingerprint is SHA256:5EVfdhssgmVKOzYqhOCvWWv/dghkgfdfhjL138q83iSXnc.
Are you sure you want to continue connecting (yes/no)?

```


Enter your root password to log in if password authentication is being used. As an alternative, if you are using a passphrase-protected SSH key, you can be asked for it the first time you use it during each session. You could also be asked to change the root password if this is your first time accessing the server with a password.

## Step 2: Create a New User
The next step is to create a new system user account with restricted access and set it up to utilize sudo to execute administrative commands. Instead of using the root account for regular operations, creating a new user account becomes a great idea.
Create a new user by running the following command.

```
# adduser james
```

In the example above, a new user named james is created, but you should change it to a username of your choice. You'll be prompted to set a password and provide additional information for the new user.

Go ahead and set up a strong password and fill in additional information about the new user if you want. Or you can just press **ENTER** in any field you want to skip.

You will provide this user sudo access in the coming step. As a result, the user will be able to use the **sudo** software to carry out administrative operations as the root user.

## Step 3: Grant Administrative Privileges

You now possess a brand-new user account with standard rights. By default, new users don't have administrative privileges. The need to manage servers, modify configuration files, or restart servers are examples of administrative duties that may occasionally be required.

You can grant your normal account **root** capabilities in order to avoid having to log out and log back in as the root account. By prefixing each command with sudo, you can provide your ordinary user the ability to conduct operations with administrator rights.

You must join the sudo group with the new user if you want to grant them these rights. Users who are a part of the sudo group can use the sudo command by default on Ubuntu 22.04.

Run this command as root to add your new user to the sudo group (substitute your new user for the word highlighted;

```
# usermod -aG sudo james

```
Congratulations! You have now created your system user.

Point to Note: We advise staying logged in as root until you make sure you can log in and use sudo as your new user. In this manner, if issues arise, you may analyze the situation and make any necessary adjustments at the root.

## Step 4: Configure Firewall

Enable a firewall and configure it to allow only necessary incoming and outgoing network traffic. Ubuntu 22.04 uses ufw (Uncomplicated Firewall) to manage the firewall rules. 

In order to log back in next time, you must ensure that the firewall permits SSH connections. You can approve these connections using the following:

```
# ufw allow OpenSSH
```

Then, go ahead and enable the firewall using the following command:

```
# ufw enable
```

To continue, enter **"y"** and click **Enter**.

To confirm if SSH connections are still permitted by checking the status using the following:

```
# ufw status
```

Results

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)

```


## Step 5: Provide Your Regular User with External Access

You have to ensure you can SSH directly into the account now that you have a regular user for everyday use. 

Whether your server's root account uses a password or SSH keys for authentication determines how to set up SSH access for your new user.

### *Where Password Authentication Is Used for the Root Account*
If you use a password to access your root account, SSH's password authentication feature is enabled. By starting a fresh terminal session and entering your new username in SSH, you can SSH to your new user account. Open up a new terminal and key in this command:

```
$ ssh james@server_ip_address
```

By doing this, you will be logged in after providing the password for your regular user.


**Point to Note:** *Always type sudo before a command if you need to run it with administrator rights. For example*

```
$ sudo command_to_run
```

When using sudo for the first time during each session (and occasionally thereafter), you will be prompted for your regular user password.


We strongly advise utilizing SSH keys instead of password authentication to improve server security. Learn how to set up key-based authentication by referring to our instructions for [configuring SSH keys on Linux](https://github.com/SerahNderitu/How-to-Generate-a-New-SSH-Key-on-Linux-and-Add-it-to-the-SSH-Agent).


### *Where Key Authentication Is Used by the Root Account*
Password authentication for SSH is probably removed if you log in to your root account using SSH keys. To properly log in, you must add a copy of your local public key to the new user's ``~/.ssh/authorized_keys`` file.

Since the root account's  ``~/.ssh/authorized_keys`` file already contains your public key, you can replicate that file and directory structure to your new user account while still in the same session.

The ``rsync`` command is the easiest technique to copy the files with the correct ownership and permissions. In a single command, this will copy the root user's ``.ssh`` directory, preserve the permissions, and change the file owners.


**Point to Note:** *Sources and destinations with a trailing slash are handled differently by the rsync command than sources and destinations without one. Make sure the source directory ``(~/.ssh)`` does not have a trailing slash when using the ``rsync`` command below. Verify that you are not using ~/.ssh/.
Therefore, use this ``( ~/.ssh )`` and **NOT** this ``( ~/.ssh/ )``.*

The contents of the root account's ``~/.ssh`` directory will be copied to the home directory of the **sudo** user if a trailing slash is unintentionally added to the command. This prevents the full ~/.ssh directory structure from being copied. SSH won't be able to locate and use the files since they are in the incorrect location.

Replace the highlighted section to match your regular username below

```
$ rsync --archive --chown=james:james ~/.ssh /home/james
```

Now, start a fresh terminal window and try to log in with your new username. You can do that using the following;

```
$ ssh james@server_ip_address

```

The remote user's SSH password shouldn't be required for authentication when you log into the new user account. When you use your SSH key for the first time in a terminal session, if it was configured with a keyphrase, you might be prompted for that password in order to unlock it.

### Step 6: Disable SSH Root Login
To disable SSH root login, you can follow these steps. First, log in to your server as the root user using SSH. Then, open the SSH configuration file. The location of the file is commonly found at ``/etc/ssh/sshd_config``.

Enter this command:

```
$ sudo gedit /etc/ssh/sshd_config
```
Once the file is open, locate the line that says ``PermitRootLogin`` and change its value to no. Save the file and exit the text editor. 

Finally, restart the SSH service with ``sudo service SSH restart`` to apply the changes. Now, the root user will no longer be able to log in directly via SSH, enhancing the security of your system.
                                                                                                     
Congratulations! Now you have a strong base for your server. At this point, you can now install any necessary software on your server.
