## 使用Homestead搭建Laravel开发环境

### 安装VirtualBox和Vagrant

虽然官方推荐了很多虚拟机软件，但这里推荐VirtualBox。

### 安装Homestead Vagrant Box

安装VirtualBox和Vagrant后，打开命令行通过`$ vagrant box add laravel/homestead`命令安装Homestead Vagrant Box。这个过程需要下载box，正常情况下的输出如下：

```
C:\Users\Administrator>vagrant box add laravel/homestead
==> box: Loading metadata for box 'laravel/homestead'
    box: URL: https://vagrantcloud.com/laravel/homestead
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) hyperv
2) parallels
3) virtualbox
4) vmware_desktop

Enter your choice: 3
==> box: Adding box 'laravel/homestead' (v9.5.1) for provider: virtualbox
    box: Downloading: https://vagrantcloud.com/laravel/boxes/homestead/versions/9.5.1/providers/virtualbox.box
Download redirected to host: vagrantcloud-files-production.s3.amazonaws.com
    box:
    box: Calculating and comparing box checksum...
==> box: Successfully added box 'laravel/homestead' (v9.5.1) for 'virtualbox'!
```

我用的是**Windows 10**操作系统，下载下来的box默认放置在**C:\Users\Administrator\.vagrant.d\boxes**目录。如果下载失败可以尝试使用下载器下载，下载链接从命令行的输出中可以找到。下载完成后使用`vagrant box add`命令添加。可以使用`$ vagrant box list`命令查看已经添加的box。

```
$ vagrant box list
laravel/homestead (virtualbox, 9.5.1)
```

### 下载Homestead项目代码

首先下载并安装git ，然后克隆代码。安装git是为了后续操作的方便。通过`$ git clone https://github.com/laravel/homestead.git`命令克隆最新的代码。克隆完成后，使用git bash命令行窗口，先执行`git checkout release`切换到稳定版，克隆时默认是master分支。

然后，在Homestead根目录中使用`$ bash init.sh`命令来进行初始化，创建Homestead.yaml等配置文件。在Windows 操作系统里，可也以通过执行目录下的**init.bat**文件来进行初始化。

### 配置Homestead.yaml文件

```yaml
folders:
    - map: ~/code/project # 项目在电脑中的目录
      to: /home/vagrant/project # 项目在虚拟机中的目录

sites:
    - map: homestead.test # 映射的域名
      to: /home/vagrant/code/public # 映射的具体路径
```

Homestead提供了很多常用的软件，在项目中可以根据需要进行合理配置。

### 启动Homestead

在完成上面的安装步骤后，就可以使用`vagrant up`命令启动虚拟机。这里如果没有配置过 ssh，会碰到如下的报错：

```
$ vagrant up
Check your Homestead.yaml (or Homestead.json) file, the path to your private key does not exist.
```

如果安装了git，可以使用`ssh-keygen`命令创建ssh key。这里直接默认回车到底就行。

```
Administrator@DESKTOP-P1770NP MINGW64 /e/WorkspacePHP/Homestead (release)
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Created directory '/c/Users/Administrator/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:lTmlCXbYyR7Sjp3Ac3ShHi93fkbeJhjzzXzeligCLiw Administrator@DESKTOP-P1770NP
The key's randomart image is:
+---[RSA 3072]----+
|       .o*o.+.   |
|       .*oBO     |
|         XXo     |
|        .o=+     |
|        S o = . .|
|       .   o B *.|
|    . . .   . = %|
|   E o . . . . B+|
|    . .   . .  .o|
+----[SHA256]-----+
```

启动完成后，虚拟机内的Nginx服务器就已经运行了，就可以通过配置的IP或者域名访问项目了。

### Homestead 端口转发

执行`$ vagrant up`启动虚拟机的时候会输出一些配置信息。

