<div align="center"><h2> 6. Настройка KeepAlived </h2></div>

***

### Немного теории:
  * Для управления VIP (Virtual IP) используется отсдельное ПО. Его задача состоит в том, чтобы обеспечить единную точку входа для конечных пользователей
  * В качестве менеджера по управлению VIP могут выступать:
      * [`KeepAlived`](https://keepalived.org/index.html) ❗️Именно его мы и будет использовать
      * [`VIP-Manager`](https://github.com/cybertec-postgresql/vip-manager)
        * Можно не использовать `HAProxy`

*** 
### Список ВМ с которыми идет взаимодействие на данном этапе
  :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
  |--------------:|---------------|
  | **`hap1`** | `10.129.0.11` |
  | **`hap2`** | `10.129.0.12` |      
  
***
### Установка `KeepAlived`
  * ❗️Выполняем установку на 2х ВМ
    ```bash
    sudo apt-get install keepalived -y
    ```
    ```console
    ubuntu@hap1:~$ sudo apt-get install keepalived -y
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:
      ipvsadm libgdbm-compat4 libmysqlclient21 libperl5.30 libsensors-config libsensors5
      libsnmp-base libsnmp35 mysql-common perl perl-modules-5.30
    Suggested packages:
      heartbeat ldirectord lm-sensors snmp-mibs-downloader perl-doc libterm-readline-gnu-perl
      | libterm-readline-perl-perl make libb-debug-perl liblocale-codes-perl
    The following NEW packages will be installed:
      ipvsadm keepalived libgdbm-compat4 libmysqlclient21 libperl5.30 libsensors-config
      libsensors5 libsnmp-base libsnmp35 mysql-common perl perl-modules-5.30
    0 upgraded, 12 newly installed, 0 to remove and 0 not upgraded.
    Need to get 9,881 kB of archives.
    After this operation, 59.8 MB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal/main amd64 mysql-common all 5.8+1.0.5ubuntu2 [7,496 B]
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libmysqlclient21 amd64 8.0.33-0ubuntu0.20.04.4 [1,327 kB]
    Get:3 http://mirror.yandex.ru/ubuntu focal/main amd64 libgdbm-compat4 amd64 1.18.1-5 [6,244 B]
    Get:4 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl-modules-5.30 all 5.30.0-9ubuntu0.4 [2,739 kB]
    Get:5 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libperl5.30 amd64 5.30.0-9ubuntu0.4 [3,959 kB]
    Get:6 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors-config all 1:3.6.0-2ubuntu1.1 [6,052 B]
    Get:7 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors5 amd64 1:3.6.0-2ubuntu1.1 [27.2 kB]
    Get:8 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsnmp-base all 5.8+dfsg-2ubuntu2.7 [206 kB]
    Get:9 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsnmp35 amd64 5.8+dfsg-2ubuntu2.7 [978 kB]
    Get:10 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 keepalived amd64 1:2.0.19-2ubuntu0.2 [361 kB]
    Get:11 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl amd64 5.30.0-9ubuntu0.4 [224 kB]
    Get:12 http://mirror.yandex.ru/ubuntu focal/main amd64 ipvsadm amd64 1:1.31-1 [40.2 kB]
    Fetched 9,881 kB in 0s (23.9 MB/s)   
    Selecting previously unselected package mysql-common.
    (Reading database ... 102602 files and directories currently installed.)
    Preparing to unpack .../00-mysql-common_5.8+1.0.5ubuntu2_all.deb ...
    Unpacking mysql-common (5.8+1.0.5ubuntu2) ...
    Selecting previously unselected package libmysqlclient21:amd64.
    Preparing to unpack .../01-libmysqlclient21_8.0.33-0ubuntu0.20.04.4_amd64.deb ...
    Unpacking libmysqlclient21:amd64 (8.0.33-0ubuntu0.20.04.4) ...
    Selecting previously unselected package libgdbm-compat4:amd64.
    Preparing to unpack .../02-libgdbm-compat4_1.18.1-5_amd64.deb ...
    Unpacking libgdbm-compat4:amd64 (1.18.1-5) ...
    Selecting previously unselected package perl-modules-5.30.
    Preparing to unpack .../03-perl-modules-5.30_5.30.0-9ubuntu0.4_all.deb ...
    Unpacking perl-modules-5.30 (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package libperl5.30:amd64.
    Preparing to unpack .../04-libperl5.30_5.30.0-9ubuntu0.4_amd64.deb ...
    Unpacking libperl5.30:amd64 (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package libsensors-config.
    Preparing to unpack .../05-libsensors-config_1%3a3.6.0-2ubuntu1.1_all.deb ...
    Unpacking libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libsensors5:amd64.
    Preparing to unpack .../06-libsensors5_1%3a3.6.0-2ubuntu1.1_amd64.deb ...
    Unpacking libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libsnmp-base.
    Preparing to unpack .../07-libsnmp-base_5.8+dfsg-2ubuntu2.7_all.deb ...
    Unpacking libsnmp-base (5.8+dfsg-2ubuntu2.7) ...
    Selecting previously unselected package libsnmp35:amd64.
    Preparing to unpack .../08-libsnmp35_5.8+dfsg-2ubuntu2.7_amd64.deb ...
    Unpacking libsnmp35:amd64 (5.8+dfsg-2ubuntu2.7) ...
    Selecting previously unselected package keepalived.
    Preparing to unpack .../09-keepalived_1%3a2.0.19-2ubuntu0.2_amd64.deb ...
    Unpacking keepalived (1:2.0.19-2ubuntu0.2) ...
    Selecting previously unselected package perl.
    Preparing to unpack .../10-perl_5.30.0-9ubuntu0.4_amd64.deb ...
    Unpacking perl (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package ipvsadm.
    Preparing to unpack .../11-ipvsadm_1%3a1.31-1_amd64.deb ...
    Unpacking ipvsadm (1:1.31-1) ...
    Setting up ipvsadm (1:1.31-1) ...
    Setting up mysql-common (5.8+1.0.5ubuntu2) ...
    update-alternatives: using /etc/mysql/my.cnf.fallback to provide /etc/mysql/my.cnf (my.cnf) in auto mode
    Setting up libmysqlclient21:amd64 (8.0.33-0ubuntu0.20.04.4) ...
    Setting up perl-modules-5.30 (5.30.0-9ubuntu0.4) ...
    Setting up libsnmp-base (5.8+dfsg-2ubuntu2.7) ...
    Setting up libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Setting up libgdbm-compat4:amd64 (1.18.1-5) ...
    Setting up libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Setting up libperl5.30:amd64 (5.30.0-9ubuntu0.4) ...
    Setting up libsnmp35:amd64 (5.8+dfsg-2ubuntu2.7) ...
    Setting up keepalived (1:2.0.19-2ubuntu0.2) ...
    Created symlink /etc/systemd/system/multi-user.target.wants/keepalived.service → /lib/systemd/system/keepalived.service.
    Setting up perl (5.30.0-9ubuntu0.4) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    Processing triggers for systemd (245.4-4ubuntu3.22) ...
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for dbus (1.12.16-2ubuntu2.3) ...
    ubuntu@hap1:~$ 
    ```
    ```console
    ubuntu@hap2:~$ sudo apt-get install keepalived -y
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:
      ipvsadm libgdbm-compat4 libmysqlclient21 libperl5.30 libsensors-config libsensors5
      libsnmp-base libsnmp35 mysql-common perl perl-modules-5.30
    Suggested packages:
      heartbeat ldirectord lm-sensors snmp-mibs-downloader perl-doc libterm-readline-gnu-perl
      | libterm-readline-perl-perl make libb-debug-perl liblocale-codes-perl
    The following NEW packages will be installed:
      ipvsadm keepalived libgdbm-compat4 libmysqlclient21 libperl5.30 libsensors-config
      libsensors5 libsnmp-base libsnmp35 mysql-common perl perl-modules-5.30
    0 upgraded, 12 newly installed, 0 to remove and 0 not upgraded.
    Need to get 9,881 kB of archives.
    After this operation, 59.8 MB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal/main amd64 mysql-common all 5.8+1.0.5ubuntu2 [7,496 B]
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libmysqlclient21 amd64 8.0.33-0ubuntu0.20.04.4 [1,327 kB]
    Get:3 http://mirror.yandex.ru/ubuntu focal/main amd64 libgdbm-compat4 amd64 1.18.1-5 [6,244 B]
    Get:4 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl-modules-5.30 all 5.30.0-9ubuntu0.4 [2,739 kB]
    Get:5 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libperl5.30 amd64 5.30.0-9ubuntu0.4 [3,959 kB]
    Get:6 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors-config all 1:3.6.0-2ubuntu1.1 [6,052 B]
    Get:7 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors5 amd64 1:3.6.0-2ubuntu1.1 [27.2 kB]
    Get:8 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsnmp-base all 5.8+dfsg-2ubuntu2.7 [206 kB]
    Get:9 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsnmp35 amd64 5.8+dfsg-2ubuntu2.7 [978 kB]
    Get:10 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 keepalived amd64 1:2.0.19-2ubuntu0.2 [361 kB]
    Get:11 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl amd64 5.30.0-9ubuntu0.4 [224 kB]
    Get:12 http://mirror.yandex.ru/ubuntu focal/main amd64 ipvsadm amd64 1:1.31-1 [40.2 kB]
    Fetched 9,881 kB in 0s (61.4 MB/s)   
    Selecting previously unselected package mysql-common.
    (Reading database ... 102602 files and directories currently installed.)
    Preparing to unpack .../00-mysql-common_5.8+1.0.5ubuntu2_all.deb ...
    Unpacking mysql-common (5.8+1.0.5ubuntu2) ...
    Selecting previously unselected package libmysqlclient21:amd64.
    Preparing to unpack .../01-libmysqlclient21_8.0.33-0ubuntu0.20.04.4_amd64.deb ...
    Unpacking libmysqlclient21:amd64 (8.0.33-0ubuntu0.20.04.4) ...
    Selecting previously unselected package libgdbm-compat4:amd64.
    Preparing to unpack .../02-libgdbm-compat4_1.18.1-5_amd64.deb ...
    Unpacking libgdbm-compat4:amd64 (1.18.1-5) ...
    Selecting previously unselected package perl-modules-5.30.
    Preparing to unpack .../03-perl-modules-5.30_5.30.0-9ubuntu0.4_all.deb ...
    Unpacking perl-modules-5.30 (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package libperl5.30:amd64.
    Preparing to unpack .../04-libperl5.30_5.30.0-9ubuntu0.4_amd64.deb ...
    Unpacking libperl5.30:amd64 (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package libsensors-config.
    Preparing to unpack .../05-libsensors-config_1%3a3.6.0-2ubuntu1.1_all.deb ...
    Unpacking libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libsensors5:amd64.
    Preparing to unpack .../06-libsensors5_1%3a3.6.0-2ubuntu1.1_amd64.deb ...
    Unpacking libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libsnmp-base.
    Preparing to unpack .../07-libsnmp-base_5.8+dfsg-2ubuntu2.7_all.deb ...
    Unpacking libsnmp-base (5.8+dfsg-2ubuntu2.7) ...
    Selecting previously unselected package libsnmp35:amd64.
    Preparing to unpack .../08-libsnmp35_5.8+dfsg-2ubuntu2.7_amd64.deb ...
    Unpacking libsnmp35:amd64 (5.8+dfsg-2ubuntu2.7) ...
    Selecting previously unselected package keepalived.
    Preparing to unpack .../09-keepalived_1%3a2.0.19-2ubuntu0.2_amd64.deb ...
    Unpacking keepalived (1:2.0.19-2ubuntu0.2) ...
    Selecting previously unselected package perl.
    Preparing to unpack .../10-perl_5.30.0-9ubuntu0.4_amd64.deb ...
    Unpacking perl (5.30.0-9ubuntu0.4) ...
    Selecting previously unselected package ipvsadm.
    Preparing to unpack .../11-ipvsadm_1%3a1.31-1_amd64.deb ...
    Unpacking ipvsadm (1:1.31-1) ...
    Setting up ipvsadm (1:1.31-1) ...
    Setting up mysql-common (5.8+1.0.5ubuntu2) ...
    update-alternatives: using /etc/mysql/my.cnf.fallback to provide /etc/mysql/my.cnf (my.cnf) in auto mode
    Setting up libmysqlclient21:amd64 (8.0.33-0ubuntu0.20.04.4) ...
    Setting up perl-modules-5.30 (5.30.0-9ubuntu0.4) ...
    Setting up libsnmp-base (5.8+dfsg-2ubuntu2.7) ...
    Setting up libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Setting up libgdbm-compat4:amd64 (1.18.1-5) ...
    Setting up libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Setting up libperl5.30:amd64 (5.30.0-9ubuntu0.4) ...
    Setting up libsnmp35:amd64 (5.8+dfsg-2ubuntu2.7) ...
    Setting up keepalived (1:2.0.19-2ubuntu0.2) ...
    Created symlink /etc/systemd/system/multi-user.target.wants/keepalived.service → /lib/systemd/system/keepalived.service.
    Setting up perl (5.30.0-9ubuntu0.4) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    Processing triggers for systemd (245.4-4ubuntu3.22) ...
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for dbus (1.12.16-2ubuntu2.3) ...
    ubuntu@hap2:~$ 
    ```

  * Проверяем установленную версию `KeepAlived` на 2х ВМ
    ```bash
    sudo keepalived -v
    ```
    ```console
    ubuntu@hap1:~$ sudo keepalived -v
    Keepalived v2.0.19 (10/19,2019)
    
    Copyright(C) 2001-2019 Alexandre Cassen, <acassen@gmail.com>
    
    Built with kernel headers for Linux 5.4.166
    Running on Linux 5.4.0-155-generic #172-Ubuntu SMP Fri Jul 7 16:10:02 UTC 2023
    
    configure options: --build=x86_64-linux-gnu --prefix=/usr --includedir=${prefix}/include --mandir=${prefix}/share/man --infodir=${prefix}/share/info --sysconfdir=/etc --localstatedir=/var --disable-silent-rules --libdir=${prefix}/lib/x86_64-linux-gnu --runstatedir=/run --disable-maintainer-mode --disable-dependency-tracking --with-kernel-dir=debian/ --enable-snmp --enable-sha1 --enable-snmp-rfcv2 --enable-snmp-rfcv3 --enable-dbus --enable-json --enable-bfd --enable-regex build_alias=x86_64-linux-gnu CFLAGS=-g -O2 -fdebug-prefix-map=/build/keepalived-QKJBdn/keepalived-2.0.19=. -fstack-protector-strong -Wformat -Werror=format-security LDFLAGS=-Wl,-Bsymbolic-functions -Wl,-z,relro CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=2
    
    Config options:  NFTABLES LVS REGEX VRRP VRRP_AUTH JSON BFD OLD_CHKSUM_COMPAT FIB_ROUTING SNMP_V3_FOR_V2 SNMP_VRRP SNMP_CHECKER SNMP_RFCV2 SNMP_RFCV3 DBUS
    
    System options:  PIPE2 SIGNALFD INOTIFY_INIT1 VSYSLOG EPOLL_CREATE1 IPV4_DEVCONF IPV6_ADVANCED_API LIBNL3 RTA_ENCAP RTA_EXPIRES RTA_NEWDST RTA_PREF FRA_SUPPRESS_PREFIXLEN FRA_SUPPRESS_IFGROUP FRA_TUN_ID RTAX_CC_ALGO RTAX_QUICKACK RTEXT_FILTER_SKIP_STATS FRA_L3MDEV FRA_UID_RANGE RTAX_FASTOPEN_NO_COOKIE RTA_VIA FRA_OIFNAME FRA_PROTOCOL FRA_IP_PROTO FRA_SPORT_RANGE FRA_DPORT_RANGE RTA_TTL_PROPAGATE IFA_FLAGS IP_MULTICAST_ALL LWTUNNEL_ENCAP_MPLS LWTUNNEL_ENCAP_ILA NET_LINUX_IF_H_COLLISION LIBIPVS_NETLINK IPVS_DEST_ATTR_ADDR_FAMILY IPVS_SYNCD_ATTRIBUTES IPVS_64BIT_STATS IPVS_TUN_TYPE IPVS_TUN_CSUM IPVS_TUN_GRE VRRP_VMAC VRRP_IPVLAN IFLA_LINK_NETNSID CN_PROC SOCK_NONBLOCK SOCK_CLOEXEC O_PATH GLOB_BRACE INET6_ADDR_GEN_MODE VRF SO_MARK SCHED_RT SCHED_RESET_ON_FORK
    ubuntu@hap1:~$ 
    ```
    ```console
    ubuntu@hap2:~$ sudo keepalived -v
    Keepalived v2.0.19 (10/19,2019)
    
    Copyright(C) 2001-2019 Alexandre Cassen, <acassen@gmail.com>
    
    Built with kernel headers for Linux 5.4.166
    Running on Linux 5.4.0-155-generic #172-Ubuntu SMP Fri Jul 7 16:10:02 UTC 2023
    
    configure options: --build=x86_64-linux-gnu --prefix=/usr --includedir=${prefix}/include --mandir=${prefix}/share/man --infodir=${prefix}/share/info --sysconfdir=/etc --localstatedir=/var --disable-silent-rules --libdir=${prefix}/lib/x86_64-linux-gnu --runstatedir=/run --disable-maintainer-mode --disable-dependency-tracking --with-kernel-dir=debian/ --enable-snmp --enable-sha1 --enable-snmp-rfcv2 --enable-snmp-rfcv3 --enable-dbus --enable-json --enable-bfd --enable-regex build_alias=x86_64-linux-gnu CFLAGS=-g -O2 -fdebug-prefix-map=/build/keepalived-QKJBdn/keepalived-2.0.19=. -fstack-protector-strong -Wformat -Werror=format-security LDFLAGS=-Wl,-Bsymbolic-functions -Wl,-z,relro CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=2
    
    Config options:  NFTABLES LVS REGEX VRRP VRRP_AUTH JSON BFD OLD_CHKSUM_COMPAT FIB_ROUTING SNMP_V3_FOR_V2 SNMP_VRRP SNMP_CHECKER SNMP_RFCV2 SNMP_RFCV3 DBUS
    
    System options:  PIPE2 SIGNALFD INOTIFY_INIT1 VSYSLOG EPOLL_CREATE1 IPV4_DEVCONF IPV6_ADVANCED_API LIBNL3 RTA_ENCAP RTA_EXPIRES RTA_NEWDST RTA_PREF FRA_SUPPRESS_PREFIXLEN FRA_SUPPRESS_IFGROUP FRA_TUN_ID RTAX_CC_ALGO RTAX_QUICKACK RTEXT_FILTER_SKIP_STATS FRA_L3MDEV FRA_UID_RANGE RTAX_FASTOPEN_NO_COOKIE RTA_VIA FRA_OIFNAME FRA_PROTOCOL FRA_IP_PROTO FRA_SPORT_RANGE FRA_DPORT_RANGE RTA_TTL_PROPAGATE IFA_FLAGS IP_MULTICAST_ALL LWTUNNEL_ENCAP_MPLS LWTUNNEL_ENCAP_ILA NET_LINUX_IF_H_COLLISION LIBIPVS_NETLINK IPVS_DEST_ATTR_ADDR_FAMILY IPVS_SYNCD_ATTRIBUTES IPVS_64BIT_STATS IPVS_TUN_TYPE IPVS_TUN_CSUM IPVS_TUN_GRE VRRP_VMAC VRRP_IPVLAN IFLA_LINK_NETNSID CN_PROC SOCK_NONBLOCK SOCK_CLOEXEC O_PATH GLOB_BRACE INET6_ADDR_GEN_MODE VRF SO_MARK SCHED_RT SCHED_RESET_ON_FORK
    ubuntu@hap2:~$ 
    ```

### Настройка `KeepAlived`
  * ❗️Выполняем настройку на 2х ВМ
  * Редактируем конфигурационный файл [`/etc/keepalived/keepalived.conf`](config_files/keepalived.conf). Мы будем настраивать в конфигурации `BACKUP-BACKUP`, где все сервера равнозначны и нет предпочтительных
    ```bash
    sudo vim /etc/keepalived/keepalived.conf
    ```

  * 

***
### Настройка службы `KeepAlived` в ОС
  * ❗️Выполняем настройку на 2х ВМ
  * Разрешаем запуск службы `KeepAlived` при загрузке ОС
    ```bash
    sudo systemctl enable keepalived
    ```
  * Запускаем службу `KeepAlived` в ОС
    ```bash
    sudo systemctl start keepalived
    ```
  * Проверяем статус службы `KeepAlived`
    ```bash
    sudo systemctl status keepalived
    ```
    ```console
  
    ```
    ```console
  
    ```
      
***  
