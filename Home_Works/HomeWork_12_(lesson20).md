<div align="center"><h2> Отчет о выполнении домашнего задания по теме: "Секционирование " </h2></div>

***

> ### Секционировать большую таблицу из демо базы flights
  * Работаем с ВМ в YandexCloud
    :hammer_and_wrench: Параметр | :memo: Значение |:hammer_and_wrench: Параметр | :memo: Значение |
    --------------:|---------------|--------------:|---------------|
    | Название ВМ | **`pg-srv`** | Гарантированная доля vCPU | `100%` | 
    | Зона доступности | `ru-central1-b` | Внутренний IPv4 | `10.129.0.21` | 
    | Операционная система | `Ubuntu 20.04 LTS` | Публичный IPv4 | `51.250.107.139` |
    | Платформа | `Intel Cascade Lake	` | Тип диска | `SSD` | 
    | vCPU | `2` | Объём дискового пространства | `50 ГБ` |
    | RAM | `8 ГБ` | Макс. IOPS (чтение / запись) | `1000 / 1000` |
    | Прерываемая | :ballot_box_with_check: | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |

    ```console
    eugink@nb-xiaomi ~ $ yc compute instance create \
      --name pg-srv \
      --hostname pg-srv \
      --cores 2 \
      --memory 8 \
      --create-boot-disk size=50G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4,ipv4-address=10.129.0.21 \
      --zone ru-central1-b \
      --core-fraction 100 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/eugink/.ssh/eugin_yandex_key.pub
       ```
    
  * Устанавливаем PostgreSQL15
    ```console
    sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
    ```
    <pre><details><summary>Вывод терминала</summary>
    ubuntu@pg-srv:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
    Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease [114 kB]
    Get:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease [108 kB]               
    Get:4 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]            
    Get:5 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 Packages [2,678 kB]
    Get:6 http://mirror.yandex.ru/ubuntu focal-updates/main i386 Packages [848 kB]
    Get:7 http://mirror.yandex.ru/ubuntu focal-updates/main Translation-en [447 kB]     
    Get:8 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 c-n-f Metadata [16.9 kB]    
    Get:9 http://mirror.yandex.ru/ubuntu focal-updates/restricted amd64 Packages [2,045 kB]    
    Get:10 http://mirror.yandex.ru/ubuntu focal-updates/restricted Translation-en [286 kB]
    Get:11 http://mirror.yandex.ru/ubuntu focal-updates/universe i386 Packages [733 kB]
    Get:12 http://mirror.yandex.ru/ubuntu focal-updates/universe amd64 Packages [1,076 kB]
    Get:13 http://mirror.yandex.ru/ubuntu focal-updates/universe Translation-en [256 kB]
    Get:14 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [2,267 kB]
    Get:15 http://security.ubuntu.com/ubuntu focal-security/main i386 Packages [613 kB]
    Fetched 11.6 MB in 3s (4,435 kB/s)                        
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    5 packages can be upgraded. Run 'apt list --upgradable' to see them.
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    Calculating upgrade... Done
    The following NEW packages will be installed:
      linux-headers-5.4.0-153 linux-headers-5.4.0-153-generic linux-image-5.4.0-153-generic linux-modules-5.4.0-153-generic linux-modules-extra-5.4.0-153-generic
    The following packages will be upgraded:
      accountsservice libaccountsservice0 linux-generic linux-headers-generic linux-image-generic
    5 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
    2 standard LTS security updates
    Need to get 77.3 MB of archives.
    After this operation, 380 MB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 accountsservice amd64 0.6.55-0ubuntu12~20.04.6 [61.4 kB]
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libaccountsservice0 amd64 0.6.55-0ubuntu12~20.04.6 [72.7 kB]
    Get:3 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-modules-5.4.0-153-generic amd64 5.4.0-153.170 [15.0 MB]
    Get:4 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-image-5.4.0-153-generic amd64 5.4.0-153.170 [10.5 MB]
    Get:5 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-modules-extra-5.4.0-153-generic amd64 5.4.0-153.170 [39.2 MB]
    Get:6 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-generic amd64 5.4.0.153.150 [1,904 B]                                                                                                                                  
    Get:7 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-image-generic amd64 5.4.0.153.150 [2,604 B]                                                                                                                            
    Get:8 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-headers-5.4.0-153 all 5.4.0-153.170 [11.0 MB]                                                                                                                          
    Get:9 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-headers-5.4.0-153-generic amd64 5.4.0-153.170 [1,363 kB]                                                                                                               
    Get:10 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-headers-generic amd64 5.4.0.153.150 [2,464 B]                                                                                                                         
    Fetched 77.3 MB in 11s (7,031 kB/s)                                                                                                                                                                                                        
    (Reading database ... 102594 files and directories currently installed.)
    Preparing to unpack .../0-accountsservice_0.6.55-0ubuntu12~20.04.6_amd64.deb ...
    Unpacking accountsservice (0.6.55-0ubuntu12~20.04.6) over (0.6.55-0ubuntu12~20.04.5) ...
    Preparing to unpack .../1-libaccountsservice0_0.6.55-0ubuntu12~20.04.6_amd64.deb ...
    Unpacking libaccountsservice0:amd64 (0.6.55-0ubuntu12~20.04.6) over (0.6.55-0ubuntu12~20.04.5) ...
    Selecting previously unselected package linux-modules-5.4.0-153-generic.
    Preparing to unpack .../2-linux-modules-5.4.0-153-generic_5.4.0-153.170_amd64.deb ...
    Unpacking linux-modules-5.4.0-153-generic (5.4.0-153.170) ...
    Selecting previously unselected package linux-image-5.4.0-153-generic.
    Preparing to unpack .../3-linux-image-5.4.0-153-generic_5.4.0-153.170_amd64.deb ...
    Unpacking linux-image-5.4.0-153-generic (5.4.0-153.170) ...
    Selecting previously unselected package linux-modules-extra-5.4.0-153-generic.
    Preparing to unpack .../4-linux-modules-extra-5.4.0-153-generic_5.4.0-153.170_amd64.deb ...
    Unpacking linux-modules-extra-5.4.0-153-generic (5.4.0-153.170) ...
    Preparing to unpack .../5-linux-generic_5.4.0.153.150_amd64.deb ...
    Unpacking linux-generic (5.4.0.153.150) over (5.4.0.152.149) ...
    Preparing to unpack .../6-linux-image-generic_5.4.0.153.150_amd64.deb ...
    Unpacking linux-image-generic (5.4.0.153.150) over (5.4.0.152.149) ...
    Selecting previously unselected package linux-headers-5.4.0-153.
    Preparing to unpack .../7-linux-headers-5.4.0-153_5.4.0-153.170_all.deb ...
    Unpacking linux-headers-5.4.0-153 (5.4.0-153.170) ...
    Selecting previously unselected package linux-headers-5.4.0-153-generic.
    Preparing to unpack .../8-linux-headers-5.4.0-153-generic_5.4.0-153.170_amd64.deb ...
    Unpacking linux-headers-5.4.0-153-generic (5.4.0-153.170) ...
    Preparing to unpack .../9-linux-headers-generic_5.4.0.153.150_amd64.deb ...
    Unpacking linux-headers-generic (5.4.0.153.150) over (5.4.0.152.149) ...
    Setting up linux-headers-5.4.0-153 (5.4.0-153.170) ...
    Setting up linux-modules-5.4.0-153-generic (5.4.0-153.170) ...
    Setting up linux-image-5.4.0-153-generic (5.4.0-153.170) ...
    I: /boot/vmlinuz.old is now a symlink to vmlinuz-5.4.0-152-generic
    I: /boot/initrd.img.old is now a symlink to initrd.img-5.4.0-152-generic
    I: /boot/vmlinuz is now a symlink to vmlinuz-5.4.0-153-generic
    I: /boot/initrd.img is now a symlink to initrd.img-5.4.0-153-generic
    Setting up linux-headers-5.4.0-153-generic (5.4.0-153.170) ...
    Setting up libaccountsservice0:amd64 (0.6.55-0ubuntu12~20.04.6) ...
    Setting up accountsservice (0.6.55-0ubuntu12~20.04.6) ...
    Setting up linux-headers-generic (5.4.0.153.150) ...
    Setting up linux-modules-extra-5.4.0-153-generic (5.4.0-153.170) ...
    Setting up linux-image-generic (5.4.0.153.150) ...
    Setting up linux-generic (5.4.0.153.150) ...
    Processing triggers for dbus (1.12.16-2ubuntu2.3) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    Processing triggers for linux-image-5.4.0-153-generic (5.4.0-153.170) ...
    /etc/kernel/postinst.d/initramfs-tools:
    update-initramfs: Generating /boot/initrd.img-5.4.0-153-generic
    /etc/kernel/postinst.d/zz-update-grub:
    Sourcing file `/etc/default/grub'
    Sourcing file `/etc/default/grub.d/init-select.cfg'
    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-5.4.0-153-generic
    Found initrd image: /boot/initrd.img-5.4.0-153-generic
    Found linux image: /boot/vmlinuz-5.4.0-152-generic
    Found initrd image: /boot/initrd.img-5.4.0-152-generic
    Found linux image: /boot/vmlinuz-5.4.0-42-generic
    Found initrd image: /boot/initrd.img-5.4.0-42-generic
    done
    OK
    Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
    Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease             
    Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease                                                               
    Get:4 http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease [117 kB]                                                  
    Get:5 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]                              
    Get:6 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 Packages [265 kB]
    Fetched 496 kB in 1s (557 kB/s)                                   
    Reading package lists... Done
    N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease' doesn't support architecture 'i386'
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following packages were automatically installed and are no longer required:
      linux-headers-5.4.0-42 linux-headers-5.4.0-42-generic linux-image-5.4.0-42-generic linux-modules-5.4.0-42-generic linux-modules-extra-5.4.0-42-generic
    Use 'sudo apt autoremove' to remove them.
    The following additional packages will be installed:
      libcommon-sense-perl libgdbm-compat4 libjson-perl libjson-xs-perl libllvm10 libperl5.30 libpq5 libsensors-config libsensors5 libtypes-serialiser-perl libxslt1.1 perl perl-modules-5.30 postgresql-client-15 postgresql-client-common
      postgresql-common ssl-cert sysstat
    Suggested packages:
      lm-sensors perl-doc libterm-readline-gnu-perl | libterm-readline-perl-perl make libb-debug-perl liblocale-codes-perl postgresql-doc-15 openssl-blacklist isag
    The following NEW packages will be installed:
      libcommon-sense-perl libgdbm-compat4 libjson-perl libjson-xs-perl libllvm10 libperl5.30 libpq5 libsensors-config libsensors5 libtypes-serialiser-perl libxslt1.1 perl perl-modules-5.30 postgresql-15 postgresql-client-15
      postgresql-client-common postgresql-common ssl-cert sysstat
    0 upgraded, 19 newly installed, 0 to remove and 0 not upgraded.
    Need to get 41.6 MB of archives.
    After this operation, 186 MB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl-modules-5.30 all 5.30.0-9ubuntu0.4 [2,739 kB]
    Get:2 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-client-common all 250.pgdg20.04+1 [93.3 kB]
    Get:3 http://mirror.yandex.ru/ubuntu focal/main amd64 libgdbm-compat4 amd64 1.18.1-5 [6,244 B]
    Get:4 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libperl5.30 amd64 5.30.0-9ubuntu0.4 [3,959 kB]
    Get:5 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-common all 250.pgdg20.04+1 [239 kB]
    Get:6 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 libpq5 amd64 15.3-1.pgdg20.04+1 [184 kB]
    Get:7 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-client-15 amd64 15.3-1.pgdg20.04+1 [1,680 kB]
    Get:8 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-15 amd64 15.3-1.pgdg20.04+1 [16.3 MB]
    Get:9 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl amd64 5.30.0-9ubuntu0.4 [224 kB]
    Get:10 http://mirror.yandex.ru/ubuntu focal/main amd64 libjson-perl all 4.02000-2 [80.9 kB]
    Get:11 http://mirror.yandex.ru/ubuntu focal/main amd64 ssl-cert all 1.0.39 [17.0 kB]
    Get:12 http://mirror.yandex.ru/ubuntu focal/main amd64 libcommon-sense-perl amd64 3.74-2build6 [20.1 kB]
    Get:13 http://mirror.yandex.ru/ubuntu focal/main amd64 libtypes-serialiser-perl all 1.0-1 [12.1 kB]
    Get:14 http://mirror.yandex.ru/ubuntu focal/main amd64 libjson-xs-perl amd64 4.020-1build1 [83.7 kB]
    Get:15 http://mirror.yandex.ru/ubuntu focal/main amd64 libllvm10 amd64 1:10.0.0-4ubuntu1 [15.3 MB]
    Get:16 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors-config all 1:3.6.0-2ubuntu1.1 [6,052 B]
    Get:17 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors5 amd64 1:3.6.0-2ubuntu1.1 [27.2 kB]
    Get:18 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libxslt1.1 amd64 1.1.34-4ubuntu0.20.04.1 [151 kB]
    Get:19 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 sysstat amd64 12.2.0-2ubuntu0.3 [448 kB]
    Fetched 41.6 MB in 3s (15.6 MB/s) 
    Preconfiguring packages ...
    Selecting previously unselected package perl-modules-5.30.
    (Reading database ... 139024 files and directories currently installed.)
    Preparing to unpack .../00-perl-modules-5.30_5.30.0-9ubuntu0.4_all.deb ...
    Unpacking perl-modules-5.30 (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package libgdbm-compat4:amd64.
    Preparing to unpack .../01-libgdbm-compat4_1.18.1-5_amd64.deb ...
    Unpacking libgdbm-compat4:amd64 (1.18.1-5) ...
    Selecting previously unselected package libperl5.30:amd64.
    Preparing to unpack .../02-libperl5.30_5.30.0-9ubuntu0.4_amd64.deb ...
    Unpacking libperl5.30:amd64 (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package perl.
    Preparing to unpack .../03-perl_5.30.0-9ubuntu0.4_amd64.deb ...
    Unpacking perl (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package libjson-perl.
    Preparing to unpack .../04-libjson-perl_4.02000-2_all.deb ...
    Unpacking libjson-perl (4.02000-2) ...
    Selecting previously unselected package postgresql-client-common.
    Preparing to unpack .../05-postgresql-client-common_250.pgdg20.04+1_all.deb ...
    Unpacking postgresql-client-common (250.pgdg20.04+1) ...
    Selecting previously unselected package ssl-cert.
    Preparing to unpack .../06-ssl-cert_1.0.39_all.deb ...
    Unpacking ssl-cert (1.0.39) ...
    Selecting previously unselected package postgresql-common.
    Preparing to unpack .../07-postgresql-common_250.pgdg20.04+1_all.deb ...
    Adding 'diversion of /usr/bin/pg_config to /usr/bin/pg_config.libpq-dev by postgresql-common'
    Unpacking postgresql-common (250.pgdg20.04+1) ...
    Selecting previously unselected package libcommon-sense-perl.
    Preparing to unpack .../08-libcommon-sense-perl_3.74-2build6_amd64.deb ...
    Unpacking libcommon-sense-perl (3.74-2build6) ...
    Selecting previously unselected package libtypes-serialiser-perl.
    Preparing to unpack .../09-libtypes-serialiser-perl_1.0-1_all.deb ...
    Unpacking libtypes-serialiser-perl (1.0-1) ...
    Selecting previously unselected package libjson-xs-perl.
    Preparing to unpack .../10-libjson-xs-perl_4.020-1build1_amd64.deb ...
    Unpacking libjson-xs-perl (4.020-1build1) ...
    Selecting previously unselected package libllvm10:amd64.
    Preparing to unpack .../11-libllvm10_1%3a10.0.0-4ubuntu1_amd64.deb ...
    Unpacking libllvm10:amd64 (1:10.0.0-4ubuntu1) ...
    Selecting previously unselected package libpq5:amd64.
    Preparing to unpack .../12-libpq5_15.3-1.pgdg20.04+1_amd64.deb ...
    Unpacking libpq5:amd64 (15.3-1.pgdg20.04+1) ...
    Selecting previously unselected package libsensors-config.
    Preparing to unpack .../13-libsensors-config_1%3a3.6.0-2ubuntu1.1_all.deb ...
    Unpacking libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libsensors5:amd64.
    Preparing to unpack .../14-libsensors5_1%3a3.6.0-2ubuntu1.1_amd64.deb ...
    Unpacking libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libxslt1.1:amd64.
    Preparing to unpack .../15-libxslt1.1_1.1.34-4ubuntu0.20.04.1_amd64.deb ...
    Unpacking libxslt1.1:amd64 (1.1.34-4ubuntu0.20.04.1) ...
    Selecting previously unselected package postgresql-client-15.
    Preparing to unpack .../16-postgresql-client-15_15.3-1.pgdg20.04+1_amd64.deb ...
    Unpacking postgresql-client-15 (15.3-1.pgdg20.04+1) ...
    Selecting previously unselected package postgresql-15.
    Preparing to unpack .../17-postgresql-15_15.3-1.pgdg20.04+1_amd64.deb ...
    Unpacking postgresql-15 (15.3-1.pgdg20.04+1) ...
    Selecting previously unselected package sysstat.
    Preparing to unpack .../18-sysstat_12.2.0-2ubuntu0.3_amd64.deb ...
    Unpacking sysstat (12.2.0-2ubuntu0.3) ...
    Setting up perl-modules-5.30 (5.30.0-9ubuntu0.4) ...
    Setting up libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Setting up libpq5:amd64 (15.3-1.pgdg20.04+1) ...
    Setting up libllvm10:amd64 (1:10.0.0-4ubuntu1) ...
    Setting up ssl-cert (1.0.39) ...
    Setting up libgdbm-compat4:amd64 (1.18.1-5) ...
    Setting up libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Setting up libxslt1.1:amd64 (1.1.34-4ubuntu0.20.04.1) ...
    Setting up libperl5.30:amd64 (5.30.0-9ubuntu0.4) ...
    Setting up sysstat (12.2.0-2ubuntu0.3) ...
    
    Creating config file /etc/default/sysstat with new version
    update-alternatives: using /usr/bin/sar.sysstat to provide /usr/bin/sar (sar) in auto mode
    Created symlink /etc/systemd/system/multi-user.target.wants/sysstat.service → /lib/systemd/system/sysstat.service.
    Setting up perl (5.30.0-9ubuntu0.4) ...
    Setting up libjson-perl (4.02000-2) ...
    Setting up postgresql-client-common (250.pgdg20.04+1) ...
    Setting up libcommon-sense-perl (3.74-2build6) ...
    Setting up postgresql-client-15 (15.3-1.pgdg20.04+1) ...
    update-alternatives: using /usr/share/postgresql/15/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
    Setting up postgresql-common (250.pgdg20.04+1) ...
    Adding user postgres to group ssl-cert
    
    Creating config file /etc/postgresql-common/createcluster.conf with new version
    Building PostgreSQL dictionaries from installed myspell/hunspell packages...
    Removing obsolete dictionary files:
    '/etc/apt/trusted.gpg.d/apt.postgresql.org.gpg' -> '/usr/share/postgresql-common/pgdg/apt.postgresql.org.gpg'
    Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /lib/systemd/system/postgresql.service.
    Setting up libtypes-serialiser-perl (1.0-1) ...
    Setting up postgresql-15 (15.3-1.pgdg20.04+1) ...
    Creating new PostgreSQL cluster 15/main ...
    /usr/lib/postgresql/15/bin/initdb -D /var/lib/postgresql/15/main --auth-local peer --auth-host scram-sha-256 --no-instructions
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.
    
    The database cluster will be initialized with locale "en_US.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".
    
    Data page checksums are disabled.
    
    fixing permissions on existing directory /var/lib/postgresql/15/main ... ok
    creating subdirectories ... ok
    selecting dynamic shared memory implementation ... posix
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting default time zone ... Etc/UTC
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... ok
    syncing data to disk ... ok
    Setting up libjson-xs-perl (4.020-1build1) ...
    Processing triggers for systemd (245.4-4ubuntu3.22) ...
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    ubuntu@pg-srv:~$ 
    </details></pre>
  * Загружаем большую БД с полетами за год [demo-big](https://edu.postgrespro.ru/demo-big.zip)
    <pre><details><summary>Вывод терминала</summary>
    ubuntu@pg-srv:~$ cd /tmp/
    ubuntu@pg-srv:/tmp$ 
    ubuntu@pg-srv:/tmp$ wget https://edu.postgrespro.ru/demo-big.zip
    --2023-06-28 20:42:55--  https://edu.postgrespro.ru/demo-big.zip
    Resolving edu.postgrespro.ru (edu.postgrespro.ru)... 213.171.56.196
    Connecting to edu.postgrespro.ru (edu.postgrespro.ru)|213.171.56.196|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 243203214 (232M) [application/zip]
    Saving to: ‘demo-big.zip’
    
    demo-big.zip                                               100%[========================================================================================================================================>] 231.94M  22.3MB/s    in 11s     
    
    2023-06-28 20:43:05 (22.1 MB/s) - ‘demo-big.zip’ saved [243203214/243203214]
    
    ubuntu@pg-srv:/tmp$ 
    ubuntu@pg-srv:/tmp$ chmod 777 /tmp/demo-big.zip 
    ubuntu@pg-srv:/tmp$ 
    ubuntu@pg-srv:/tmp$ unzip demo-big.zip 
    Archive:  demo-big.zip
      inflating: demo-big-20170815.sql   
    ubuntu@pg-srv:/tmp$ 
    ubuntu@pg-srv:/tmp$ ls -ltr
    total 1146768
    -rw-rw-r-- 1 ubuntu ubuntu 931068524 Jan  5  2018 demo-big-20170815.sql
    -rwxrwxrwx 1 ubuntu ubuntu 243203214 Jan 11  2018 demo-big.zip
    drwx------ 3 root   root        4096 Jun 28 20:16 systemd-private-52a2987604b24b48975b7e85755c6c51-systemd-timesyncd.service-32aSZi
    drwx------ 3 root   root        4096 Jun 28 20:16 systemd-private-52a2987604b24b48975b7e85755c6c51-systemd-resolved.service-Tqv8dj
    drwx------ 3 root   root        4096 Jun 28 20:16 systemd-private-52a2987604b24b48975b7e85755c6c51-systemd-logind.service-KJrn8e
    drwxr-xr-x 2 root   root        4096 Jun 28 20:20 ubuntu-advantage
    ubuntu@pg-srv:/tmp$ 
    ubuntu@pg-srv:/tmp$ sudo -u postgres psql -f demo-big-20170815.sql 
    SET
    SET
    SET
    SET
    SET
    SET
    SET
    SET
    psql:demo-big-20170815.sql:17: ERROR:  database "demo" does not exist
    CREATE DATABASE
    You are now connected to database "demo" as user "postgres".
    SET
    SET
    SET
    SET
    SET
    SET
    SET
    SET
    CREATE SCHEMA
    COMMENT
    CREATE EXTENSION
    COMMENT
    SET
    CREATE FUNCTION
    CREATE FUNCTION
    COMMENT
    SET
    SET
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE VIEW
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE VIEW
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE SEQUENCE
    ALTER SEQUENCE
    CREATE VIEW
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE VIEW
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    CREATE TABLE
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    COMMENT
    ALTER TABLE
    COPY 9
    COPY 104
    COPY 7925812
    COPY 2111110
    COPY 214867
    COPY 1339
    COPY 8391852
    COPY 2949857
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER TABLE
    ALTER DATABASE
    ALTER DATABASE
    ubuntu@pg-srv:/tmp$ 
    </details></pre>
  * Смотрим, какой диапазон данных(mix, max) есть в таблице `flights`
  ```pgsql
  demo=# select min(scheduled_departure),max(scheduled_departure) from flights;
            min           |          max           
  ------------------------+------------------------
   2016-08-14 23:45:00+00 | 2017-09-14 17:55:00+00
  (1 row)
  ```

  * Создаем партицированную таблицу `flights_new`, с ключом по диапазону `range` по полю `scheduled_departure`+партиции
  ```pgsql
  demo=# create table flights_new (like flights) partition by range (scheduled_departure);
  CREATE TABLE
  demo=# 
  demo=# create table flights_new_2016_08 partition of flights_new for values from ('2016-08-01') to ('2016-09-01');
  CREATE TABLE
  demo=# create table flights_new_2016_09 partition of flights_new for values from ('2016-09-01') to ('2016-10-01');
  CREATE TABLE
  demo=# create table flights_new_2016_10 partition of flights_new for values from ('2016-10-01') to ('2016-11-01');
  CREATE TABLE
  demo=# create table flights_new_2016_11 partition of flights_new for values from ('2016-11-01') to ('2016-12-01');
  CREATE TABLE
  demo=# create table flights_new_2016_12 partition of flights_new for values from ('2016-12-01') to ('2017-01-01');
  CREATE TABLE
  demo=# create table flights_new_2017_01 partition of flights_new for values from ('2017-01-01') to ('2017-02-01');
  CREATE TABLE
  demo=# create table flights_new_2017_02 partition of flights_new for values from ('2017-02-01') to ('2017-03-01');
  CREATE TABLE
  demo=# create table flights_new_2017_03 partition of flights_new for values from ('2017-03-01') to ('2017-04-01');
  CREATE TABLE
  demo=# create table flights_new_2017_04 partition of flights_new for values from ('2017-04-01') to ('2017-05-01');
  CREATE TABLE
  demo=# create table flights_new_2017_05 partition of flights_new for values from ('2017-05-01') to ('2017-06-01');
  CREATE TABLE
  demo=# create table flights_new_2017_06 partition of flights_new for values from ('2017-06-01') to ('2017-07-01');
  CREATE TABLE
  demo=# create table flights_new_2017_07 partition of flights_new for values from ('2017-07-01') to ('2017-08-01');
  CREATE TABLE
  demo=# create table flights_new_2017_08 partition of flights_new for values from ('2017-08-01') to ('2017-09-01');
  CREATE TABLE
  demo=# create table flights_new_2017_09 partition of flights_new for values from ('2017-09-01') to ('2017-10-01');
  CREATE TABLE
  demo=# 
  demo=# 
  demo=# \d+ flights_new
                                                 Partitioned table "bookings.flights_new"
         Column        |           Type           | Collation | Nullable | Default | Storage  | Compression | Stats target | Description 
  ---------------------+--------------------------+-----------+----------+---------+----------+-------------+--------------+-------------
   flight_id           | integer                  |           | not null |         | plain    |             |              | 
   flight_no           | character(6)             |           | not null |         | extended |             |              | 
   scheduled_departure | timestamp with time zone |           | not null |         | plain    |             |              | 
   scheduled_arrival   | timestamp with time zone |           | not null |         | plain    |             |              | 
   departure_airport   | character(3)             |           | not null |         | extended |             |              | 
   arrival_airport     | character(3)             |           | not null |         | extended |             |              | 
   status              | character varying(20)    |           | not null |         | extended |             |              | 
   aircraft_code       | character(3)             |           | not null |         | extended |             |              | 
   actual_departure    | timestamp with time zone |           |          |         | plain    |             |              | 
   actual_arrival      | timestamp with time zone |           |          |         | plain    |             |              | 
  Partition key: RANGE (scheduled_departure)
  Partitions: flights_new_2016_08 FOR VALUES FROM ('2016-08-01 00:00:00+00') TO ('2016-09-01 00:00:00+00'),
              flights_new_2016_09 FOR VALUES FROM ('2016-09-01 00:00:00+00') TO ('2016-10-01 00:00:00+00'),
              flights_new_2016_10 FOR VALUES FROM ('2016-10-01 00:00:00+00') TO ('2016-11-01 00:00:00+00'),
              flights_new_2016_11 FOR VALUES FROM ('2016-11-01 00:00:00+00') TO ('2016-12-01 00:00:00+00'),
              flights_new_2016_12 FOR VALUES FROM ('2016-12-01 00:00:00+00') TO ('2017-01-01 00:00:00+00'),
              flights_new_2017_01 FOR VALUES FROM ('2017-01-01 00:00:00+00') TO ('2017-02-01 00:00:00+00'),
              flights_new_2017_02 FOR VALUES FROM ('2017-02-01 00:00:00+00') TO ('2017-03-01 00:00:00+00'),
              flights_new_2017_03 FOR VALUES FROM ('2017-03-01 00:00:00+00') TO ('2017-04-01 00:00:00+00'),
              flights_new_2017_04 FOR VALUES FROM ('2017-04-01 00:00:00+00') TO ('2017-05-01 00:00:00+00'),
              flights_new_2017_05 FOR VALUES FROM ('2017-05-01 00:00:00+00') TO ('2017-06-01 00:00:00+00'),
              flights_new_2017_06 FOR VALUES FROM ('2017-06-01 00:00:00+00') TO ('2017-07-01 00:00:00+00'),
              flights_new_2017_07 FOR VALUES FROM ('2017-07-01 00:00:00+00') TO ('2017-08-01 00:00:00+00'),
              flights_new_2017_08 FOR VALUES FROM ('2017-08-01 00:00:00+00') TO ('2017-09-01 00:00:00+00'),
              flights_new_2017_09 FOR VALUES FROM ('2017-09-01 00:00:00+00') TO ('2017-10-01 00:00:00+00')
  
  demo=# 
  ```
  * Заливаем данные из таблицы `flights` в таблицу `flights_new` и смотрим размер
  ```pgsql
  demo=# insert into flights_new (select * from flights);
  INSERT 0 214867
  demo=# 
  demo=# 
  demo=# select 'select pg_size_pretty(pg_table_size('''||tablename||'''));' from pg_tables where tablename like 'flights_new%';
                             ?column?                           
  --------------------------------------------------------------
   select pg_size_pretty(pg_table_size('flights_new'));
   select pg_size_pretty(pg_table_size('flights_new_2016_08'));
   select pg_size_pretty(pg_table_size('flights_new_2016_09'));
   select pg_size_pretty(pg_table_size('flights_new_2016_10'));
   select pg_size_pretty(pg_table_size('flights_new_2016_11'));
   select pg_size_pretty(pg_table_size('flights_new_2016_12'));
   select pg_size_pretty(pg_table_size('flights_new_2017_01'));
   select pg_size_pretty(pg_table_size('flights_new_2017_02'));
   select pg_size_pretty(pg_table_size('flights_new_2017_03'));
   select pg_size_pretty(pg_table_size('flights_new_2017_04'));
   select pg_size_pretty(pg_table_size('flights_new_2017_05'));
   select pg_size_pretty(pg_table_size('flights_new_2017_06'));
   select pg_size_pretty(pg_table_size('flights_new_2017_07'));
   select pg_size_pretty(pg_table_size('flights_new_2017_08'));
   select pg_size_pretty(pg_table_size('flights_new_2017_09'));
  (15 rows)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new'));
   pg_size_pretty 
  ----------------
   0 bytes
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2016_08'));
   pg_size_pretty 
  ----------------
   944 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2016_09'));
   pg_size_pretty 
  ----------------
   1640 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2016_10'));
   pg_size_pretty 
  ----------------
   1704 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2016_11'));
   pg_size_pretty 
  ----------------
   1648 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2016_12'));
   pg_size_pretty 
  ----------------
   1696 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_01'));
   pg_size_pretty 
  ----------------
   1696 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_02'));
   pg_size_pretty 
  ----------------
   1536 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_03'));
   pg_size_pretty 
  ----------------
   1696 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_04'));
   pg_size_pretty 
  ----------------
   1648 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_05'));
   pg_size_pretty 
  ----------------
   1696 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_06'));
   pg_size_pretty 
  ----------------
   1640 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_07'));
   pg_size_pretty 
  ----------------
   1704 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_08'));
   pg_size_pretty 
  ----------------
   1624 kB
  (1 row)
  
  demo=#  select pg_size_pretty(pg_table_size('flights_new_2017_09'));
   pg_size_pretty 
  ----------------
   728 kB
  (1 row)
  
  demo=# 
  ```

  * Видим, что основная таблица не соддержит данных, а партиции-содержат
  * Чтобы работал режим выборки данных из партиций, надо чтобы параметры [enable_partition_pruning](https://postgrespro.ru/docs/postgresql/15/runtime-config-query#GUC-ENABLE-PARTITION-PRUNING) был равен `on`
  * В реальной жизни, я бы создал партицированную таблицу и порциями заливал туда данные из непартиционированной.
  * Не хватает split и merge partition в стандартной реализации postgre
