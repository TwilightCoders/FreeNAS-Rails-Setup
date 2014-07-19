# New Jail
* **name:** webserver
* **type:** portjail (x64)
Click 'save'


1. Click 'jails'
1. Click 'webserver'
1. Click 'shell'

## Ports
In the shell that you launched for the web server jail you created we need to fetch extract and update our ports.

1. portsnap fetch
1. portsnap extract
1. portsnap update

### Port: bash

`cd /usr/ports/shells/bash`
`make install clean`

This will prompt for configuration options. Configure as you see fit. I recommend (in the subsequent dependent port installations, disabling EXAMPLES *pic here*

## SSH access

1. `vi /etc/rc.conf`
2. change `sshd_enable="NO"` to `sshd_enable="YES"`

parameters: needs to belong to group "wheel" - reason? to enable SSH
also make sure when choosing a shell input "bash" - otherwise csh will default, and this won't work

### adduser
1. type: `adduser`
2. Invite to login group 'wheel'
3. Set shell path to bash (/usr/local/bin/bash)


Step 3: Install Git

(can be done anywhere)
portsnap fetch
portsnap extract

Now you can navigate to:
cd /usr/ports/devel/git
make install clean

(this will take a while, but stick with it)


Step 4: Install MySQL:

cd /usr/ports/databases/mysql56-server
make install clean


Step 5: Install rbenv

cd
git clone git://github.com/sstephenson/rbenv.git .rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
source ~/.bash_profile

(this only works because we installed git, and we are using bash)

Step 6: Install Ruby

rbenv install 2.1.2
rbenv global 2.1.2
rbenv rehash

(test to see if Ruby runs - "ruby -v" - if not, run "exec $SHELL -l")

Step 7: Install Rails

gem install bundler rails
