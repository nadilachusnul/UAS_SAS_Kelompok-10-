## Final Project - Sistem Administrasi Server

Nadila Chusnul K - 1202190020 \
Anastasya Rahma Juniarti - 120219058\
Kelompok 10 

 Login ssh terlebih dahulu 
      ![A1](aset/G1.jpg)

Buat LXC :
## Install LXC php ubuntu 

-  6 LXC ubuntu PHP 7.4
    
```bash
sudo lxc-create -n lxc_php7_1 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_2 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_3 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_5 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_6 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
 ```
   ![A1](aset/G2.jpg)
    ![A1](aset/G3.jpg)
      ![A1](aset/G4.jpg)
        ![A1](aset/G5.jpg)
          ![A1](aset/G6.jpg)
            ![A1](aset/G7.jpg)

## Install LXC php Debian 
- 2 LXC debian PHP 5.6

```bash
sudo lxc-create -n lxc_php5_1 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php5_2 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```
 ![A1](aset/G8.jpg)
    ![A1](aset/G9.jpg)

## Install LXC php mariadb
- 1 LXC debian mariadb server 

```bash
sudo lxc-create -n lxc_mariadb -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```
![A1](aset/G10.jpg)

Masuk ansible 
```bash
cd ~/ansible/UAS 
```
![A1](aset/G11.jpg)

Setting autostart, dan IP setiap lxc, ssh
```bash
lxc-ls - f
```
![A1](aset/G12.jpg)

Setting Nginx vm
```bash
nano /etc/nginx/sites-available/kelompok10.fpsas 
```
```bash
  upstream laravel {
        least_conn;
        server lxc_php7_1.dev;
        server lxc_php7_4_laravel.dev;
        server lxc_php7_6_laravel.dev;
        server lxc_php7_2_laravel.dev;
  }
  upstream yii {
        server lxc_php7_2_yii.dev weight=2;
        server lxc_php7_1_yii.dev weight=3;
        server lxc_php7_4_yii.dev weight=4;
        server lxc_php7_5_yii.dev weight=1;
        server lxc_php7_6_yii.dev weight=6;
   }
  upstream ci {
        server lxc_php5_1.dev;
        server lxc_php5_2.dev;
  }
  upstream wp {
        ip_hash;
        server lxc_php7_3_wp.dev;
        server lxc_php7_2.dev;
        server lxc_php7_4_wp.dev;
        server lxc_php7_5_wp.dev;
  }

  server {
          listen 80;
          listen [::]:80;

          server_name kelompok10.fpsas;

          root /var/www/html;
          index index.html;

          location /app {
                  rewrite /app/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php5_1.dev;
                  proxy_pass http://ci;
          }

          location /product {
                  rewrite /product/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php7_6_yii.dev;
                  proxy_pass http://yii;
          }

          location /phpmyadmin {
                  rewrite /phpmyadmin/?(.*)$ /$1 break;
                  proxy_pass http://lxc_db_server.dev;
          }

          location / {
                  #rewrite /php7/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php7_1.dev;
                  proxy_pass http://laravel;
          }

  }

  server {
         listen 80;

         server_name news.kelompok10.fpsas;

         root /var/www/html;
         index index.html;

         location / {
                  #rewrite /php7/?(.*)$ /$1 break;
                  #proxy_pass http://lxc_php7_3_wp.dev;
                  proxy_pass http://wp;
         }
  }
```

Deploy website menggunakan ansible:

Setting hosts ansible
```bash
nano hosts 
```
![A1](aset/G13.jpg)
    ![A1](aset/G14.jpg)

Daftar lxc pada /etc/hosts di vm
```bash
nano / etc/host 
nano hosts 
```
```bash
127.0.0.1 localhost
127.0.1.1 sas01
127.0.0.1 kelompok10.fpsas news.kelompok10.fpsas

#laravel
10.0.3.101  lxc_php7_1.dev
10.0.3.102  lxc_php7_2_laravel.dev
10.0.3.104  lxc_php7_4_laravel.dev
10.0.3.106  lxc_php7_6_laravel.dev

#yii
10.0.3.102  lxc_php7_2_yii.dev
10.0.3.101  lxc_php7_1_yii.dev
10.0.3.104  lxc_php7_4_yii.dev
10.0.3.105  lxc_php7_5_yii.dev
10.0.3.106  lxc_php7_6_yii.dev

#wp
10.0.3.103  lxc_php7_3_wp.dev
10.0.3.102  lxc_php7_2.dev
10.0.3.104  lxc_php7_4_wp.dev
10.0.3.105  lxc_php7_5_wp.dev

#ci
10.0.3.111  lxc_php5_1.dev
10.0.3.112  lxc_php5_2.dev

#db
10.0.3.200  lxc_db_server.dev
#10.0.3.201  lxc_db_server_coba.dev

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
![A1](aset/G15.jpg)
    ![A1](aset/G16.jpg)

## Setting Ansible Mariadb 
```bash
Nano install-mariadb.yml
```
```bash
- hosts: database
  vars:
    username: 'admin'
    password: '123'
    domain: 'lxc_db_server.dev'
  roles:
    - db
    - pma
