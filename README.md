# Rendu TP4

## Mise en place du Lab

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