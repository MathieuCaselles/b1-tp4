# Rendu TP4

## I - Mise en place du Lab

_client1 ping router1.tp4 sur l'IP 10.1.0.254_

    [root@client1 ~]# ping 10.1.0.254
    PING 10.1.0.254 (10.1.0.254) 56(84) bytes of data.
    64 bytes from 10.1.0.254: icmp_seq=1 ttl=64 time=0.582 ms
    64 bytes from 10.1.0.254: icmp_seq=2 ttl=64 time=0.351 ms
    64 bytes from 10.1.0.254: icmp_seq=3 ttl=64 time=0.495 ms
    64 bytes from 10.1.0.254: icmp_seq=4 ttl=64 time=0.371 ms
    64 bytes from 10.1.0.254: icmp_seq=5 ttl=64 time=0.275 ms
    64 bytes from 10.1.0.254: icmp_seq=6 ttl=64 time=0.384 ms
    ^C
    --- 10.1.0.254 ping statistics ---
    6 packets transmitted, 6 received, 0% packet loss, time 5015ms
    rtt min/avg/max/mdev = 0.275/0.409/0.582/0.103 ms

_server1 ping router1.tp4 sur l'IP 10.2.0.254_

    [root@server1 ~]# ping 10.2.0.254
    PING 10.2.0.254 (10.2.0.254) 56(84) bytes of data.
    64 bytes from 10.2.0.254: icmp_seq=1 ttl=64 time=0.634 ms
    64 bytes from 10.2.0.254: icmp_seq=2 ttl=64 time=0.365 ms
    64 bytes from 10.2.0.254: icmp_seq=3 ttl=64 time=0.393 ms
    64 bytes from 10.2.0.254: icmp_seq=4 ttl=64 time=0.389 ms
    64 bytes from 10.2.0.254: icmp_seq=5 ttl=64 time=0.393 ms
    ^C
    --- 10.2.0.254 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4009ms
    rtt min/avg/max/mdev = 0.365/0.434/0.634/0.103 ms

### Mise en place du routage statique

1. sur router1 :  

        [root@router1 ~]# sudo sysctl -w net.ipv4.conf.all.forwarding=1
        net.ipv4.conf.all.forwarding = 1
        [root@router1 ~]# sudo sysctl  net.ipv4.conf.all.forwarding
        net.ipv4.conf.all.forwarding = 1
        [root@router1 ~]# sudo systemctl disable firewalld
        [root@router1 ~]# ip route show
        10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.254 metric 100
        10.2.0.0/24 dev enp0s9 proto kernel scope link src 10.2.0.254 metric 101

2. sur client1 :  

        [root@client1 ~]# ip route show
        10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.10 metric 100
        10.2.0.0/24 via 10.1.0.254 dev enp0s8 proto static metric 100

3. sur server1 :

        [root@server1 ~]# ip route show
        10.1.0.0/24 via 10.2.0.254 dev enp0s8 proto static metric 100
        10.2.0.0/24 dev enp0s8 proto kernel scope link src 10.2.0.10 metric 100

4. test :

    _client1 ping server1 :_  

        [root@client1 ~]# ping 10.2.0.10
        PING 10.2.0.10 (10.2.0.10) 56(84) bytes of data.
        64 bytes from 10.2.0.10: icmp_seq=1 ttl=63 time=0.630 ms
        64 bytes from 10.2.0.10: icmp_seq=2 ttl=63 time=0.721 ms
        64 bytes from 10.2.0.10: icmp_seq=3 ttl=63 time=0.940 ms
        ^C
        --- 10.2.0.10 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time

    _server1 ping client1 :_  

        [root@server1 ~]# ping 10.1.0.10
        PING 10.1.0.10 (10.1.0.10) 56(84) bytes of data.
        64 bytes from 10.1.0.10: icmp_seq=1 ttl=63 time=0.694 ms
        64 bytes from 10.1.0.10: icmp_seq=2 ttl=63 time=0.752 ms
        64 bytes from 10.1.0.10: icmp_seq=3 ttl=63 time=0.732 ms
        ^C
        --- 10.1.0.10 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2006ms
        rtt min/avg/max/mdev = 0.694/0.726/0.752/0.024 ms

    _traceroute_

        [root@client1 ~]# traceroute 10.2.0.10
        traceroute to 10.2.0.10 (10.2.0.10), 30 hops max, 60 byte packets
        1  10.1.0.254 (10.1.0.254)  0.298 ms  0.197 ms  0.126 ms
        2  10.2.0.10 (10.2.0.10)  0.487 ms !X  0.376 ms !X  0.306 ms !X

