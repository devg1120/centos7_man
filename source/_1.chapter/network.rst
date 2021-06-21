##################################
ネットワーク環境設定
##################################

************************
デバイス接続レイヤの操作
************************


接続プロファイル一覧
--------------------

::

 # nmcli connection
 # nmcli c
 
 NAME    UUID                                  TYPE      DEVICE
 eth0    dc175057-25d3-4817-a66f-dd59a81913c4  ethernet  enp0s3
 
※ NAME/DEVICE 同一名である必要はない



デバイス一覧
------------

::

  # nmcli device
  # nmcli d
  
  DEVICE  TYPE      STATE     CONNECTION
  enp0s3  ethernet  接続済み  eth0
  lo      loopback  管理無し  --
  



接続プロファイルの接続と切断
--------------------------------

::

 nmcli connection up|down '接続プロファイル名'
 
 # nmcli c down enp0s3

 # nmcli c up enp0s3
                 enp0s3:接続名


デバイスの接続と切断
----------------------

::

  nmcli device connect connect|disconnect 'デバイス名'
      
   (例) nmcli d c disconnect enp5s0
      

接続プロファイル追加
--------------------

::

  nmcli connection add type 'タイプ名' ifname 'デバイス名' con-name '接続プロファイル名'
  
  # nmcli c a type ethernet ifname enp5s0 con-name eth1
 
 
※　これで /etc/sysconfig/network-scripts に ifcfg-eth1 というファイルが作成される。
 

接続プロファイル削除
--------------------

::

  nmcli connection delete '接続プロファイル名'
  
  # nmcli c d eth0
  


IPレイヤ環境の設定
------------------
- IPアドレスの設定

::

  nmcli connection modify '接続プロファイル名' ipv4.method manual ipv4.addresses IPアドレス/ネットマスク
      
  # nmcli c m eth0 ipv4.method manual ipv4.addresses 192.168.1.2/24


- ゲートウェイの設定

::

  nmcli connection modify '接続プロファイル名' ipv4.gateway 'ゲートウェイアドレス'
      
  # nmcli c m eth0 ipv4.gateway 192.168.1.1
  


- DNSサーバの設定

::

  nmcli connection modify '接続プロファイル名' ipv4.dns 'DNSサーバのアドレス'
      
  # nmcli c m eth0 ipv4.dns 192.168.1.1

- 自動接続の設定

::

  nmcli connection modify '接続プロファイル名' connection.autoconnect yes
      
  # nmcli c m eth0 connection.autoconnect yes
  
  


network-scripts確認
-------------------

::

 # ls -l /etc/sysconfig/network-scripts/ifcfg-*
 -rw-r--r--. 1 root root 308  6月 11 10:26 /etc/sysconfig/network-scripts/ifcfg-enp0s3
 -rw-r--r--. 1 root root 254  5月 22  2020 /etc/sysconfig/network-scripts/ifcfg-lo

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
 NAME=enp0s3                                       接続名
 UUID=e08ab9aa-6ae3-496b-80a6-13b7af76b852
 DEVICE=enp0s3　　　　　　　　　　　　　　　　　　　デバイス名
 ONBOOT=yes
 IPADDR=10.0.2.5
 PREFIX=24
 

**************************
IPネットワークレイヤの操作
**************************


NETWORKコマンド(CentOS6/CentOS7)
--------------------------------

+--------------+--------------------------+
|(CentOS6)     |  (CentOS7)               |
|#net-tools    |  #iproute2               |
+==============+==========================+
|ifconfig      |  ip a(addr), ip l(link)  |
+--------------+--------------------------+
|route         |  ip r(route)             |
+--------------+--------------------------+
|netstat       |  ss                      |
+--------------+--------------------------+
|netstat -i    |  ip -s l(link)           |
+--------------+--------------------------+
|arp           |  ip n(neighbor)          |
+--------------+--------------------------+

リンク状態の確認
----------------

::

 # ip a show dev eth0
 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
     link/ether 52:54:00:41:c6:32 brd ff:ff:ff:ff:ff:ff
     inet 192.168.122.70/24 brd 192.168.122.255 scope global eth0
        valid_lft forever preferred_lft forever
     inet6 fe80::5054:ff:fe41:c632/64 scope link 
        valid_lft forever preferred_lft forever

リンクの接続と切断
-------------------

::

 # ip l set eth1 up
 # ip l set eth1 down


ルーティングテーブルの確認
--------------------------

::

 # ip r
 default via 192.168.122.1 dev eth0  proto static  metric 1024 
 192.168.122.0/24 dev eth0  proto kernel  scope link  src 192.168.122.70 
 192.168.122.0/24 dev eth1  proto kernel  scope link  src 192.168.122.69 

デフォルトゲートウェイの追加、削除
----------------------------------

::

 # ip route add default via 192.168.122.1
 # ip route del default via 192.168.122.1


デバイスごとのパケット処理数
----------------------------

::

 # ip -s l
 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT 
     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
     RX: bytes  packets  errors  dropped overrun mcast   
     82260      1040     0       0       0       0      
     TX: bytes  packets  errors  dropped carrier collsns 
     82260      1040     0       0       0       0      
 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
     link/ether 52:54:00:41:c6:32 brd ff:ff:ff:ff:ff:ff
     RX: bytes  packets  errors  dropped overrun mcast   
     3861081    6396     0       0       0       0      
     TX: bytes  packets  errors  dropped carrier collsns 
     237222     1733     0       0       0       0      
 3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
     link/ether 52:54:00:a2:e8:73 brd ff:ff:ff:ff:ff:ff
     RX: bytes  packets  errors  dropped overrun mcast   
     1478655    2098     0       0       0       0      
     TX: bytes  packets  errors  dropped carrier collsns 
     20043      126      0       0       0       0  


TCPソケットの状態確認
---------------------

::

 # ss -nat
 State      Recv-Q Send-Q        Local Address:Port          Peer Address:Port 
 LISTEN     0      100               127.0.0.1:25                       *:*     
 LISTEN     0      128                       *:22                       *:*     
 ESTAB      0      0            192.168.122.70:22           192.168.122.1:34328 
 LISTEN     0      100                     ::1:25                      :::*     
 LISTEN     0      128                      :::80                      :::*     
 LISTEN     0      128                      :::22                      :::*  


ARPテーブルの確認
-----------------

::

 # ip n
 192.168.122.71 dev eth0 lladdr 52:54:00:a5:2c:75 REACHABLE
 192.168.122.1 dev eth0 lladdr 02:13:97:c1:47:ec REACHABLE