```

![A1](aset/G17.jpg)

```bash
mkdir roles 
mkdir roles/db
mkdir roles/db/handlers
```

## Roles db 

```bash
nano roles/db/handlers/main.yml 
```

```bash
---
- name: restart mysql
  become: yes
  become_user: root
  become_method: su
  action: service name=mysql state=restarted 
```
![A1](aset/G18.jpg)

```bash
mkdir roles/db/tasks
nano roles/db/tasks/main.yml
```

```bash
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install mariadb
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
   - python
   - mariadb-server
   - mariadb-client
   - python-mysqldb
   - python-pymysql

- name: Stop MySQL
  service: name=mysqld state=stopped

- name: set environment variables
  shell: systemctl set-environment MYSQLD_OPTS="--skip-grant-tables"

- name: Start MySQL
  service: name=mysqld state=started

- name: sql query
  command:  mysql -u root --execute="UPDATE mysql.user SET authentication_string = PASSWORD('{{ password }}') WHERE User = 'root' AND Host = 'localhost';"

- name: sql query flush
  command:  mysql -u root --execute="FLUSH PRIVILEGES"

- name: Stop MySQL
  service: name=mysqld state=stopped

- name: unset environment variables
  shell: systemctl unset-environment MYSQLD_OPTS

- name: Start MySQL
  service: name=mysqld state=started

- name: Create user for mysql
  command:  mysql -u root --execute="CREATE USER IF NOT EXISTS '{{ username }}'@'localhost' IDENTIFIED BY '{{ password }}';"

- name: GRANT ALL PRIVILEGES to user {{username}}
  command:  mysql -u root --execute="GRANT ALL PRIVILEGES ON * . * TO '{{ username }}'@'localhost';"

- name: Create user for remote mysql
  command:  mysql -u root --execute="CREATE USER IF NOT EXISTS '{{ username }}'@'%' IDENTIFIED BY '{{ password }}';"

- name: GRANT ALL PRIVILEGES to remote user {{username}}
  command:  mysql -u root --execute="GRANT ALL PRIVILEGES ON * . * TO '{{ username }}'@'%';"

- name: sql query flush
  command:  mysql -u root --execute="FLUSH PRIVILEGES"

- name: Create DB Landing
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `landing`;"

- name: Create DB product
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `product`;"

- name: Create DB app
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `app`;"

- name: Create DB news
  command:  mysql -u root --execute="CREATE DATABASE IF NOT EXISTS `news`;"

- name: Copy .my.cnf file with root password credentials
  template:
    src=templates/my.cnf
    dest=/etc/mysql/mariadb.conf.d/50-server.cnf
  notify: restart mysql
```

![A1](aset/G19.jpg)

```bash
mkdir roles/db/templates
nano roles/db/templates/my.cnf
```
```bash
#
# These groups are read by MariaDB server.
# Use it for options that only the server (but not clients) should see
#
# See the examples of server my.cnf files in /usr/share/mysql

# this is read by the standalone daemon and embedded servers
[server]

# this is only for the mysqld standalone daemon
[mysqld]

#
# * Basic Settings
#
user                    = mysql
pid-file                = /run/mysqld/mysqld.pid
socket                  = /run/mysqld/mysqld.sock
port                   = 3306
basedir                 = /usr
datadir                 = /var/lib/mysql
tmpdir                  = /tmp
lc-messages-dir         = /usr/share/mysql
skip-external-locking

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0

#
# * Fine Tuning
#
key_buffer_size        = 16M
max_allowed_packet     = 16M
thread_stack           = 192K
thread_cache_size      = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam_recover_options = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10

#
# * Query Cache Configuration
#
query_cache_limit      = 1M
query_cache_size        = 16M

#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file       = /var/log/mysql/mysql.log
#general_log            = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Enable the slow query log to see queries with especially long duration
#slow_query_log_file    = /var/log/mysql/mariadb-slow.log
#long_query_time        = 10
#log_slow_rate_limit    = 1000
#log_slow_verbosity     = query_plan
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id              = 1
#log_bin                = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size        = 100M
#binlog_do_db           = include_database_name
#binlog_ignore_db       = exclude_database_name

