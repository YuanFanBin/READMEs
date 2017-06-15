# CentOS7

## install mysql

### 1. Download and add the repository, then update.

```sh
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
$ sudo yum update
```

### 2. Install Mysql

```sh
$ sudo yum install mysql-server
$ sudo systemctl start mysqld
$ mysql -u root -p
```

ref: https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-centos-7

## close firewall

```sh
$ sudo yum install iptables-services
$ systemctl status iptables.service
¡ñ iptables.service - IPv4 firewall with iptables
   Loaded: loaded (/usr/lib/systemd/system/iptables.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
$ systemctl stop firewalld.service      # CLOSE
$ systemctl disable firewalld.service   # DISABLE
```

ref: http://www.cnblogs.com/glxsc/p/5221001.html