## II - Spéléologie réseau

### 1. Arp

#### A. Manip 1

1. Je vide bien la table arp de toute mes machines (la preuve dans le code montrant les autres étapes)

2. client1 :

        [root@client1 ~]# sudo ip neigh flush all
        [root@client1 ~]# ip neigh show
        10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:10 DELAY
    
    La ligne dit que la mac de la machine possédant l'IP 10.1.0.1 est 0a:00:27:00:00:10. C'est à la carte réseau d'hôte car on est conecté à cet vm

3. server1 :

        [root@server1 ~]# sudo ip neigh flush all
        [root@server1 ~]# ip neigh show
        10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:18 REACHABLE
    
    La ligne dit que la mac de la machine possédant l'IP 10.2.0.1 est 0a:00:27:00:00:18. C'est à la carte réseau d'hôte car on est conecté à cet vm

4. client1 :

        [root@client1 ~]# ping 10.2.0.10
        PING 10.2.0.10 (10.2.0.10) 56(84) bytes of data.
        64 bytes from 10.2.0.10: icmp_seq=1 ttl=63 time=16.7 ms
        64 bytes from 10.2.0.10: icmp_seq=2 ttl=63 time=0.577 ms
        64 bytes from 10.2.0.10: icmp_seq=3 ttl=63 time=0.733 ms
        ^C
        --- 10.2.0.10 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2007ms
        rtt min/avg/max/mdev = 0.577/6.006/16.710/7.569 ms
        [root@client1 ~]# ip neigh show
        10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:10 REACHABLE
        10.1.0.254 dev enp0s8 lladdr 08:00:27:7b:aa:aa REACHABLE
    
    En fait client1 a du passer par l'adresse 10.1.0.254 qui est l'adresse passerelle vers 10.2.0.0/24 grace à router 1. Client1 a donc pu noter ajouter sa mac dans sa table arp.
    
5. server1 :

        [root@server1 ~]# ip neigh show
        10.2.0.254 dev enp0s8 lladdr 08:00:27:96:68:53 STALE
        10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:18 DELAY
    
    En fait server1 a reçu sa demande par 10.2.0.254 qui est l'adresse passerelle vers 10.1.0.0/24 grace à router 1. Server1 a donc pu noter ajouter sa mac dans sa table arp.

#### B. Manip 2

1. Je vide bien la table arp de toute mes machines (la preuve dans le code montrant les autres étapes)

2. router1 :

        [root@router1 ~]# sudo ip neigh flush all
        [root@router1 ~]# ip neigh show
        10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:10 DELAY

        La ligne dit que la mac de la machine possédant l'IP 10.1.0.1 est 0a:00:27:00:00:10. C'est à la carte réseau d'hôte car on est conecté à cet vm

3. client1 :

        [root@client1 ~]# sudo ip neigh flush all
        [root@client1 ~]# ping 10.2.0.10
        PING 10.2.0.10 (10.2.0.10) 56(84) bytes of data.
        64 bytes from 10.2.0.10: icmp_seq=1 ttl=63 time=0.712 ms
        64 bytes from 10.2.0.10: icmp_seq=2 ttl=63 time=0.716 ms
        64 bytes from 10.2.0.10: icmp_seq=3 ttl=63 time=0.667 ms
        ^C
        --- 10.2.0.10 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2002ms
        rtt min/avg/max/mdev = 0.667/0.698/0.716/0.030 ms