#
# * Security Features
#
# Read the manual, too, if you want chroot!
#chroot = /var/lib/mysql/
#
# For generating SSL certificates you can use for example the GUI tool "tinyca".
#
#ssl-ca = /etc/mysql/cacert.pem
#ssl-cert = /etc/mysql/server-cert.pem
#ssl-key = /etc/mysql/server-key.pem
#
# Accept only connections using the latest and most secure TLS protocol version.
# ..when MariaDB is compiled with OpenSSL:
#ssl-cipher = TLSv1.2
# ..when MariaDB is compiled with YaSSL (default in Debian):
#ssl = on

#
# * Character sets
#
# MySQL/MariaDB default is Latin1, but in Debian we rather default to the full
# utf8 4-byte character set. See also client.cnf
#
character-set-server  = utf8mb4
collation-server      = utf8mb4_general_ci

#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!

#
# * Unix socket authentication plugin is built-in since 10.0.22-6
#
# Needed so the root database user can authenticate without a password but
# only when running as the unix root user.
#
# Also available for other users if required.
# See https://mariadb.com/kb/en/unix_socket-authentication-plugin/

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is only read by MariaDB-10.3 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
[mariadb-10.3]
```

![A1](aset/G20.jpg)

```bash
mkdir roles/pma
mkdir roles/pma/handlers
nano roles/pma/handlers/main.yml 
```
```bash
---
 - name: stop apache2
   become: yes
   become_user: root
   become_method: su
   action: service name=apache2 state=stopped

 - name: restart nginx
   become: yes
   become_user: root
   become_method: su
   action: service name=nginx state=restarted

 - name: restart php
   become: yes
   become_user: root
   become_method: su
   action: service name=php7.2-fpm state=restarted
```
![A1](aset/G21.jpg)

```bash
mkdir roles/pma/tasks
nano roles/pma /tasks/main.yml 
```

```bash
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install nginx phpmyadmin
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - curl
    - nginx
    - nginx-extras
    - php7.3-fpm
    - php-mbstring
    - php-zip
    - php-gd
    - php-json
    - php-curl
    - wget
    - php7.3-xml
    - php7.3-mysqli

- name: wget phpmyadmin
  shell: wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.tar.gz

- name: tar latest.tar.gz
  shell: tar -zxvf phpMyAdmin-5.1.1-all-languages.tar.gz

- name: move phpmyadmin
  shell: sudo mv phpMyAdmin-5.1.1-all-languages /usr/share/phpmyadmin

- name: Create a tmp directory for phpMyAdmin and then change the permission.
  command: mkdir /usr/share/phpmyadmin/tmp

- name: change permission storage
  command: chmod 777 /usr/share/phpmyadmin/tmp

- name: Set the ownership of the phpMyAdmin directory.
  command: chown -R www-data:www-data /usr/share/phpmyadmin

- name: copy config
  template:
    src=templates/conf.inc.php
    dest=/usr/share/phpmyadmin/config.inc.php

- name: enable module php mbstring
  command: phpenmod mbstring
  notify:
    - restart php

- name: Copy pma.local
  template:
    src=templates/pma.local
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Symlink pma.local
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - stop apache2
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1 {{ domain }}"
    state: present
```
![A1](aset/G22.jpg)

```bash
mkdir roles/pma/templates
nano roles/pma/templates/conf.inc.php 
```

```bash
<?php
/**
 * phpMyAdmin sample configuration, you can use it as base for
 * manual configuration. For easier setup you can use setup/
 *
 * All directives are explained in documentation in the doc/ folder
 * or at <https://docs.phpmyadmin.net/>.
 */

declare(strict_types=1);

/**
 * This is needed for cookie based authentication to encrypt password in
 * cookie. Needs to be 32 chars long.
 */
$cfg['blowfish_secret'] = 'KELOMPOK10SISTEMADMINISTRASISERVE'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */

/**
 * Servers configuration
 */
$i = 0;

/**
 * First server
 */
$i++;
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
/* Server parameters */
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;

/**
 * phpMyAdmin configuration storage settings.
 */

/* User used to manipulate with storage */
$cfg['Servers'][$i]['controlhost'] = 'localhost';
$cfg['Servers'][$i]['controlport'] = '3306';
$cfg['Servers'][$i]['controluser'] = '{{username}}';
$cfg['Servers'][$i]['controlpass'] = '{{password}}';

