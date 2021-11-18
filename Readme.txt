 1. EC2 installation
2. Java installation
$ yum install -y java-1.8.0-openjdk-devel.x86_64
$ alternatives --config java
$ java -version

3. Jenkins install
$ wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
$ rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
$ yum install -y jenkins

4. Start Jenkins
$ systemctl start jenkins

5. Setting passwor




1. Web server installation
#!/bin/bash
yum update -y
yum install httpd -y
yum install git -y
amazon-linux-extras install epel
yum install epel-release
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
yum install -y php70 php70-php php70-php-fpm php70-php-pecl-memcached php70-php-mysqlnd php70-php-xml
ln -s /usr/bin/php70 /usr/bin/php
service httpd start
chkconfig httpd on


2. Configure Apache( để deploy code)
$ useradd www-user 

$cd /var/www/html/
$mkdir tinhocthatladongian
$chown -Rf www-user:apache tinhocthatladongian   (phân quyền cho phép www-user được phép truy cập vào apache và tinhocj... để lấy code rồi build)

#Cấu hình cho apache
vi /etc/httpd/config/httpd.conf

#vì mình ko có tên miền , nên mình sẽ tạo ra 1 con máy ảo để chạy web ở cổng 81

Listen 81
NameVirtualHost *:81
<VirtualHost *:81>
    DocumentRoot /var/www/html/tinhocthatladongian/
    <Directory "/var/www/html/tinhocthatladongian">
      Order deny,allow
      Allow from all
      AllowOverride All
      Require all granted
   </Directory>
</VirtualHost>

service httpd start
service httpd restart

3. Creati ng ssh key
$ sudo su - www-user
$ ssh-keygen -t rsa
$ cd /home/www-user/.ssh/
$ mv id_rsa.pub authorized_keys

4. Copy private key to Jenkin server
cd /home
mkdir jenkins
vi web-key.pem
chown -Rf jenkins:root
chmod 400 web-key.pem

- Test connect to web server
vi /etc/passwd
jenkins:x:996:994:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
su - jenkins
ssh www-user@[Web Server Address] -i /home/jenkins/web-key.pem 

5. Github configuration
Webhook
http://[Jenkin URL]:8080/github-webhook/ 

6. Jenkins configuration
6.1 Git configuration

6.2 Build Triggers
Check on "GitHub hook trigger for GITScm polling"

6.3 Build 
Execute shell

#!/bin/bash
ssh -T -i /home/jenkins/web-key.pem www-user@[Web Server Address] << EOF
cd /var/www/html/tinhocthatladongian
git pull
EOF
1