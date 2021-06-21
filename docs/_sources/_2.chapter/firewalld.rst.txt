##################################
firewalld環境設定
##################################

1. インバウンド通信／アウトバウンド通信


+ インバウンド通信(以下、INPUT)
    外部から内部への通信のことです。(外から来る通信)
+ アウトバウンド通信(以下、OUTPUT)
    内部から外部への通信のことです。(外に出る通信)

.. note::

  firewalldでは、ゾーンに対してはINPUTの設定となるため、OUTPUTの設定は行えません。
  そのため、OUTPUTの設定はiptablesダイレクトルールを利用する必要があります。


2. ゾーン


+ drop 
   全てのパケットを破棄する。内部から外部へのパケットは許可されるが、返信されてきたパケットも破棄してしまうので実質的に通信不可となる。

+ block 
   外部からのパケットは基本的に破棄される。内部からの通信パケットの返信は許可されるようになっている。

+ public 
   デフォルトでは「ssh」と「dhcpv6-client」のみ許可されている。

+ external 
   デフォルトでは「ssh」のみ許可される。ipマスカレードが有効になる。

+ dmz 
   デフォルトでは「ssh」のみ許可されている。

+ work 
   デフォルトでは「dhcpv6-client」と「ipp-client」、「 ssh」が許可される。

+ home 
   デフォルトでは「dhcpv6-client」と「ipp-client」と「mdns」と「samba-client」、「ssh」が許可される。

+ internal 
   デフォルトでは「dhcpv6-client」と「ipp-client」と「mdns」と「samba-client」、「ssh」が許可される。

+ trusted 
   全てのパケットが許可される。


************************
基本操作
************************


サービス自動起動の有効化/無効化
-------------------------------

::

 # systemctl enable firewalld.service
 # systemctl disable firewalld.service


サービス起動/停止/状態確認
--------------------------

::

  # systemctl start firewalld.service
  # systemctl stop firewalld.service
  # systemctl status firewalld.service
  

************************
ゾーン操作
************************


アクティブゾーンの確認
--------------------------------

::

 # firewall-cmd --get-active-zones
 
 public
  interfaces: enp0s3


デフォルトゾーンの確認
----------------------

::

 # firewall-cmd --get-default-zone
 
 public
      

Activeゾーンポリシーの確認
--------------------------

::

  # firewall-cmd --list-all
  
  public (active)
    target: default
    icmp-block-inversion: no
    interfaces: enp0s3
    sources: 
    services: dhcpv6-client ssh
    ports: 514/tcp
    protocols: 
    masquerade: no
    forward-ports: 
    source-ports: 
    icmp-blocks: 
    rich rules: 
 

 

任意のゾーン設定の確認
----------------------

::

  # firewall-cmd --zone=block --list-all
  
  block
    target: %%REJECT%%
    icmp-block-inversion: no
    interfaces: 
    sources: 
    services: 
    ports: 
    protocols: 
    masquerade: no
    forward-ports: 
    source-ports: 
    icmp-blocks: 
    rich rules: 
  
::

  # firewall-cmd --list-all-zones
  すべてのゾーン

ゾーンの新規追加
----------------

::

 # firewall-cmd --new-zone Original --permanent
 success

 # firewall-cmd --reload
 success

::

 # firewall-cmd --add-service=ssh --zone=OriginalZ --permanent
 success

 # cat /etc/firewalld/zones/Original.xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
 <service name="ssh"/>
 </zone>

 # firewall-cmd --reload

::

 # firewall-cmd --list-all --zone=Original
 Original
   target: default
   icmp-block-inversion: no
   interfaces:
   sources:
   services: ssh
   ports:
   protocols:
   masquerade: no
   forward-ports:
   source-ports:
   icmp-blocks:
   rich rules:
  
      

インターフェイスの割り当てゾーン変更
--------------------------------------

::

 # firewall-cmd --zone=trusted --change-interface=enp0s3
 # nmcli connection modify enp0s3 connection.zone internal　--permanent

::

 # cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 
 TYPE=Ethernet
 PROXY_METHOD=none
 BROWSER_ONLY=no
 BOOTPROTO=dhcp
 DEFROUTE=yes
 IPV4_FAILURE_FATAL=no
 IPV6INIT=yes
 IPV6_AUTOCONF=yes
 IPV6_DEFROUTE=yes
 IPV6_FAILURE_FATAL=no
 IPV6_ADDR_GEN_MODE=stable-privacy
 NAME=enp0s3
 UUID=e08ab9aa-6ae3-496b-80a6-13b7af76b852
 DEVICE=enp0s3
 ONBOOT=yes
 IPADDR=10.0.2.5
 PREFIX=24
      

ゾーンへサービスを追加/削除
--------------------------------------

::

 # firewall-cmd --zone=public --add-service=http  --permanent
 # firewall-cmd --zone=public --remove-service =http  --permanent
 # firewall-cmd --reload


**************************
サービスの操作
**************************


指定できるサービス一覧
----------------------