/* Storage database and tables */
$cfg['Servers'][$i]['pmadb'] = 'phpmyadmin';
$cfg['Servers'][$i]['bookmarktable'] = 'pma__bookmark';
$cfg['Servers'][$i]['relation'] = 'pma__relation';
$cfg['Servers'][$i]['table_info'] = 'pma__table_info';
$cfg['Servers'][$i]['table_coords'] = 'pma__table_coords';
$cfg['Servers'][$i]['pdf_pages'] = 'pma__pdf_pages';
$cfg['Servers'][$i]['column_info'] = 'pma__column_info';
$cfg['Servers'][$i]['history'] = 'pma__history';
$cfg['Servers'][$i]['table_uiprefs'] = 'pma__table_uiprefs';
$cfg['Servers'][$i]['tracking'] = 'pma__tracking';
$cfg['Servers'][$i]['userconfig'] = 'pma__userconfig';
$cfg['Servers'][$i]['recent'] = 'pma__recent';
$cfg['Servers'][$i]['favorite'] = 'pma__favorite';
$cfg['Servers'][$i]['users'] = 'pma__users';
$cfg['Servers'][$i]['usergroups'] = 'pma__usergroups';
$cfg['Servers'][$i]['navigationhiding'] = 'pma__navigationhiding';
$cfg['Servers'][$i]['savedsearches'] = 'pma__savedsearches';
$cfg['Servers'][$i]['central_columns'] = 'pma__central_columns';
$cfg['Servers'][$i]['designer_settings'] = 'pma__designer_settings';
$cfg['Servers'][$i]['export_templates'] = 'pma__export_templates';

/**
 * End of servers configuration
 */

/**
 * Directories for saving/loading files from server
 */
$cfg['UploadDir'] = '';
$cfg['SaveDir'] = '';

/**
 * Whether to display icons or text or both icons and text in table row
 * action segment. Value can be either of 'icons', 'text' or 'both'.
 * default = 'both'
 */
//$cfg['RowActionType'] = 'icons';

/**
 * Defines whether a user should be displayed a "show all (records)"
 * button in browse mode or not.
 * default = false
 */
//$cfg['ShowAll'] = true;

/**
 * Number of rows displayed when browsing a result set. If the result
 * set contains more rows, "Previous" and "Next".
 * Possible values: 25, 50, 100, 250, 500
 * default = 25
 */
//$cfg['MaxRows'] = 50;

/**
 * Disallow editing of binary fields
 * valid values are:
 *   false    allow editing
 *   'blob'   allow editing except for BLOB fields
 *   'noblob' disallow editing except for BLOB fields
 *   'all'    disallow editing
 * default = 'blob'
 */
//$cfg['ProtectBinary'] = false;

/**
 * Default language to use, if not browser-defined or user-defined
 * (you find all languages in the locale folder)
 * uncomment the desired line:
 * default = 'en'
 */
//$cfg['DefaultLang'] = 'en';
//$cfg['DefaultLang'] = 'de';

/**
 * How many columns should be used for table display of a database?
 * (a value larger than 1 results in some information being hidden)
 * default = 1
 */
//$cfg['PropertiesNumColumns'] = 2;

/**
 * Set to true if you want DB-based query history.If false, this utilizes
 * JS-routines to display query history (lost by window close)
 *
 * This requires configuration storage enabled, see above.
 * default = false
 */
//$cfg['QueryHistoryDB'] = true;

/**
 * When using DB-based query history, how many entries should be kept?
 * default = 25
 */
//$cfg['QueryHistoryMax'] = 100;

/**
 * Whether or not to query the user before sending the error report to
 * the phpMyAdmin team when a JavaScript error occurs
 *
 * Available options
 * ('ask' | 'always' | 'never')
 * default = 'ask'
 */
//$cfg['SendErrorReports'] = 'always';

/**
 * You can find more configuration options in the documentation
 * in the doc/ folder or at <https://docs.phpmyadmin.net/>.
 */
```

![A1](aset/G23.jpg)

```bash
nano roles/pma/templates/pma.local 
```

```bash
server {
    listen 80;

    server_name {{servername}};

    root /usr/share/phpmyadmin;

    index index.php;

    location / {

         try_files $uri $uri/ @phpmyadmin;

    }

    location @phpmyadmin {
            fastcgi_pass unix:/run/php/php7.3-fpm.sock;   #Sesuaikan dengan versi PHP
            fastcgi_param SCRIPT_FILENAME /usr/share/phpmyadmin/index.php;
            include /etc/nginx/fastcgi_params;
            fastcgi_param SCRIPT_NAME /index.php;

    }

    location ~ \.php$ {

            fastcgi_pass unix:/run/php/php7.3-fpm.sock;  #Sesuaikan dengan versi PHP
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /usr/share/phpmyadmin$fastcgi_script_name;
            include fastcgi_params;

    }
}
```
![A1](aset/G24.jpg)

Run ansible install-mariadb.yml
```bash
ansible-playbook -i hosts install-mariadb.yml -k 
```
![A1](aset/G25.jpg)

hasilnya seperti ini:
![A1](aset/G26.jpg)
    ![A1](aset/G27.jpg)
         ![A1](aset/G28.jpg)

## Setting ansible Laravel 

```bash
nano install-laravel.yml
```
```bash
- hosts: laravel
  vars:
    domain: 'lxc_php7_1.dev'
  roles:
    - laravel

