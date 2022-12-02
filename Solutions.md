### Basic commandas

```
ansible -i hosts.ini example -a "free -h" -u [username]
ansible -i hosts.ini example -m ping -u [username]
```
```
ansible multi -a "hostname"
```
## to perform the command on each server in sequence
```
ansible multi -a "hostname" -f 1

```
## To check the disk space available for our application

```
ansible multi -a "df -h"
```
## To Check enough memory on our servers:

```
ansible multi -a "free -m"

```
## To check date and time
```
ansible multi -a "date"
```

### to install the chrony daemon on the server to keep the time in sync'
```
ansible multi -b -m yum -a "name=chrony state=present"

```
### chrony daemon is started and set to run on boot.

```
ansible multi -b -m service -a "name=chronyd state=started enabled=yes"
```
### to make sure our servers are synced closely to the official time on a time server:
```
ansible multi -b -a "chronyc tracking"
```
### Configure the Application servers and check version
```
ansible app -b -m yum -a "name=python3-pip state=present"
ansible app -b -m pip -a "name=django<4 state=present"
ansible app -a "python -m django --version"
```
### Configure the Database servers, Install mariadb and make sure its enabled
```
ansible db -b -m yum -a "name=mariadb-server state=present"
ansible db -b -m service -a "name=mariadb state=started \
   enabled=yes"
```

### configure the system firewall to ensure only the app servers can access the database:
```
ansible db -b -m yum -a "name=firewalld state=present"
ansible db -b -m service -a "name=firewalld state=started enabled=yes"
ansible db -b -m firewalld -a"zone=database state=present permanent=yes"
ansible db -b -m firewalld -a "source=192.168.60.0/24  zone=database state=enabled permanent=yes"
ansible db -b -m firewalld -a"port=3306/tcp zone=database state=enabled permanent=yes"
```

### MySQL access for one user from our app servers. The MySQL modules require the the PyMySQL module to be present on the managed server.

```
ansible db -b -m yum -a "name=python3-PyMySQL state=present"
ansible db -b -m mysql_user -a "name=django host=% password=12345 priv=*.*:ALL state=present"
```

### check the status of chronyd and restart the service on the  app server
```
ansible app -b -a "systemctl status chronyd"
ansible app -b -a "service chronyd restart" --limit "192.168.60.4"
#--limit argument to limit the command to a specific host in the specified group.
# Limit hosts with a simple pattern (asterisk is a wildcard).
ansible app -b -a "service ntpd restart" --limit "*.4"
# Limit hosts with a regular expression (prefix with a tilde).
ansible app -b -a "service ntpd restart" --limit ~".*\.4"

```
### Manage users and groups

#add an admin group on the app servers for the server administrators:
```
ansible app -b -m group -a "name=admin state=present"

```
# add the user johndoe to the app servers

```
ansible app -b -m user -a "name=johndoe group=admin createhome=yes"
```

# to delete the account
```
 ansible app -b -m user -a "name=johndoe state=absent remove=yes"
```
# Install git

```
ansible app -b -m package -a "name=git state=present"
```
### Manage files and directories

# Get information about a file
```
ansible multi -m stat -a "path=/etc/environment"
```

# Copy a file to the servers
  most file copy operations can be completed with Ansible’s copy module
  similar to scp and/or rsync
  more advanced file copy modules like rsync is avilable in ansible
  **note:** f you include a trailing slash, only the contents of the directory will be copied into the dest. If you omit the trailing slash, 
  the contents and the directory itself will be copied into the dest.
  to copy hundreds of files, especially in very deeply- nested directory structures use  unarchive module, or using Ansible’s 
  synchronize or rsync modules.
  
```
ansible multi -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```
# Retrieve a file from the servers
add the parameter flat=yes, and set the dest to dest=/tmp/ (add a trailing slash), to make Ansible fetch the files directly into the /tmp directory.
Only use flat=yes if you’re copying files from a single host.
```
ansible multi -b -m fetch -a "src=/etc/hosts dest=/tmp"
```
# Create directories and files

create files and directories (like touch), manage permissions and ownership on files and directories,
modify SELinux properties, and create symlinks.

```
ansible multi -m file -a "dest=/tmp/test mode=644 state=directory
ansible multi -m file -a "src=/src/file dest=/dest/symlink state=link # need to create a file before
```
# Delete directories and files
other file-management modules like lineinfile, ini_file, and unarchive.
```
ansible multi -m file -a "dest=/tmp/test state=absent"
```
# Run operations in the background
```
-B <seconds>: the maximum amount of time (in seconds) to let the job run.
-P <seconds>: the amount of time (in seconds) to wait between polling the servers for an updated job status
```

# Update servers asynchronously with asynchronous jobs
  
  can tasks in Ansible playbooks in the background, asynchronously, by defining an async and poll parameter on the play
  
  ```
  ansible multi -b -B 3600 -P 0 -a "yum -y update"
  ansible multi -b -m async_status -a "jid= <put it form above step"
  ```
# check log files
```
ansible multi -b -a "tail /var/log/messages"
ansible multi -b -m shell -a "tail /var/log/messages | grep ansible-command | wc -l"
```

# Manage cron jobs
```
ansible multi -b -m cron -a "name='daily-cron-all-servers' hour=4 job='/path/to/daily-script.sh'"
ansible multi -b -m cron -a "name='daily-cron-all-servers' state=absent"
```
# Deploy a version-controlled application
update the git checkout to the application’s new version branch, 1.2.4, on all the app servers:
```
ansible app -b -m git -a "repo=git://example.com/path/to/repo.git dest=/opt/myapp update=yes version=1.2.4"
ansible app -b -a "/opt/myapp/update.sh"
```







## Solution 2

Using a playbook ~/playbooks/apache.yml (create new if doesn't exist) perform the below given tasks on node01:


a. Install httpd and php packages.

b. Change default document root of Apache to /var/www/html/myroot in default Apache config /etc/httpd/conf/httpd.conf. Make sure /var/www/html/myroot path exists.

c. There is a template ~/playbooks/templates/phpinfo.php.j2 on ansible controller node. Copy this template to Apache document root on node01 host as phpinfo.php file and make sure owner and group owner is apache user.

d. Start and enable httpd service.

e. Add rule in firewalld public zone to open http port 80 for public access so that phpinfo.php page is accessible in browser, also rule should be permanent.
