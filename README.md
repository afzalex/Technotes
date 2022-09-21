# Technical Notes by **Afzal**
Server Setup Repository : https://github.com/afzalex/serversetup.git
About Repository : https://github.com/afzalex/about.git

# Configurations


### EC2 user-data to add web server with web page on 80 port number
```bash
#!/usr/bin/env bash
su ec2-user
sudo yum install httpd -y
sudo service httpd start

cat <<EOF | tee /tmp/index.html
<html>
    <head>
        <title> In the Pale Moonlight</title>
        <style>
            html, body { background: #000; padding: 0; margin: 0; }
            img { display: block; margin: 0px auto; }
        </style>
    </head>
    <body>
        <img src='https://c.tenor.com/ITctI_ZpHIoAAAAM/brain-linux.gif' height='100%' />
    </body>
</html>
EOF

sudo mv /tmp/index.html /var/www/html/index.html
```


### Enable docker remote API
```bash
sudo vim /lib/systemd/system/docker.service
```
now search following line

    ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
and modify it to add `-H tcp://0.0.0.0:2375`

    ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock
now reload docker daemon
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```


### Configuring VIM according to our needs 
Edit or Create ~/.vimrc 
```vi
set number
set tabstop=4
set softtabstop=4
set shiftwidth=4
set noexpandtab
set colorcolumn=110
set autoindent
highlight ColorColumn ctermbg=darkgray
```

### Automate git password authentiation
- On Windows :
    ```sh
    git config --global credential.helper wincred
    ```
    OR 
    to persist password for specific time period
    ```sh
    git config --global credential.helper 'cache --timeout=3600'
    ```
- On Linux : 
    ```sh
    git config credential.helper store
    ```


### Adding alias to view commit tree of git in CUI (shell/console)

Add following line in git config file i.e. ~/.gitconfig
```vi
[alias]
    flog = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
```
Now formatted tree could be seen with following command
```sh
git flog
```

**OR**  

Now formatted tree could be seen with following command
```sh
curl https://www.afzalex.com/scripts/install-flog.sh | /bin/bash
```

### Setting up rsync to backup data
Create a file /etc/rsyncd.conf
```vi
motd file = /etc/rsyncd.motd
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock

/home/afzal/tempo/
	path = /home/afzal/tempo/
	comment = Temporary rsync location 
	uid = nobody
	gid = nobody
	read only = no
	list = yes
	auth users = afzal
	secrets file = /etc/rsyncd.scrt