- hosts: lxc_php7_2_laravel
  vars:
    domain: 'lxc_php7_2_laravel.dev'
  roles:
    - laravel

- hosts: lxc_php7_4_laravel
  vars:
    domain: 'lxc_php7_4_laravel.dev'
  roles:
    - laravel

- hosts: lxc_php7_6_laravel
  vars:
    domain: 'lxc_php7_6_laravel.dev'
  roles:
    - laravel
```
![A1](aset/G29.jpg)

```bash
mkdir roles/laravel
mkdir roles/laravel/handlers
nano roles/laravel/handlers/main.yml 
```
```bash
---
- name: stop apache2
  become: yes
  become_user: root
  become_method: su
  action: service name=apache2 state=stopped

- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php7.2-fpm state=restarted
```
![A1](aset/G30.jpg)

```bash
mkdir roles/laravel/tasks
nano roles/laravel/tasks/main.yml 
```

```bash
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - curl
    - python-apt
    - software-properties-common

- name: Add Php Repository 7.4
  apt_repository:
    repo: "ppa:ondrej/php"
    state: present
    filename: php.list
    update_cache: true

- name: install nginx php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - git
    - nginx
    - nginx-extras
    - php7.4
    - php7.4-fpm
    - php7.4-curl
    - php7.4-mbstring
    - php7.4-xml
    - php7.4-gd
    - php7.4-opcache
    - php7.4-zip
    - php7.4-json
    - php7.4-cli

- name: Download and install Composer
  shell: curl -sS https://getcomposer.org/installer | php
  args:
    chdir: /usr/src/
    creates: /usr/local/bin/composer
    warn: false
  become: yes

- name: Add Composer to global path
  copy:
    dest: /usr/local/bin/composer
    group: root
    mode: '0755'
    owner: root
    src: /usr/src/composer.phar
    remote_src: yes
  become: yes

- name: Composer create project
  composer:
    command: create-project
    arguments: laravel/laravel landing
    working_dir: /var/www/html
    prefer_dist: yes
  environment:
    COMPOSER_NO_INTERACTION: "1"

- name: set APP_URL
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^APP_URL='
    line=APP_URL=http://kelompok10.fpsas

- name: set DB_HOST
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^DB_HOST='
    line=DB_HOST=10.0.3.200

- name: set DB_DATABASE
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^DB_DATABASE='
    line=DB_DATABASE=landing

- name: set DB_USERNAME
  lineinfile:
     dest=/var/www/html/landing/.env
     regexp='^DB_USERNAME='
     line=DB_USERNAME=admin

- name: set DB_PASSWORD
  lineinfile:
    dest=/var/www/html/landing/.env
    regexp='^DB_PASSWORD='
    line=DB_PASSWORD=123
    

- name: change permission storage
  command: chmod -R 777 /var/www/html/landing/storage

- name: Copy landing.conf
  template:
    src=templates/landing
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
      servername: '{{ domain }}'

- name: www.conf
  lineinfile:
     dest: /etc/php/7.4/fpm/pool.d/www.conf
     regexp: '^listen = /run/php/php7.4-fpm.sock'
     line: listen = 127.0.0.1:9001
     state: present

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: Symlink landing.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1   {{ domain }}"
    state: present
