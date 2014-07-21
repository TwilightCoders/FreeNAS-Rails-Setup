# 1. Jail
1. **name:** webserver
2. **type:** portjail (x64)
3. Click 'save'
4. Click 'jails'
5. Click 'webserver'
6. Click 'shell'

## 1.1. SSH access

1. `vi /etc/rc.conf`
2. change `sshd_enable="NO"` to `sshd_enable="YES"`
3. start ssh daemon (sshd) `/etc/rc.d/sshd start`

parameters: needs to belong to group "wheel" - reason? to enable SSH
also make sure when choosing a shell input "bash" - otherwise csh will default, and this won't work

### 1.1.1. adduser (your own account)
1. type: `adduser`
2. Invite to login group 'wheel'
3. Set shell path to bash (We'll install this in the next steps) `/usr/local/bin/bash`

### 1.1.2. promote www to real user
1. `mkdir -p /home/www/sites`
2. `cp ~/.profile /home/www` (Copy our profile over to www, because it never had one to begin with)
3. `chown -R www:www /home/www`
4. `chpass www`
5. set home to `/home/www`
6. set shell to `/usr/local/bin/bash`

## 1.2. Ports
In the shell that you launched for the web server jail you created we need to fetch extract and update our ports.

1. portsnap fetch
1. portsnap extract
1. portsnap update

### 1.2.1. Port: bash
This will prompt for configuration options. Configure as you see fit. I recommend (in the subsequent dependent port installations, disabling EXAMPLES *pic here*

1. `cd /usr/ports/shells/bash`
2. `make install clean`
3. I turn off P4 and CVS

### 1.2.2. Port: sudo

1. `cd /usr/ports/security/sudo`
2. `make install clean`
3. I like to select `INSULTS` and `LDAP` as options
4. `vi /usr/local/etc/sudoers`
5. uncomment (remove '#') this line `# %wheel ALL=(ALL) ALL` (near the bottom of the file)
6. `wq!` (Write, Quit, Because I Said So)


NOTE: AT THIS POINT YOU CAN SSH INTO YOUR BOX IF YOU'D LIKE TO; NO MORE FreeNAS WEBGUI SHELL
If you do, and you log in with the user you created earlier, you'll likely need to install ports using the `sudo` command. Oh, how convenient; we just installed that.

### 1.2.3. Port: git

1. `cd /usr/ports/devel/git`
2. `make install clean`
3. When it asks about `curl`, I install with LDAP/S and LIBSSH2
4. When `ca_root_nss-3.16.1` select ETCSYMLINK
5. DOCS not needed for `xmlto`, `getopt`, `libgcrypt`, `libgpg-error`, `docbook-xsl`, `xmlcatmgr`, `w3m`, `boehm-gc`, `libatomic_ops`
6. EXAMPLES not needed for `python27`, `p5*`

(this will take a while, but stick with it)


### 1.2.4. Port: MySQL

1. `cd /usr/ports/databases/mysql56-server`
2. `make install clean`


### 1.2.5. Port: sqlite3

1. `cd /usr/ports/databases/sqlite3`
2. `make install clean`
3. I turn on `MEMMAN`


### 1.2.6. Port: node
V8 JavaScript for client and server

1. `cd /usr/ports/www/node`
2. `make install clean`


### 1.2.7. Port: nginx

1. `cd /usr/ports/www/nginx`
2. `make install clean`
3. I turn on `HTTP_GEOIP`, `HTTP_GZIP_STATIC`, `HTTP_GUNZIP_FILTER`, `HTTP_PERL`, `MAIL`, `MAIL_SSL`, `HTTP_AUTH_LDAP`, `HTTP_AUTH_DIGEST`, `HTTP_FANCYINDEX`, `HTTP_PUSH`, `HTTP_PUSH_STREAM`, `HTTP_REDIS`, `HTTP_RESPONSE`, `HTTP_UPLOAD`, `HTTP_UPLOAD_PROGRESS` (you can always install more or less depending on what your server needs)
4. `echo 'nginx_enable="YES"' > /etc/rc.conf`

### 1.2.8. Port: rbenv (https://github.com/sstephenson/rbenv)

1. `cd /usr/ports/devel/rbenv`
2. `make install clean`
3. echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
4. echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

### 1.2.9. Port: ruby-build (https://github.com/sstephenson/ruby-build)

1. `cd /usr/ports/devel/ruby-build`
2. `make install clean`
3. You can leave RBENV checked; we already installed it.
4. Execute the commands we placed into our `.bash_profile`, `source ~/.bash_profile` (or `. ~/.bash_profile`)

# 2. Web Server

edit webserver jail get mac address fix router


Next we're going to install the things we need to get our server hooked up and running. We'll start with Ruby.

## 2.1. Setup
First however, we need to do all of this inside the `www` user, as it will be running nginx etc.

1. `sudo su - www`
2. `echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile`
3. `echo 'eval "$(rbenv init -)"' >> ~/.bash_profile`
4. `. ~/.bash_profile`

## 2.2. Ruby

1. `rbenv install 2.1.2`
2. `rbenv global 2.1.2`
3. `rbenv rehash`

(test to see if Ruby runs - "ruby -v" - if not, run "exec $SHELL -l")

## 2.3 Rails
Let's get Bundler and Rails installed, as we'll need those to get a basic project up and running.

1. `gem install bundler rails`
2. `gem install sqlite3 -- --with-sqlite3-dir=/usr/local`
3. `rbenv rehash`
4. `rails new sites/SITE_PROJECT_NAME`

## 2.4 Unicorn
Partial credit goes to:
* http://sirupsen.com/setting-up-unicorn-with-nginx


1. `gem install unicorn`
2. `rbenv rehash`
3. `ruby -e "$(curl -fsSL https://raw.github.com/TwilightCoders/FreeNAS-Rails-Setup/master/unicorn.rb -o sites/SITE_PROJECT_NAME/config/unicorn.rb)"`
4. Edit the unicorn.rb file that is now in your config directory for the rails project you made earlier. `vi sites/SITE_PROJECT_NAME/config/unicorn.rb`
5. Set `site_name` to the name of your project (SITE_PROJECT_NAME, or whatever you called it)
6. `:wq!` (write, quite, because I said so)


## 2.5 nginx
We've already installed nginx, so now it's time to do a little setup.

Partial credit goes to:
* http://bsdbox.co/2014/01/12/emp-nginx-mysql-php-fpm-on-freebsd
* http://theflyingdeveloper.com/server-setup-ubuntu-nginx-unicorn-capistrano-postgres

1. `cd /usr/local/etc/nginx`
2. `mkdir sites`
3. `mkdir conf.d`
4. `ruby -e "$(curl -fsSL https://raw.github.com/TwilightCoders/FreeNAS-Rails-Setup/master/options -o conf.d/options)"`
5. `ruby -e "$(curl -fsSL https://raw.github.com/TwilightCoders/FreeNAS-Rails-Setup/master/default.rails.site -o sites/default.rails.site)"`
6. `ruby -e "$(curl -fsSL https://raw.github.com/TwilightCoders/FreeNAS-Rails-Setup/master/nginx.conf -o nginx.conf)"`

# 3. Launch!
Alright! That wasn't too bad, now was it? Just a lot of coffee breaks waiting for things to compile/install.

Now the fun begins. From the 'www' user (`sudo su - www`)

1. `unicorn_rails -c /home/www/SITE_PROJECT_NAME/config/unicorn.rb -D`
2. `/etc/rc.d/nginx start`
3. Direct your web browser to the IP/domain of your server.