```
$ vagrant up
Bringing machine 'homestead' up with 'virtualbox' provider...
==> homestead: Checking if box 'laravel/homestead' version '9.5.1' is up to date...
==> homestead: Clearing any previously set forwarded ports...
==> homestead: Fixed port collision for 3306 => 33060. Now on port 2200.
==> homestead: Clearing any previously set network interfaces...
==> homestead: Preparing network interfaces based on configuration...
    homestead: Adapter 1: nat
    homestead: Adapter 2: hostonly
==> homestead: Forwarding ports...
    homestead: 80 (guest) => 8000 (host) (adapter 1)
    homestead: 443 (guest) => 44300 (host) (adapter 1)
    homestead: 3306 (guest) => 2200 (host) (adapter 1)
    homestead: 4040 (guest) => 4040 (host) (adapter 1)
    homestead: 5432 (guest) => 54320 (host) (adapter 1)
    homestead: 8025 (guest) => 8025 (host) (adapter 1)
    homestead: 9600 (guest) => 9600 (host) (adapter 1)
    homestead: 27017 (guest) => 27017 (host) (adapter 1)
    homestead: 22 (guest) => 2222 (host) (adapter 1)
==> homestead: Running 'pre-boot' VM customizations...
==> homestead: Booting VM...
==> homestead: Waiting for machine to boot. This may take a few minutes...
    homestead: SSH address: 127.0.0.1:2222
    homestead: SSH username: vagrant
    homestead: SSH auth method: private key
==> homestead: Machine booted and ready!
==> homestead: Checking for guest additions in VM...
    homestead: The guest additions on this VM do not match the installed version of
    homestead: VirtualBox! In most cases this is fine, but in rare cases it can
    homestead: prevent things such as shared folders from working properly. If you see
    homestead: shared folder errors, please make sure the guest additions within the
    homestead: virtual machine match the version of VirtualBox you have installed on
    homestead: your host and reload your VM.
    homestead:
    homestead: Guest Additions Version: 6.0.0 r127566
    homestead: VirtualBox Version: 6.1
==> homestead: Setting hostname...
==> homestead: Configuring and enabling network interfaces...
==> homestead: Mounting shared folders...
    homestead: /vagrant => E:/WorkspacePHP/Homestead
    homestead: /home/vagrant/code => E:/WorkspacePHP/KTCommunity
==> homestead: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> homestead: flag to force provisioning. Provisioners marked to run always will still run.
```

Homestead默认配置mysql的3306端口应该被转发到33060端口，但是可以看到这里应该是由于什么原因，被转发到2200端口了，所以从外部使用Navicate等工具连接虚拟机的mysql时，端口号就要填2200而不是默认配置的33060。如果端口访问不通可以把默认的和配置的端口都尝试一下。

### 使用命令行进入虚拟机

虚拟机启动完成后，可以使用`$ vagrant ssh`登入虚拟机。