```

![A1](aset/G31.jpg)

```bash
mkdir roles/laravel/templates
nano roles/laravel/templates/landing 
```
```bash
server {

    listen 80;

    server_name {{ domain }};

    root /var/www/html/landing/public;
    index index.php index.html index.htm;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        #fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_pass 127.0.0.1:9001;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```
![A1](aset/G32.jpg)

Run ansible install-laravel.yml
```bash
ansible-playbook -i hosts install-laravel.yml -k
```

Hasil nya : 
![A1](aset/G33.jpg)

## Setting ansible Codeigniter

```bash
nano install-ci.yml 
```

```bash
- hosts: ci
  vars:
    git_url: 'https://github.com/aldonesia/sas-ci'
    destdir: '/var/www/html/ci'
    domain: 'lxc_php5_1.dev'
  roles:
    - ci

- hosts: lxc_php5_2
  vars:
    git_url: 'https://github.com/aldonesia/sas-ci'
    destdir: '/var/www/html/ci'
    domain: 'lxc_php5_2.dev'
  roles:
    - ci
```
![A1](aset/G34.jpg)


```bash
mkdir roles/ci
mkdir roles/ci/handlers
nano roles/ci/handlers/main.yml 
```

```bash
---
- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php5.6-fpm state=restarted
```
![A1](aset/G35.jpg)

```bash
mkdir roles/ci/tasks
nano roles/ci/tasks/main.yml
```

```bash
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php5
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - wget
    - curl
    - python-apt
    - software-properties-common
    - git

- name: Add key
  apt_key:
    url: https://packages.sury.org/php/apt.gpg
    state: present

- name: Add Php Repository
  apt_repository:
      repo: "deb https://packages.sury.org/php/ buster main"
      state: present
      filename: php.list
      update_cache: true

- name: install nginx php5
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - nginx
    - nginx-extras
    - php5.6
    - php5.6-fpm
    - php5.6-common
    - php5.6-cli
    - php5.6-curl
    - php5.6-mbstring
    - php5.6-mysqlnd
    - php5.6-xml

- name: Git clone repo sas-ci
  become: yes
  git:
    repo: '{{ git_url }}'
    dest: "{{ destdir }}"

- name: Copy app.conf
  template:
    src=templates/app.conf
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: Symlink app.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1 {{ domain }}"
    state: present
```
![A1](aset/G36.jpg)

```bash
mkdir roles/ci/templates
nano roles/ci/templates/app.conf 
```
```bash
server {
  listen 80;
  server_name {{servername}};
  root {{ destdir }};
  index index.php;
  location / {
     try_files $uri $uri/ /index.php?$query_string;
  }
  location ~ \.php$ {
     fastcgi_pass unix:/run/php/php5.6-fpm.sock;  #Sesuaikan dengan versi PHP
     fastcgi_index index.php;
     fastcgi_param SCRIPT_FILENAME {{ destdir }}$fastcgi_script_name;
     include fastcgi_params;
  }
}
```
![A1](aset/G37.jpg)

Run ansible install-ci.yml
```bash
ansible-playbook -i hosts install-ci.yml -k
```
![A1](aset/G38.jpg)

## Setting ansible Yii 2.0

```bash
nano install-yii.yml 
```

```bash
- hosts: lxc_php7_2_yii
  vars:
    domain: 'lxc_php7_2_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_1_yii
  vars:
    domain: 'lxc_php7_1_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_4_yii
  vars:
    domain: 'lxc_php7_4_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_5_yii
  vars:
    domain: 'lxc_php7_5_yii.dev'
  roles:
    - yii

- hosts: lxc_php7_6_yii
  vars:
    domain: 'lxc_php7_6_yii.dev'
  roles:
    - yii
```
![A1](aset/G39.jpg)

```bash
mkdir roles/yii
mkdir roles/yii/handlers
nano roles/yii/handlers/main.yml 
```

```bash
---
- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php5.6-fpm state=restarted
```
![A1](aset/G40.jpg)

```bash
mkdir roles/yii/tasks
nano roles/yii/tasks/main.yml 
```

```bash
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - curl
    - python-apt
    - software-properties-common

- name: Add Php Repository 7.4
  apt_repository:
    repo: "ppa:ondrej/php"
    state: present
    filename: php.list
    update_cache: true

- name: install nginx php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
   - git
   - nginx
   - nginx-extras
   - php7.4
   - php7.4-fpm
   - php7.4-curl
   - php7.4-mbstring
   - php7.4-xml
   - php7.4-gd
   - php7.4-opcache
   - php7.4-zip
   - php7.4-json
   - php7.4-cli
   - php7.4-intl

- name: Download and install Composer
  shell: curl -sS https://getcomposer.org/installer | php
  args:
    chdir: /usr/src/
    creates: /usr/local/bin/composer
    warn: false
  become: yes

- name: Add Composer to global path
  copy:
    dest: /usr/local/bin/composer
    group: root
    mode: '0755'
    owner: root
    src: /usr/src/composer.phar
    remote_src: yes
  become: yes

- name: Composer create project
  composer:
    command: create-project
    arguments: yiisoft/yii2-app-basic product
    working_dir: /var/www/html
    prefer_dist: yes
 environment:
    COMPOSER_NO_INTERACTION: "1"

- name: set permission
  shell: chown -R www-data:www-data /var/www/html/product

- name: setting db
  template:
    src=templates/db.php
    dest=/var/www/html/product/config/db.php

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: www.conf
  lineinfile:
    dest: /etc/php/7.4/fpm/pool.d/www.conf
    regexp: '^listen = /run/php/php7.4-fpm.sock'
    line: listen = 127.0.0.1:9001
    state: present

- name: Copy product.conf
  template:
    src=templates/product.config
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Symlink product.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{domain}}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1   {{ domain }}"
    state: present
```
![A1](aset/G41.jpg)

```bash
mkdir roles/yii/templates
nano roles/yii/templates/db.php 
```
```bash
<?php

return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=127.0.3.200:3306;dbname=product',
    'username' => 'admin',
    'password' => '123',
    'charset' => 'utf8',
];
```
![A1](aset/G42.jpg)

```bash
nano roles/yii/templates/product.config 
```

```bash
server {
        listen 80;
        listen [::]:80;

        server_name {{servername}};

        root /var/www/html/product/web;
        index index.php;

        charset utf-8;

        location / {
                index  index.html index.php;
                try_files $uri $uri/ /index.php?$args;
        }

        location ~ ^/(protected|framework|themes/\w+/views) {
                deny  all;
        }

        # pass the PHP scripts to FastCGI server listening on UNIX socket
        location ~ \.php {
                fastcgi_split_path_info  ^(.+\.php)(.*)$;
        #let yii catch the calls to unexising PHP files
                set $fsn /index.php;
                if (-f $document_root$fastcgi_script_name){
                        set $fsn $fastcgi_script_name;
                }
                fastcgi_pass   127.0.0.1:9001;
                include fastcgi_params;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fsn;

       #PATH_INFO and PATH_TRANSLATED can be omitted, but RFC 3875 specifies them for CGI
                fastcgi_param  PATH_INFO        $fastcgi_path_info;
                fastcgi_param  PATH_TRANSLATED  $document_root$fsn;
        }

        # prevent nginx from serving dotfiles (.htaccess, .svn, .git, etc.)
        location ~ /\. {
                deny all;
                access_log off;
                log_not_found off;
        }
}
```
![A1](aset/G43.jpg)

Run ansible install-yii.yml
```bash
ansible-playbook -i hosts install-yii.yml -k
```
![A1](aset/G44.jpg)

## Setting ansible Wordpress

```bash
nano install-wp.yml 
```

```bash
- hosts: lxc_php7_4_wp
  vars:
    username: 'admin'
    password: '123'
    domain: lxc_php7_4_wp.dev
  roles:
    - wp

- hosts: lxc_php7_3_wp
  vars:
    username: 'admin'
    password: '123'
    domain: lxc_php7_3_wp.dev
  roles:
    - wp

- hosts: lxc_php7_5_wp
  vars:
    username: 'admin'
    password: '123'
    domain: lxc_php7_5_wp.dev
  roles:
    - wp

- hosts: wp
  vars:
username: 'admin'
    password: '123'
    domain: lxc_php7_2.dev
  roles:
    - wp
```
![A1](aset/G45.jpg)


```bash
mkdir roles/wp
mkdir roles/wp/handlers
nano roles/wp/handlers/main.yml
```

```bash
---
- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php7.4-fpm state=restarted
```

![A1](aset/G46.jpg)

```bash
mkdir roles/wp/tasks
nano roles/wp/tasks/main.yml 
```

```bash
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - curl
    - python-apt
    - software-properties-common

- name: Add Php Repository 7.4
  apt_repository:
    repo: "ppa:ondrej/php"
    state: present
    filename: php.list
    update_cache: true

- name: install nginx php7.4
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - nginx
    - nginx-extras
    - curl
    - wget
    - php7.4
    - php7.4-fpm
    - php7.4-curl
    - php7.4-xml
    - php7.4-gd
    - php7.4-opcache
    - php7.4-mbstring
    - php7.4-zip
    - php7.4-json
    - php7.4-cli
    - php7.4-mysqlnd
    - php7.4-xmlrpc

- name: wget wordpress
  shell: wget -c http://wordpress.org/latest.tar.gz

- name: tar latest.tar.gz
  shell: tar -xvzf latest.tar.gz

- name: copy folder wordpress
  shell: cp -R wordpress /var/www/html/blog

- name: chmod
  become: yes
  become_user: root
  become_method: su
  command: chmod 775 -R /var/www/html/blog/

- name: copy .wp-config.conf
  template:
    src=templates/wp.conf
    dest=/var/www/html/blog/wp-config.php

- name: copy wp.local
  template:
    src=templates/wp.local
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: Symlink sites-available wordpress
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1   {{ domain }}"
    state: present

- name: enable module php mbstring
  command: phpenmod mbstring
  notify:
    - restart php
```

![A1](aset/G47.jpg)

```bash
mkdir roles/db/templates
nano roles/db/templates/wp.conf wp.local
```

```bash
<?php
/**
* The base configuration for WordPress
*
* The wp-config.php creation script uses this file during the installation.
* You don't have to use the web site, you can copy this file to "wp-config.php"
* and fill in the values.
*
* This file contains the following configurations:
*
* * MySQL settings
* * Secret keys
* * Database table prefix
* * ABSPATH
*
* @link https://wordpress.org/support/article/editing-wp-config-php/
*
* @package WordPress
*/

define( 'WP_HOME', 'http://news.kelompok10.fpsas' );
define( 'WP_SITEURL', 'http://news.kelompok10.fpsas' );

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'news' );

/** MySQL database username */
define( 'DB_USER', 'admin' );

/** MySQL database password */
define( 'DB_PASSWORD', '123' );

/** MySQL hostname */
define( 'DB_HOST', '10.0.3.200:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
* Authentication unique keys and salts.
*
* Change these to different unique phrases! You can generate these using
* the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
*
* You can change these at any point in time to invalidate all existing cookies.
* This will force all users to have to log in again.
*
* @since 2.6.0
*/
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
* WordPress database table prefix.
*
* You can have multiple installations in one database if you give each
* a unique prefix. Only numbers, letters, and underscores please!
*/
$table_prefix = 'wp_';

/**
* For developers: WordPress debugging mode.
*
* Change this to true to enable the display of notices during development.
* It is strongly recommended that plugin and theme developers use WP_DEBUG
* in their development environments.
*
* For information on other constants that can be used for debugging,
* visit the documentation.
*
* @link https://wordpress.org/support/article/debugging-in-wordpress/
*/
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
      define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```
![A1](aset/G48.jpg)

```bash
nano roles/db/templates/wp.local 
```

```bash
server {

    listen 80;

    server_name {{ domain }};

    root /var/www/html/blog;
    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
![A1](aset/G49.jpg)

Run ansible install-wp.yml
```bash
ansible-playbook -i hosts install-wp.yml -k
```
![A1](aset/G50.jpg)

## Jmeter
### Load testing 

## Kelompok10.fpsas
- 50

![A1](aset/G51.jpg)
![A1](aset/G52.jpg)
![A1](aset/G53.jpg)

- 150

![A1](aset/G54.jpg)
![A1](aset/G55.jpg)
![A1](aset/G56.jpg)

- 300
![A1](aset/G57.jpg)
![A1](aset/G58.jpg)
![A1](aset/G59.jpg)

- 500

![A1](aset/G60.jpg)
![A1](aset/G61.jpg)
![A1](aset/G62.jpg)

## news.kelompok10.fpsas
- 50

![A1](aset/G63.jpg)
![A1](aset/G64.jpg)
![A1](aset/G65.jpg)

- 150

![A1](aset/G66.jpg)
![A1](aset/G67.jpg)
![A1](aset/G68.jpg)

- 300

![A1](aset/G69.jpg)
![A1](aset/G70.jpg)
![A1](aset/G71.jpg)

- 500

![A1](aset/G72.jpg)
![A1](aset/G73.jpg)
![A1](aset/G74.jpg)

## Analisis
Nilai rata – rata Throughput 

- 50 user

kelompok10.fpsas/ : 19.3/s\
kelompok10.fpsas/app : 21.4/s\
kelompok10.fpsas/product : 20.9/s\
news.kelompok10.fpsas/ : 7.6/s

- 150 user

kelompok10.fpsas/ : 18.3/s\
kelompok10.fpsas/app : 19.6/s\
kelompok10.fpsas/product : 19.6/s\
news.kelompok10.fpsas/ : 7.6/s

- 300 user

kelompok10.fpsas/ : 20.5/s\
kelompok10.fpsas/app : 22.1/s\
kelompok10.fpsas/product : 22,1/s\
news.kelompok10.fpsas/ : 7.8/s

- 500 user

kelompok10.fpsas/ : 25.7/s\
kelompok10.fpsas/app : 27.1/s\
kelompok10.fpsas/product : 27.3/s\
news.kelompok10.fpsas/ : 9.4/s

### Nilai rata - rata jumlah user setiap detik 

- 50 user

kelompok10.fpsas/ : 1391\
kelompok10.fpsas/app : 9\
kelompok10.fpsas/product : 44\
news.kelompok10.fpsas/ : 4104\

- 150 user

kelompok10.fpsas/ : 4262\
kelompok10.fpsas/app : 9\
kelompok10.fpsas/product : 52\
news.kelompok10.fpsas/ : 11833

- 300 user

kelompok10.fpsas/ : 7509\
kelompok10.fpsas/app : 9\
kelompok10.fpsas/product : 46\
news.kelompok10.fpsas/ : 21954

- 500 user

kelompok10.fpsas/ : 7886\
kelompok10.fpsas/app : 78\
kelompok10.fpsas/product : 97\
news.kelompok10.fpsas/ : 23895
