Put this stuff after

## Server
edit webserver jail get mac address fix router

# New Jail
* **name:** webserver
* **type:** portjail (x64)
Click 'save'


1. Click 'jails'
1. Click 'webserver'
1. Click 'shell'

## SSH access

1. `vi /etc/rc.conf`
2. change `sshd_enable="NO"` to `sshd_enable="YES"`
3. start ssh daemon (sshd) `/etc/rc.d/sshd start`

parameters: needs to belong to group "wheel" - reason? to enable SSH
also make sure when choosing a shell input "bash" - otherwise csh will default, and this won't work

### adduser (your own account)
1. type: `adduser`
2. Invite to login group 'wheel'
3. Set shell path to bash (We'll install this in the next steps) `/usr/local/bin/bash`

### promote www to real user
1. `mkdir /home/www`
2. `chown -R www:www /home/www`
3. `chpass www`
4. set home to `/home/www`
5. set shell to `/usr/local/bin/bash`

## Ports
In the shell that you launched for the web server jail you created we need to fetch extract and update our ports.

1. portsnap fetch
1. portsnap extract
1. portsnap update

### Port: bash
This will prompt for configuration options. Configure as you see fit. I recommend (in the subsequent dependent port installations, disabling EXAMPLES *pic here*

1. `cd /usr/ports/shells/bash`
2. `make install clean`
3. I turn off P4 and CVS

### Port: sudo

1. `cd /usr/ports/security/sudo`
2. `make install clean`
3. I like to select `INSULTS` and `LDAP` as options
4. `vi /usr/local/etc/sudoers`
5. uncomment (remove '#') this line `# %wheel ALL=(ALL) ALL` (near the bottom of the file)
6. `wq!` (Write, Quit, Because I Said So)


NOTE: AT THIS POINT YOU CAN SSH INTO YOUR BOX IF YOU'D LIKE TO; NO MORE FreeNAS WEBGUI SHELL
If you do, and you log in with the user you created earlier, you'll likely need to install ports using the `sudo` command. Oh, how convenient; we just installed that.

### Port: git

1. `cd /usr/ports/devel/git`
2. `make install clean`
3. When it asks about `curl`, I install with LDAP/S and LIBSSH2
4. When `ca_root_nss-3.16.1` select ETCSYMLINK
5. DOCS not needed for `xmlto`, `getopt`, `libgcrypt`, `libgpg-error`, `docbook-xsl`, `xmlcatmgr`, `w3m`, `boehm-gc`, `libatomic_ops`
6. EXAMPLES not needed for `python27`, `p5*`

(this will take a while, but stick with it)


### Port: MySQL

1. `cd /usr/ports/databases/mysql56-server`
2. `make install clean`


### Port: rbenv (https://github.com/sstephenson/rbenv)

1. `cd /usr/ports/devel/rbenv`
2. `make install clean`
3. echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
4. echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

### Port: ruby-build (https://github.com/sstephenson/ruby-build)

1. `cd /usr/ports/devel/ruby-build`
2. `make install clean`
3. You can leave RBENV checked; we already installed it.
4. Execute the commands we placed into our `.bash_profile`, `source ~/.bash_profile` (or `. ~/.bash_profile`)

# Web Server
Next we're going to install the things we need to get our server hooked up and running. We'll start with Ruby.

## Setup
First however, we need to do all of this inside the `www` user, as it will be running nginx etc.

1. `sudo su - www`
2. echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
3. echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
4. `. ~/.bash_profile`

## Ruby

1. `rbenv install 2.1.2`
2. `rbenv global 2.1.2`
3. `rbenv rehash`

(test to see if Ruby runs - "ruby -v" - if not, run "exec $SHELL -l")

## Rails
Let's get Bundler and Rails installed, as we'll need those to get a basic project up and running.

1. `gem install bundler rails`