::

 # firewall-cmd --get-services
 
 RH-Satellite-6 RH-Satellite-6-capsule amanda-client 
 amanda-k5-client amqp amqps apcupsd audit bacula bacula-client 
 bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc 
 ceph ceph-mon cfengine condor-collector ctdb dhcp dhcpv6 dhcpv6-client 
 distcc dns docker-registry docker-swarm dropbox-lansync elasticsearch 
 etcd-client etcd-server finger freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust 
 ftp ganglia-client ganglia-master git gre high-availability 
 http https imap imaps ipp ipp-client ipsec irc ircs iscsi-target 
 isns jenkins kadmin kerberos kibana klogin kpasswd kprop kshell 
 ldap ldaps libvirt libvirt-tls lightning-network llmnr managesieve 
 matrix mdns minidlna mongodb mosh mountd mqtt mqtt-tls ms-wbt mssql 
 murmur mysql nfs nfs3 nmea-0183 nrpe ntp nut openvpn ovirt-imageio 
 ovirt-storageconsole ovirt-vmconsole plex pmcd pmproxy pmwebapi pmwebapis 
 pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster 
 quassel radius redis rpc-bind rsh rsyncd rtsp salt-master samba samba-client samba-dc 
 sane sip sips slp smtp smtp-submission smtps snmp snmptrap spideroak-lansync 
 squid ssh steam-streaming svdrp svn syncthing syncthing-gui synergy syslog syslog-tls 
 telnet tftp tftp-client tinc tor-socks transmission-client upnp-client vdsm vnc-server 
 wbem-http wbem-https wsman wsmans xdmcp xmpp-bosh xmpp-client xmpp-local xmpp-server 
 zabbix-agent zabbix-server

サービス定義ファイル
--------------------

::

 ftpサービス
 
 # cat /usr/lib/firewalld/services/ftp.xml
 
 <?xml version="1.0" encoding="utf-8"?>
 <service>
   <short>FTP</short>
   <description>FTP is a protocol used for remote file transfer. If you plan to make your FTP server publicly available, enable this option. You need the vsftpd package installed for this option to be useful.</description>
   <port protocol="tcp" port="21"/>
   <module name="nf_conntrack_ftp"/>
 </service>


サービスの新規追加
------------------

::

 # firewall-cmd --permanent --new-service hoge
 
 # cat /etc/firewalld/services/hoge.xml
 <?xml version="1.0" encoding="utf-8"?>
 <service>
 </service>


サービスの削除
--------------

::

 # firewall-cmd --permanent --delete-service=hoge
 

ゾーンの設定確認
----------------

::

 # firewall-cmd --list-services --zone=public
 # firewall-cmd --list-services --zone=public --permanent

 

ゾーンへの許可サービスの追加
----------------------------

::

 # firewall-cmd --add-service=ftp --zone=public 
 # firewall-cmd --add-service=ftp --zone=public --permanent
 # firewall-cmd --reload
 

ゾーンへの許可サービスの削除
----------------------------

::

 # firewall-cmd --remove-service=ftp --zone=public
 # firewall-cmd --remove-service=ftp --zone=public --permanent
 # firewall-cmd --reload
 

**************************
ポート番号の操作
**************************

ポート番号と名前の紐づけ定義ファイル
------------------------------------

::

 # cat /etc/services | grep postgres
 
 postgres        5432/tcp        postgresql      # POSTGRES
 postgres        5432/udp        postgresql      # POSTGRES

 

ポート番号設定の確認
--------------------

::

 # firewall-cmd --list-ports --zone=public
 
 514/tcp
 
 # firewall-cmd --list-ports --zone=public --permanent
 
 514/tcp
 

許可ポート番号の追加
--------------------

::

 # firewall-cmd --add-port=postgres/tcp --add-port=60000/udp
 # firewall-cmd --add-port=postgres/tcp --add-port=60000/udp --permanent
 # firewall-cmd --reload
 

許可ポート番号の削除
--------------------

::

 # firewall-cmd --remove-port=postgres/tcp --remove-port=60000/udp
 # firewall-cmd --remove-port=8080/tcp --remove-port=60000/udp --permanent
 # firewall-cmd --reload
 

**************************
ソースIPアドレスの操作
**************************

ソースIPアドレス設定の確認
--------------------------

::

 # firewall-cmd --list-sources --zone=public
 # firewall-cmd --list-sources --zone=public --permanent
 

許可ソースIPアドレスの追加
--------------------------

::

 # firewall-cmd --add-source=192.168.0.0/24 --zone=public
 # firewall-cmd --add-source=192.168.0.0/24 --zone=public --permanent
 # firewall-cmd --reload
 

拒否ソースIPアドレスの追加
--------------------------

::

 # firewall-cmd --add-source=192.168.11.0/24 --zone=drop
 # firewall-cmd --add-source=192.168.11.0/24 --zone=drop --permanent
 # firewall-cmd --reload
 

許可/拒否IPアドレスの削除
-------------------------

::

 # firewall-cmd --remove-source=192.168.11.0/24 --zone=drop
 # firewall-cmd --remove-source=192.168.11.0/24 --zone=drop --permanent
 # firewall-cmd --reload
 


**************************
OUTPUTダイレクトルール
**************************

現行設定の確認
--------------

::

 # firewall-cmd --direct --get-all-rules
 # firewall-cmd --direct --get-all-rules --permanent
 


ダイレクトルールの追加
----------------------

::

 # firewall-cmd --direct --add-rule ipv4 filter OUTPUT 0 -m state --state NEW -d 192.168.11.2 -j DROP
 # firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -m state --state NEW -d 192.168.11.2 -j DROP
 # firewall-cmd --reload
 



ダイレクトルールの削除
----------------------

::

 # firewall-cmd --direct --remove-rule ipv4 filter OUTPUT 0 -m state --state NEW -d 192.168.11.2 -j DROP
 # firewall-cmd --permanent --direct --remove-rule ipv4 filter OUTPUT 0 -m state --state NEW -d 192.168.11.2 -j DROP
 # firewall-cmd --reload
 


