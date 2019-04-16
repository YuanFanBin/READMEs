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
● iptables.service - IPv4 firewall with iptables
   Loaded: loaded (/usr/lib/systemd/system/iptables.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
$ systemctl stop firewalld.service      # CLOSE
$ systemctl disable firewalld.service   # DISABLE
```

ref: http://www.cnblogs.com/glxsc/p/5221001.html


## 安装CentOS（模板机）

1. 下载CentOS：http://isoredirect.centos.org/centos/7/isos/x86_64/
2. [Centos7安装完毕后无法联网问题的解决方法](https://www.36nu.com/post/234)

   14: curl#6 - "Could not resolve host: mirrorlist.centos.org; 未知的错误"
   
```sh
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
# 将最后一行的ONBOOT=no改为ONBOOT=yes,保存并退出，重启一下network：
service network restart
```

3. 更新yum源（[CentOS更新yum源为阿里云](https://segmentfault.com/a/1190000016151886)）

```sh
yum install wget -y

# 第一步：备份/etc/yum.repos.d/CentOS-Base.repo；
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 第二步：进入yum源配置文件所在文件夹；
cd /etc/yum.repos.d/

# 第三步：下载aliyun的yum源配置文件，放入/etc/yum.repos.d/下，并更改名称为CentOS-Base.repo
wget http://mirrors.aliyun.com/repo/Centos-7.repo
mv Centos-7.repo CentOS-Base.repo

# 第四步：运行yum makecache生成缓存
yum makecache

# 第五步：更新yum
yum -y update
```

4. https://github.com/YuanFanBin/READMEs/blob/master/Arch.md

5. 