4. router1 :

        [root@router1 ~]# ip neigh show
        10.2.0.10 dev enp0s9 lladdr 08:00:27:91:25:ce STALE
        10.1.0.10 dev enp0s8 lladdr 08:00:27:2c:64:9d STALE
        10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:10 DELAY

    Étant donné que router1 transmet le message de client1 à server1, il accède directement à leur ip et donc enregistre leur mac dans sa table arp.

#### C. Manip 3

1. Je vide bien la table arp de toute mes machines

2. 

        PS C:\WINDOWS\system32> arp -a

        Interface : 192.168.102.1 --- 0xf
        Adresse Internet      Adresse physique      Type
        192.168.102.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique

        Interface : 10.1.0.1 --- 0x10
        Adresse Internet      Adresse physique      Type
        10.1.0.10             08-00-27-2c-64-9d     dynamique
        10.1.0.254            08-00-27-7b-aa-aa     dynamique
        10.1.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique

        Interface : 192.168.0.11 --- 0x13
        Adresse Internet      Adresse physique      Type
        192.168.0.14          2c-54-91-12-83-5a     dynamique
        192.168.0.254         34-27-92-43-29-24     dynamique
        192.168.0.255         ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        255.255.255.255       ff-ff-ff-ff-ff-ff     statique

        Interface : 192.168.181.1 --- 0x17
        Adresse Internet      Adresse physique      Type
        192.168.181.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique

        Interface : 10.2.0.1 --- 0x18
        Adresse Internet      Adresse physique      Type
        10.2.0.10             08-00-27-91-25-ce     dynamique
        10.2.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        239.255.255.250       01-00-5e-7f-ff-fa     statique
        PS C:\WINDOWS\system32> arp -d
        PS C:\WINDOWS\system32> arp -a

        Interface : 192.168.102.1 --- 0xf
        Adresse Internet      Adresse physique      Type
        192.168.102.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique

        Interface : 10.1.0.1 --- 0x10
        Adresse Internet      Adresse physique      Type
        10.1.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique

        Interface : 192.168.0.11 --- 0x13
        Adresse Internet      Adresse physique      Type
        192.168.0.254         34-27-92-43-29-24     dynamique
        192.168.0.255         ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        255.255.255.255       ff-ff-ff-ff-ff-ff     statique

        Interface : 192.168.181.1 --- 0x17
        Adresse Internet      Adresse physique      Type
        192.168.181.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique

        Interface : 10.2.0.1 --- 0x18
        Adresse Internet      Adresse physique      Type
        10.2.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        PS C:\WINDOWS\system32> arp -a

        Interface : 192.168.102.1 --- 0xf
        Adresse Internet      Adresse physique      Type
        192.168.102.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique

        Interface : 10.1.0.1 --- 0x10
        Adresse Internet      Adresse physique      Type
        10.1.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique

        Interface : 192.168.0.11 --- 0x13
        Adresse Internet      Adresse physique      Type
        192.168.0.254         34-27-92-43-29-24     dynamique
        192.168.0.255         ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique
        255.255.255.255       ff-ff-ff-ff-ff-ff     statique

        Interface : 192.168.181.1 --- 0x17
        Adresse Internet      Adresse physique      Type
        192.168.181.255       ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique

        Interface : 10.2.0.1 --- 0x18
        Adresse Internet      Adresse physique      Type
        10.2.0.255            ff-ff-ff-ff-ff-ff     statique
        224.0.0.22            01-00-5e-00-00-16     statique
        224.0.0.251           01-00-5e-00-00-fb     statique
        224.0.0.252           01-00-5e-00-00-fc     statique

    Je ne vois aucun changement (j'ai attendu  2 min)

    #### 4. Manip 4

    1. Je vide bien la table arp de toutes mes machines

    2. 

        [root@client1 ~]# curl google.com
        curl: (6) Could not resolve host: google.com; Unknown error

### 2. Wireshark


#### A. Interception d'ARP et ping

1. router 1 :

        [root@router1 ~]# sudo tcpdump -i enp0s9 -w ping.pcap
        tcpdump: listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes

2. client 1 :

    _Je vide la table arp_  

        [root@client1 ~]# ping -c 4 10.2.0.10
        PING 10.2.0.10 (10.2.0.10) 56(84) bytes of data.
        64 bytes from 10.2.0.10: icmp_seq=1 ttl=63 time=0.839 ms
        64 bytes from 10.2.0.10: icmp_seq=2 ttl=63 time=0.706 ms
        64 bytes from 10.2.0.10: icmp_seq=3 ttl=63 time=0.657 ms
        64 bytes from 10.2.0.10: icmp_seq=4 ttl=63 time=0.705 ms

3. router 1 :

        ^C11 packets captured
        11 packets received by filter
        0 packets dropped by kernel
        [root@router1 ~]# ls
        anaconda-ks.cfg  ping.pcap

        PS C:\Users\PHOEN> scp root@10.2.0.254:ping.pcap C:\Users\PHOEN\Desktop\
        root@10.2.0.254's password:
        Permission denied, please try again.
        root@10.2.0.254's password:
        Permission denied, please try again.
        root@10.2.0.254's password:
        ping.pcap                                                                             100% 1169   167.8KB/s

4. sur l'hôte :

        1	0.000000	10.2.0.1	224.0.0.251	MDNS	83	Standard query 0x0000 PTR _sleep-proxy._udp.local, "QM" question
        2	152.954206	10.1.0.10	10.2.0.10	ICMP	98	Echo (ping) request  id=0x0d27, seq=1/256, ttl=63 (reply in 3)
        3	152.954498	10.2.0.10	10.1.0.10	ICMP	98	Echo (ping) reply    id=0x0d27, seq=1/256, ttl=64 (request in 2)
        4	153.955246	10.1.0.10	10.2.0.10	ICMP	98	Echo (ping) request  id=0x0d27, seq=2/512, ttl=63 (reply in 5)
        5	153.955581	10.2.0.10	10.1.0.10	ICMP	98	Echo (ping) reply    id=0x0d27, seq=2/512, ttl=64 (request in 4)
        6	154.956310	10.1.0.10	10.2.0.10	ICMP	98	Echo (ping) request  id=0x0d27, seq=3/768, ttl=63 (reply in 7)
        7	154.956606	10.2.0.10	10.1.0.10	ICMP	98	Echo (ping) reply    id=0x0d27, seq=3/768, ttl=64 (request in 6)
        8	155.957361	10.1.0.10	10.2.0.10	ICMP	98	Echo (ping) request  id=0x0d27, seq=4/1024, ttl=63 (reply in 9)
        9	155.957709	10.2.0.10	10.1.0.10	ICMP	98	Echo (ping) reply    id=0x0d27, seq=4/1024, ttl=64 (request in 8)
        10	157.962952	PcsCompu_91:25:ce	PcsCompu_96:68:53	ARP	60	Who has 10.2.0.254? Tell 10.2.0.10
        11	157.962972	PcsCompu_96:68:53	PcsCompu_91:25:ce	ARP	42	10.2.0.254 is at 08:00:27:96:68:53


#### B. Interception d'une communication netcat

_Avec le port ouvert :_

    1	0.000000	10.1.0.10	10.2.0.10	TCP	74	41728 → 8888 [SYN] Seq=0 Win=29200 Len=0 MSS=1460 SACK_PERM=1 TSval=3430740 TSecr=0 WS=64
    2	0.000429	10.2.0.10	10.1.0.10	ICMP	102	Destination unreachable (Host administratively prohibited)
    3	5.002688	PcsCompu_91:25:ce	PcsCompu_96:68:53	ARP	60	Who has 10.2.0.254? Tell 10.2.0.10
    4	5.002707	PcsCompu_96:68:53	PcsCompu_91:25:ce	ARP	42	10.2.0.254 is at 08:00:27:96:68:53

_Avec le port fermé :_

    Bah ça a rien capturé...