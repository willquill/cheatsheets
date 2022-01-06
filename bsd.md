# BSD Cheatsheet

## FreeBSD

### How To Start / Stop / Restart Network and Routing Services ([source](https://www.cyberciti.biz/tips/freebsd-how-to-start-restart-stop-network-service.html))

To start FreeBSD network service:

`/etc/rc.d/netif start`

To stop FreeBSD network service:

`/etc/rc.d/netif stop`

To restart FreeBSD network service:

`/etc/rc.d/netif restart`

Manual method using ifconfig

To stop network card (NIC) on-fly:

`ifconfig network-interface down`

To start network card (NIC) on fly:

`ifconfig network-interface up`

To list down network interface:

`ifconfig -d`

To list up network interface:

`ifconfig -u`

FreeBSD Update / restart routing tables / service

It is also necessary to update routing table after restating networking service, enter:

`/etc/rc.d/routing restart`

How do I restart network service over ssh session?

You need to type the commands as follows in order to avoid problems:

`/etc/rc.d/netif restart && /etc/rc.d/routing restart`