```
Now enable rsync using systemctl
```sh
systemctl enable rsync.service
```
Add command in crontab to execute every day at 1:00 PM
```sh
crontab -e 
```
Enter below code opened window
```cron
00 13 * * * rsync --verbose --stats --compress --recursive --times --perms --links --exclude=Downloads/ /home/afzal/ /var/backups/afzal/`date +"\%a"`  >> /var/log/backup.log
```
Above command will sync **/home/afzal** into **/var/backups/afzal/Mon** where "Mon" value could be "Tue", "Wed", ..., "Sun" and output will be logged in **/var/log/backup.log**

Mind giving required previleges to files and directories used in this process

### Setting static ip in local network
Check network devices attached
```sh
ifconfig
```
Note configuration of device you want to set for static ip
Edit **/etc/network/interfaces**

### Making VirtualBox IP permanent
Edit **/etc/network/interfaces**
```vi
auto eth1 # this refers to the Host-only network interface
iface eth1 inet static
address 192.168.56.10 # Arbitrary IP address
netmask 255.255.255.0
```

### Setting linux to bootup with multicores
Edit **/etc/init.d/rc** and replace following line
```
CONCURRENCY=none
```
with this
```
CONCURRENCY=shell
```

### Speeding up ubuntu boot speed
Reduce the default grub boot time
```sh
sudo vim /etc/default/grub
GRUB_TIMEOUT=2
```
```sh
sudo update-grub
```
If apt-daily.service taking time to boot
```
sudo systemctl edit apt-daily.timer
# apt-daily timer configuration override
[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
AccuracySec=1h
RandomizedDelaySec=30min
```
<br>
<br>

---
# Tasks

### View CSV file in terminal
To use below solution, nodejs is required.
```bash
npm i -g tty-table
cat data.csv | tty-table
```


### Replace part of text file with content from another file
Consider following textual file `test_bash_file.sh`
```sh
#!/bin/bash
echo "Hello World"

#MANAGED_BLOCK_START
#Please don't modify this text, it is managed by replacement script
echo "This is the content to be replaced"
#MANAGED_BLOCK_END

echo "above content should be managed via replacement script"
```
And another textual file `test_scriptlet.dat` whose content is to be substituted in `test_bash_file.sh`
```bash
#Please don't modify this text, it is managed by replacement script
echo "Replaced Text"
echo "This is the content from replacement script"
```
Run following command to replace part of text file `test_bash_file.sh` with `test_scriptlet.dat`
```bash
awk '
    BEGIN       {p=1}
    /^#MANAGED_BLOCK_START/   {print;system("cat  test_scriptlet.dat");p=0}
    /^#MANAGED_BLOCK_END/     {p=1}
    p' test_bash_file.sh
```

To replace text and substitute environment variables in replaced text
```bash
awk '
    BEGIN       {p=1}
    /^#MANAGED_BLOCK_START/   {print;system("cat  test_scriptlet.dat  | envsubst");p=0}
    /^#MANAGED_BLOCK_END/     {p=1}
    p' test_bash_file.sh
```

### Merge audio from different video to other video file
```bash
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```
with video re-encoding
```bash
ffmpeg -i video.mp4 -i audio.wav -c:v libx264 -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

### Reduce video size
- Audio
	- Audio quantity : 1
	- Codec : MP3 
	- Bitrate : 160 (Or 256 or 320 if music is main element)
- Codec and Container
	- Codec : H.264
	- Container : MP4
- Video resolution
    ```
	2160p (3840×2160)
	1440p (2560×1440)
    1080p (1920×1080)
	720p (1280×720)
	480p (854×480)
	360p (640×360)
    240p (426×240)
    ```
- Bitrate

    | TYPE |Bitrate: 24-30fps|Bitrage: 48-60fps |
    |---|---|---|
	| 2160p(4k)	| 35-45 Mbps| 53-68 Mbps|
	| 1440p(2k)	| 16 Mbps	| 24 Mbps	|
	| 1080p		| 8 Mbps	| 12 Mbps	|
	| 720p		| 5 Mbps	| 7.5 Mbps	|
	| 480p		| 2.5 Mbps	| 4 Mbps	|
	| 360p		| 1 Mbps	| 1.5 Mbps	|
```sh
ffmpeg -i <inputfile> -acodec mp3 -vcodec h264 -b:a <audio_bitrate> -vf scale=<resolution_width>:<resolution_height> <outputfile_mp4>
ffmpeg -i input.mov -acodec mp3 -vcodec h264 -b:a 160k -vf scale=1280:720 output.mp4
```

### Creating ssl certificate / certificate authority / self sign certificate
1. Generate private rsa key
    ```sh
    openssl genrsa -aes256 -out server.key 1024
                    ^optional     ^keyName   ^keySize
    ```
    `p1` - prompted password if aes256 (or other encryption) is used
    [p1] -> server.key
2. Generate a CSR (Certificate Signing Request)
    ```sh
    openssl req -new -key server.key -out server.csr
                             ^keyName        ^csrFile
    ```
    Information will be asked about certificate. 
    Common Name should match hostname e.g. for **http://www.google.com** value should be **www.google.com**
    `p2` - Challenge Password
    [server.key, p2] -> server.csr
3. Remove Passphrase from Key (if rsa key is encrypted)
    ```sh
    cp server.key server.key.org
    openssl rsa -in server.key.org -out server.key
    ```
4. Generating a Self-Signed Certificate
    ```sh
    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
    ```
    [server.csr, server.key] -> server.crt
5. Converting certificate into PKCS12
    ```
    openssl pkcs12 -export -in server.crt -inkey server.key -out server.p12 -name default -CAfile server.crt -caname root
    ^aliasName
    ```
    `p3` - pkcs12 password
    [server.crt, server.key, p3] -> server.p12
6. Import pkcs12 into keystore
7. 
    ```
    keytool -importkeystore -deststorepass <paswd> -destkeypass <paswd> -destkeystore server.jks -srckeystore server.p12 -srcstoretype PKCS12 -srcstorepass <csr paswd> -alias default
                                              ^p4                  ^p5                                                                                         ^p2
    ```
    [server.p12, p2] -> server.jks


### Verification of keys
Verify a Private Key
:   ```sh
    openssl rsa -check -in server.key
    ```

Verify a Private Key Matches a Certificate and CSR
:  ```sh
    openssl rsa -noout -modulus -in server.key | openssl md5
    openssl x509 -noout -modulus -in server.crt | openssl md5
    openssl req -noout -modulus -in server.csr | openssl md5 
    ```
    If the output of each command is identical there is an extremely high probability that the private key, certificate, and CSR are related.

### Setup gmail to allow other applications to send mail on your behalf

Visit following links (self explanatory)
> https://myaccount.google.com/lesssecureapps
> https://accounts.google.com/b/0/DisplayUnlockCaptcha

Port used to connect with ssl : 465
Port used to connect with tls : 587

### Check which port is used by which application

```sh
netstat -plntu
```


### Install a .deb file from terminal
```sh
sudo dpkg -i <PATH_TO_DEB_FILE>
sudo apt-get -f install
```


### Making notify-send work in shell. (To make notify-send work in cron i.e. crontab and ssh)
```sh
eval "export $(egrep -z DBUS_SESSION_BUS_ADDRESS /proc/$(pgrep -u $LOGNAME gnome-session)/environ)";
notify-send "TITLE" "YOUR_MESSAGE"
```

###  Disable notify-send
```sh
mv /usr/share/dbus-1/services/org.freedesktop.Notifications.service /usr/share/dbus-1/services/org.freedesktop.Notifications.service.disabled`
```



### Adding git with ssh protocol on windows
Open git-bash 
```sh
ssh-keygen
vim ~/.bashrc
```
Enter following lines in editor
```vi
eval `ssh-agent`
ssh-add
```
Restart git-bash
```sh
rm ~/.bashrc
```
Close git-bash
RSA public key of ssh is present in .ssh directory inside %userprofile% 


### Convert openssh private key to pem format
```sh
ssh-keygen -p -N "" -m pem -f /path/to/key
```

### Analyze which service taken how much time during startup or bootup
```sh
systemd-analyze blame
```

### Linux Sending mail from command line
MSMTP (SMTP client)
:   >(Documentation could be found at http://msmtp.sourceforge.net/doc/msmtp.html)
    
    ```sh
	sudo apt-get install msmtp
    vim ~/.msmtprc
	```
	Now in opened editor set default values for all following accounts.
	```vi
	# Set default values for all following accounts.
	defaults
	auth           on
	tls            on
	tls_trust_file /etc/ssl/certs/ca-certificates.crt
	logfile        ~/.msmtp.log

	# Gmail
	account        gmail
	host           smtp.gmail.com
	port           587
	from           username@gmail.com
	user           username
	password       plain-text-password

	# A freemail service
	account        freemail
	host           smtp.freemail.example
	from           joe_smith@freemail.example
	...

	# Set a default account
	account default : gmail
	```

### Linux fetching mail from command line
fetchmail (remote-mail retrieval and forwarding utility intended to be used over on-demand TCP/IP)
`... PENDING ...`
			
	
### Adding ssh client in linux
```sh
sudo apt-get install openssh-server
```
Create a file **~/.ssh/authorized_keys** if not exist
```sh
touch ~/.ssh/authorized_keys
```
Append your public key in this file
```sh
cat rsa_public_key.pub >> ~/.ssh/authorized_keys
```
Now you can access this linux from windows by following command
```sh
ssh -i rsa_private_key username@ipaddress
```
You can use ssh-keygen to create public private rsa key pair



### Forcefully disconnect an ssh client
find process id of ssh for client
```sh
sudo netstat -tnpa | grep ssh
```
kill process
```sh
kill -9 <pid>
```



### Logging ssh session   `Important`
Download log-session script
```sh
wget http://www.jms1.net/log-session
```
Find out where the sftp-server binary is located
```sh
grep sftp /etc/ssh/sshd_config
```
Edit log-session file and replace following content
```sh
SFTP_SERVER=<location_of_sftp_server>
SFTP_SERVER=/usr/lib/openssh/sftp-server
```
Make log-session file executable
```sh
chmod 755 log-session
```
Edit ~/.ssh/authorized_keys and append following
```vi
command="<location_of_log_session>"
command="/usr/local/sbin/log-session" ssh-dss AAAAB3Nz...
```


### Securing your password with public key encryption
Securing your password (or anything for the secure transmission of information between parties) with public key encryption
Install gnupg
```sh
sudo apt-get install gnupg
```
Create key pair (give desired inputs and wait until key is created)
```sh
gpg --gen-key
```
To encrypt 
```
<CMD TO O/P> | gpg -e -r <RECIPIENT>
```
To decrypt
```
<CMD TO O/P> | gpg -d
```
To get list of keys
```
gpg -k
```
To get public key
`... PENDING ...`
	
	
### Downloading entire site with wget
```sh
wget \
 --recursive \
 --no-clobber \
 --page-requisites \
 --html-extension \
 --convert-links \
 --restrict-file-names=windows \
 --domains website.org \
 --no-parent \
     www.website.org/tutorials/html/
```

### Persist iptables configuration
install iptables-persistent
```sh
sudo apt-get install iptables-persistent
```
save iptables configuration with iptables-persistent
```sh
sudo iptables-persistent save
```

### Setting iptables to redirect port (could be used to set wildfly to get request from 80 port)
```sh
sudo iptables -A PREROUTING -t nat -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### Setting iptables to redirect port on local machine
```sh
sudo iptables -t nat -I OUTPUT -p tcp -o lo --dport 80 -j REDIRECT --to-ports 8080
```

	
### Get linux distribution information
```sh
lsb_release -a
cat /etc/*-release
uname -a
cat /proc/version
```


### Compress, zip, extract files directory etc
```sh
tar -cvzf <filename_with_.tar.gz_extension> <directory_or_file>
tar -xvzf <filename_with_.tar.gz_extension> 
tar -cvzf - <file1> <file2> ... <filen>
```

### Continue last transaction of package manager like apt-get or yum
```sh
yum-complete-transaction [--cleanup-only] 
yum history redo last
```


### View image from command line
Using caca to view image with characters
:   ```sh
    sudo apt-get install caca-utils
    cacaview <any_image_.jpg>
    ```

Using fbi which will use framebuffer
: ```sh
    sudo apt-get install fbi
    fbi <any_image_.jpg>
    ```

### Creating desktop launcher for an application
vim /home/$HOME/.local/share/applications/`<application_name>`.desktop
```vi
[Desktop Entry]
Version=1.0
Name=<Application Name>
Comment=<e.g. Java IDE>
Type=Application
Categories=<e.g. Development;IDE;>
Exec=<application location e.g. /home/${USERNAME}/applications/eclipse/eclipse>
Terminal=false
StartupNotify=true
Icon=<icon location e.g. /home/${USERNAME}/applications/eclipse/icon.xpm>
Name[en_US]=<Application Name e.g. Eclipse>
```
<br>
<br>

--- 
# Installations

### Installation scripts to setup environment 
alpine setup environment
```sh
<<<<<<<<<<<<<<<<<<<<<<<<
```

### Adding new php version in wamp server
	
1.	create new folder [path-to-wamp]/bin/php/php.#.#.# and copy files here
2. 	copy following files from older php
	1. php.ini
	2. phpForApache.ini
	3. wampserver.conf
3.	Take snapshots of wamp>PHP>PHP Settings and wamp>PHP>PHP Extendsion
4.	Open wamp>PHP>php.ini and save it as a backup
5.	Restart wamp , Change wamp>PHP>Version>latest_version
6.	Use a diff tool to get differences between old php.ini and new one to satisfy all extensions

		
### Installing LAMPP
	
1.	Install apache web server
    ```sh
    sudo apt-get install apache2
    ```
	/etc/apache2/apache2.conf contains configurations
	/etc/apache2/ports.conf contains ports configuration
2. Install PHP
    ```sh
    sudo apt-get install php5 libapache2-mod-php5
    ```
    directory for lookup is /var/www/html
3.	Install mysql
    ```sh
    sudo apt-get install mysql-server
    ```
	add following lines in /etc/apache2/apache2.conf
	```vi
	#phpMyAdmin Configuration
	Include /etc/phpmyadmin/apache.conf
	```
4.	In Ubuntu also need to run following commands to make mcrypt recognized in phpmyadmin/install mcrypt right way
	```sh
	php5enmod mcrypt
	```

### Installing LEMP
1.  Install nginx 
    ```sh
    sudo apt-get install nginx
    ```
	Edit /etc/nginx/sites-available/default
	(If /usr/share/nginx/www does not exist, it's probably called html. Make sure you update your configuration appropriately)
	1. Add index.php to the index line.
	2. Change the server_name from local host to your domain name or IP address (replace the example.com in the configuration)
	3. Change the correct lines in “location ~ \.php$ {“ section
	    ```vi
		server {
			listen   80;
	
			root /usr/share/nginx/www;
			index index.php index.html index.htm;

			server_name example.com;

			location / {
					try_files $uri $uri/ /index.html;
			}

			error_page 404 /404.html;

			error_page 500 502 503 504 /50x.html;
			location = /50x.html {
				  root /usr/share/nginx/www;
			}
			
			# pass the PHP scripts to FastCGI server listening on the php-fpm socket
			location ~ \.php$ {
					try_files $uri =404;
					fastcgi_pass unix:/var/run/php5-fpm.sock;
					fastcgi_index index.php;
					fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
					include fastcgi_params;
					
			}
		}
		```
		
2.  Install mysql with php-mysql
	```sh
    sudo apt-get install mysql-server php5-mysql
    ```
	Activate mysql
	```sh
	sudo mysql_install_db
	```
	setup script
	```sh
	sudo /usr/bin/mysql_secure_installation
	```
	
3. 	Install PHP
    ```sh
	sudo apt-get install php5-fpm
	```
	
	Configure php
	Edit php.ini file vim /etc/php5/fpm/php.ini
    ```vi
    cgi.fix_pathinfo=0
    ```
    
	Edit /etc/php5/fpm/pool.d/www.conf
	Find the line, `listen = 127.0.0.1:9000`, and change the `127.0.0.1:9000` to `/var/run/php5-fpm.sock`. 
	```vi
	listen = /var/run/php5-fpm.sock
	```
	Restart php-fpm
	```sh
    sudo service php5-fpm restart
    ```
    
4. Create info.php
    ```sh
	sudo nano /usr/share/nginx/html/info.php
	```
	```vi
	<?php
	phpinfo();
	?>
	```
	restart nginx
	```sh
	sudo service nginx restart
	```

	
### Installing Composer in Linux
1. Download the installer to the current directory
2. Verify the installer SHA-384 either by below command or cross checking at https://composer.github.io/pubkeys.html
	php -r "if (hash('SHA384', file_get_contents('composer-setup.php')) === 'fd26ce67e3b237fffd5e5544b45b0d92c41a4afe3e3f778e942e43ce6be197b9cdc7c251dcde6e2a52297ea269370680') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); }"
3.	Run the installer
		php composer-setup.php
4.	Remove the installer
		php -r "unlink('composer-setup.php');"
5.	To install composer Globally move the downloaded file to /usr/local/bin/composer
		mv composer.phar /usr/local/bin/composer

### Installing Laravel 
On Windows 
:   ```sh
    composer global require "laravel/installer"
    ```
On Linux using local composer.phar
:    ```sh
	php composer.phar global require "laravel/installer"
	```

**Plugin to add laravel framework in netbeans**
*https://github.com/nbphpcouncil/nb-laravel-plugin/releases*

			
	
### Installing java manually
Download .tar.gz file (preferred to be downloaded from oracle's site)
```sh
wget  --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" <url>
```

Extract .tar.gz file in /opt directory
```sh
tar -zxvf jdk-*u**-linux-****.tar.gz
```
Create symbolic link in order to simplify java updates in future
```sh
ln -s /opt/jdk1.8.0_144 /opt/java
```
Tell system where java and ts executables are intalled.
```sh
update-alternatives --install /usr/bin/java java /opt/java/bin/java 100
update-alternatives --config java
```
Create necessary environment variables
```sh
sudo vim /etc/profile.d/java.sh
```
```bash
if ! echo ${PATH} | grep -q /opt/java/bin ; then
	export PATH=/opt/java/bin:${PATH}
fi      
if ! echo ${PATH} | grep -q /opt/java/jre/bin ; then
   export PATH=/opt/java/jre/bin:${PATH}
fi      
export JAVA_HOME=/opt/java
export JRE_HOME=/opt/java/jre
export CLASSPATH=.:/opt/java/lib/tools.jar:/opt/java/jre/lib/rt.jar
```
```sh
sudo chmod 755 /etc/profile.d/java.sh
```


### Uninstall a package properly on Ubuntu
```sh
sudo apt-get purge git; 
sudo apt-get autoremove;
```
now delete related files if exist in your home directory 
```sh
rm ~/.gitconfig
```
			
### Installing postgresql debugger
Edit postgresql.conf file present in  c:\program files\postgresql\9.3\data directory
Un-comment or add this line:
```vi
shared_preload_libraries = '$libdir/plugin_debugger.dll'
```
Restart PostgreSQL server
In the required database run following command 
```sql
create extension pldbgapi;
```
		

### Maven download / install sources and javadocs
Download sources
```sh
mvn dependency:sources
```
Download docs
```sh
mvn dependency:resolve -Dclassifier=javadoc
```
Download sources of specific package
```sh
mvn dependency:sources -DincludeArtifactIds=guava
```
Add plugin in pom.xml
```pom
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-eclipse-plugin</artifactId>
    <configuration>
        <downloadSources>true</downloadSources>
        <downloadJavadocs>true</downloadJavadocs>
    </configuration>
</plugin>
```

### Maven run a particular class
Directly from command line
:   ```
    mvn exec:java -Dexec.mainClass="com.example.Main"
    mvn exec:java -Dexec.mainClass="com.example.Main" -Dexec.args="arg0 arg1"
    ```
Using plugin
:   ```pom
	<plugin>
	  <groupId>org.codehaus.mojo</groupId>
	  <artifactId>exec-maven-plugin</artifactId>
	  <version>1.2.1</version>
	  <executions>
		<execution>
		  <goals>
			<goal>java</goal>
		  </goals>
		</execution>
	  </executions>
	  <configuration>
		<mainClass>com.example.Main</mainClass>
		<arguments>
		  <argument>foo</argument>
		  <argument>bar</argument>
		</arguments>
	  </configuration>
	</plugin>
	```


### Mongodb on Ubuntu-16.0
*Reference taken from https://www.howtoforge.com/tutorial/install-mongodb-on-ubuntu-16.04*
Importing key
```sh
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
```
Create source list file MongoDB
```sh
echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get install mongodb-org
```
Create a new mongodb systemd service file in the '/lib/systemd/system' directory.
```sh
cd /lib/systemd/system/
vim mongod.service
```
Now update the systemd service with command below:
```sh
systemctl daemon-reload
```
Start mongodb and add it as service to be started at boot time:
```sh
systemctl start mongod
systemctl enable mongod
```

*Further to add mongodb in php*
```sh
composer require mongodb/mongodb
```


### Generic runnable/executable file/application as service in ubuntu
Edit /etc/systemd/system/prometheus.service
```vi	
[Unit]
Description=Prometheus Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/prometheus/prometheus --config.file=/usr/local/bin/prometheus/prometheus.yml

[Install]
WantedBy=multi-user.target 
```
```sh
sudo service prometheus start
```
<br>
<br>

---
# Setup
		
### Disabling lightdm (or other service) 
1. Method 1
	```sh
    echo manual | sudo tee etc/init.d/lightdm.override
    ```
	i.e. create <service>.override file to disable it (override it)
	*This method is not working after 14.0*

2. Method 2 
	edit /etc/default/grub
	replace 	GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
	with		GRUB_CMDLINE_LINUX_DEFAULT="text"
	```sh
    sudo update-grub
    ```

### Disable ssh login without password
1.  Edit /etc/ssh/sshd_config and change following settings
    ```vi
	ChallengeResponseAuthentication no
	PasswordAuthentication no
	UsePAM no
	```
	```sh
	sudo /etc/init.d/ssh reload
	```
	
### Setting git to login wihout password + using ssh in git
Create ssh-key
```sh
ssh-keygen
```
Copy your .pub file in remote git application from where you want to connect
Add your private key using ssh-agent into your system (if you don't want to provide key every time)
```sh
ssh-add ~/.ssh/id_rsa
```
<br>
<br>

---
# Useful commands

Getting list of installed packages
:   ```sh
    dpkg --get-selections | grep -v deinstall
    ```

<br>
Copying files from one machine to other 
:   ```sh
    scp /file_in_current_system root@target_machine /path_of_destination
    ```

<br>
Getting information of current distribution
:   ```sh
    uname -r
	cat /etc/*-release
	lsb_release -a
	cat /proc/version
	```

<br>
Download content
:	  curl

<br>		
Download file
:	  wget

<br>
json formatter with stream
:	  jq

<br>
Generic syntax highlighter
:	```sh
    pygmentize -l xml
	sudo apt-get install python-pygments
	sudo apt-get install python-image
	```

<br>
Formatter for xml
:   ```sh
    sudo apt-get install libxml2-utils			
	xmllint
	```

<br>
Get info of ip addresses in network (computer name, logged in user etc..)
:   ```sh
    sudo apt-get install nbtscan		
	nbtscan 192.168.1.0/24
	```

<br>
Network scanner. (find hosts systems and open ports on systems.)
:   ```sh
    sudo apt-get install nmap
	nmap <ip-address>
	```

<br>
Arbitrary TCP and UDP connections and listens 
:	  nc

<br>
To record voice 
:	  avconv -f pulse -i default file.wav

<br>
To edit sam file
:	  chntpw

<br>
To get mouse coordinates, open windows etc
:	  xdotool

<br>
Controlling audio devices
:   ```sh
    amixer -D pulse sset Master 5%+
	alsamixer
	```
	<br>
	If amixer is not found, install it using below command
	```sh
	sudo apt-get install alsa-utils
	```

<br>
Control menubar time format or modify anyway
:	gsettings set com.canonical.indicator.datetime show-seconds  true

<br>
Remove/Delete file completely (permanently)
:	  shred -zun3 -f <file name>

<br>
Reduce JPEG file size
:	  jpegoptim

<br>
Create Conda Environment
:     ```sh
      conda create --name email-sending-env python=3.7
	  ```

<br>

### Delete all idle connections of postgresql
```sql
SELECT
  DATE_TRUNC('second',NOW()-query_start) AS age,
  pg_terminate_backend(pid),
  *
FROM
  pg_stat_activity
WHERE NOW() - query_start > '00:10:00'
ORDER BY
  age DESC;
```

### git autocomplete feature in Mac
```sh
curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash > ~/.git-auto-complete.bash
source ~/.git-auto-complete.bash
```



### Use netcat or nc to execute a command or script remotely
    while true; do if [[ $(nc -lp 1234 2> /dev/null) == 'install' ]]; then ./myexec.sh; fi; done




### Useful python commands for page number converter
```python
",".join([ str(i) for i in range(1, 102) if int((i + 1) / 2) % 2 != 0])
",".join([ str(i) for i in range(1, 102) if int((i + 1) / 2) % 2 == 0])
```
<br>
<br>

---
# Informations
* In virtualbox guest 10.0.2.2 will be the IP of host
* Accessing guest from host in virtualbox
*  In virtual box   settings > Network > Attached to : NAT
   1. In virtual box   settings > Network > Attached to : NAT
   2. In guest type ifconfig
* In Ubuntu the packages installed are stored in /var/cache/apt/archives
* Crontab could be used to schedule tasks in linux. Command to edit crontab is
    ```sh
    crontab -e
    ```
* Location to install your own sh files so that it could be used as commands **/usr/local/bin/**
<br>
<br>

---
# Code

### Bash code to auto reload file and trigger some command
```bash
#!/bin/sh

FILE="/root/default.conf"
COPYLOC="/etc/nginx/http.d/default.conf"

cp -f $FILE $COPYLOC
LT=`stat -c %Z $FILE`; 
while true
do 
    AT=`stat -c %Z $FILE`
    if [[ "$AT"  != "$LT" ]]; then 
        cp -f $FILE $COPYLOC
        sleep 1
        nginx -s reload
        LT=$AT
        echo `date "+%Y/%m/%d %H:%m:%S"` [reloader] Default config file reloaded
    fi
    sleep 1
done
```


### Python code to send a simple text email
```python
import smtplib, ssl
from getpass import getpass

port = 465  # For SSL
smtp_server = "smtp.gmail.com"
sender_email = "afzalex.store@gmail.com"  # Enter your address
receiver_email = "afzalex.testing@gmail.com"  # Enter receiver address
password = getpass("Type your password and press enter: ")
message = """\
Subject: Hi there

This message is sent from Python.
"""

context = ssl.create_default_context()
with smtplib.SMTP_SSL(smtp_server, port, context=context) as server:
    server.login(sender_email, password)
    server.sendmail(sender_email, receiver_email, message)
```


### Python code for page number converter
```python
",".join([ str(i) for i in range(1, 102) if int((i + 1) / 2) % 2 != 0])
",".join([ str(i) for i in range(1, 102) if int((i + 1) / 2) % 2 == 0])
```
<br>
<br>

---
[Edit Technotes](https://github.com/afzalex/technotes/edit/main/README.md) | v2