```
$ vagrant ssh
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-96-generic x86_64)

Thanks for using
_                               _                 _
| |                             | |               | |
| |__   ___  _ __ ___   ___  ___| |_ ___  __ _  __| |
| '_ \ / _ \| '_ ` _ \ / _ \/ __| __/ _ \/ _` |/ _` |
| | | | (_) | | | | | |  __/\__ \ ||  __/ (_| | (_| |
|_| |_|\___/|_| |_| |_|\___||___/\__\___|\__,_|\__,_|

* Homestead v10.8.0
* Settler v9.5.1 (Virtualbox, Parallels, Hyper-V, VMware)

279 packages can be updated.
32 updates are security updates.
```

这里**~**目录下**code**文件夹就是上面配置的同步文件夹，会同步外部系统的代码。

```
vagrant@homestead:~$ ll
total 76
drwxr-xr-x 11 vagrant vagrant 4096 May 18 09:59 ./
drwxr-xr-x  3 root    root    4096 Apr 28 01:42 ../
-rw-r--r--  1 vagrant vagrant 8100 May 18 09:59 .bash_aliases
-rw-r--r--  1 vagrant vagrant  220 Apr 28 01:42 .bash_logout
-rw-r--r--  1 vagrant vagrant 3771 Apr 28 01:42 .bashrc
drwx------  3 vagrant vagrant 4096 Apr 28 02:03 .cache/
drwxrwxrwx  1 vagrant vagrant 4096 Mar 15 12:33 code/
drwxrwxr-x  4 vagrant vagrant 4096 Apr 28 02:04 .composer/
drwxr-xr-x  5 vagrant vagrant 4096 Apr 28 01:59 .config/
drwx------  3 vagrant vagrant 4096 Apr 28 01:43 .gnupg/
drwxr-xr-x  2 vagrant vagrant 4096 May 18 10:01 .homestead-features/
drwxr-xr-x  3 vagrant vagrant 4096 Apr 28 02:03 .local/
-rw-r--r--  1 vagrant vagrant   67 Apr 28 02:00 .my.cnf
drwxr-xr-x  4 vagrant vagrant 4096 Apr 28 01:59 .npm/
-rw-r--r--  1 vagrant vagrant  856 Apr 28 02:04 .profile
drwx------  2 vagrant vagrant 4096 May 18 09:59 .ssh/
-rw-r--r--  1 vagrant vagrant    0 Apr 28 01:43 .sudo_as_admin_successful
-rw-r--r--  1 vagrant vagrant    5 Apr 28 01:43 .vbox_version
-rw-r--r--  1 vagrant vagrant  297 Apr 28 02:03 .wget-hsts
```

### Homestead上不同的PHP版本

查看当前正在使用的PHP版本，默认是最高版本。

```
vagrant@homestead:~$ php -v
PHP 7.4.5 (cli) (built: Apr 19 2020 07:36:30) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.5, Copyright (c), by Zend Technologies
```

使用`$ update-alternatives --display php`命令可以查看所有PHP版本和当前正在使用的版本。

```
vagrant@homestead:~$ update-alternatives --display php
php - manual mode
  link best version is /usr/bin/php7.4
  link currently points to /usr/bin/php7.4
  link php is /usr/bin/php
  slave php.1.gz is /usr/share/man/man1/php.1.gz
/usr/bin/php5.6 - priority 56
  slave php.1.gz: /usr/share/man/man1/php5.6.1.gz
/usr/bin/php7.0 - priority 70
  slave php.1.gz: /usr/share/man/man1/php7.0.1.gz
/usr/bin/php7.1 - priority 71
  slave php.1.gz: /usr/share/man/man1/php7.1.1.gz
/usr/bin/php7.2 - priority 72
  slave php.1.gz: /usr/share/man/man1/php7.2.1.gz
/usr/bin/php7.3 - priority 73
  slave php.1.gz: /usr/share/man/man1/php7.3.1.gz
/usr/bin/php7.4 - priority 74
  slave php.1.gz: /usr/share/man/man1/php7.4.1.gz
```

Homestead提供了切换PHP版本的命令。这里切换到PHP7.2版本

```
vagrant@homestead:~$ php72
update-alternatives: using /usr/bin/php7.2 to provide /usr/bin/php (php) in manual mode
update-alternatives: using /usr/bin/php-config7.2 to provide /usr/bin/php-config (php-config) in manual mode
update-alternatives: using /usr/bin/phpize7.2 to provide /usr/bin/phpize (phpize) in manual mode
vagrant@homestead:~/code$ update-alternatives --display php
php - manual mode
  link best version is /usr/bin/php7.4
  link currently points to /usr/bin/php7.2
  link php is /usr/bin/php
  slave php.1.gz is /usr/share/man/man1/php.1.gz
/usr/bin/php5.6 - priority 56
  slave php.1.gz: /usr/share/man/man1/php5.6.1.gz
/usr/bin/php7.0 - priority 70
  slave php.1.gz: /usr/share/man/man1/php7.0.1.gz
/usr/bin/php7.1 - priority 71
  slave php.1.gz: /usr/share/man/man1/php7.1.1.gz
/usr/bin/php7.2 - priority 72
  slave php.1.gz: /usr/share/man/man1/php7.2.1.gz
/usr/bin/php7.3 - priority 73
  slave php.1.gz: /usr/share/man/man1/php7.3.1.gz
/usr/bin/php7.4 - priority 74
  slave php.1.gz: /usr/share/man/man1/php7.4.1.gz
vagrant@homestead:~/code$ php -v
PHP 7.2.30-1+ubuntu18.04.1+deb.sury.org+1 (cli) (built: Apr 19 2020 07:50:50) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.30-1+ubuntu18.04.1+deb.sury.org+1, Copyright (c) 1999-2018, by Zend Technologies
```

也可以通过`$ update-alternatives --config php`命令切换PHP版本。执行后，会列出当前PHP所有版本和编号。输入编号，切换到指定的版本。

### 虚拟机访问不通的问题

如果出现启动虚拟机后访问不通192.168.10.10的问题，可以尝试使用`$ vagrant provision`命令重新应用更改vagrant配置，然后使用`$ vagrant reload`命令重启虚拟机。 

如果修改了Homestead的配置，也需要使用上面的命令重新应用更改vagrant配置，并重启虚拟机。

### 关闭虚拟机

使用`$ vagrant halt`命令关闭虚拟机。

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [Laravel 6 中文文档 Homestead](https://learnku.com/docs/laravel/6.x/homestead/5127) |
| [Laravel 7 中文文档 Homestead](https://learnku.com/docs/laravel/7.x/homestead/7450) |
| [homestead.test 拒绝了我们的连接请求](https://learnku.com/laravel/t/7772/homesteadtest-refused-our-connection-request) |