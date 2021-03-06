
Table of Contents
=================

  * [vpn-slice](#vpn-slice)
    * [Who this is for](#who-this-is-for)
    * [Requirements](#requirements)
    * [Usage](#usage)
  * [Inspiration](#inspiration)
  * [License](#license)
    * [TODO](#todo)

# vpn-slice

This is a replacement for the
[`vpnc-script`](http://www.infradead.org/openconnect/vpnc-script.html)
used by [OpenConnect](http://www.infradead.org/openconnect) or
[VPNC](https://www.unix-ag.uni-kl.de/~massar/vpnc).

Instead of trying to copy the behavior of the standard Cisco VPN clients,
which normally reroute **all** your network traffic through the VPN,
this one tries to minimize your contact with an intrusive corporate VPN.
This is also known as a "split-tunnel" VPN, since it splits your traffic
between the VPN tunnel and your normal network interfaces.

`vpn-slice` makes it easy to set up a split-tunnel VPN:

* It only routes traffic for **specific hosts or subnets** through the VPN.
* It automatically looks up named hosts, using the VPN's DNS servers,
  and adds entries for them to your `/etc/hosts` (which it cleans up
  after VPN disconnection), however it **does not otherwise alter your
  `/etc/resolv.conf` at all**.

## Who this is for

If you are using a VPN to route *all* your traffic for privacy reasons
or to avoid censorship in repressive countries), then you **do not want
to use this**.

The purpose of this tool is almost the opposite; it makes it easy to
connect to a VPN while **minimizing** the traffic that passes over the
VPN.

This is for people who have to connect to the high-security VPNs of
corporations or other bureaucracies (which monitor and filter and
otherwise impede network traffic), and thus wish to route as little
traffic as possible through those VPNs.

## Requirements

* Python 3.3+
* [`dig`](https://en.wikipedia.org/wiki/Dig_(command)) (DNS lookup
  tool; tested with v9.9.5)
* Linux OS (the [`iproute2`](https://en.wikipedia.org/wiki/iproute2)
  and [`iptables`](http://en.wikipedia.org/wiki/iptables) utilities
  are used for all routing setup)

You can install the latest build with `pip` (make sure you are using
the Python 3.x version, usually invoked with `pip3`):

    $ pip3 install https://github.com/dlenski/vpn-slice/archive/master.zip

## Usage

You should specify `vpn-slice` as your connection script with
`openconnect` or `vpnc`. It has been tested with both OpenConnect
v7.06 and vpnc v0.5.3.

For example:

    $ sudo openconnect gateway.bigcorp.com -u user1234 \
        -s 'vpn-slice 192.168.1.0/24 hostname1 hostname2'
    $ cat /etc/hosts
    ...
    192.168.1.1 dnsmain00 dnsmain00.bigcorp.com			# vpn-slice-tun0 AUTOCREATED
    192.168.1.2 dnsbackup2 dnsmain2.bigcorp.com			# vpn-slice-tun0 AUTOCREATED
    192.168.1.57 hostname1 hostname1.bigcorp.com		# vpn-slice-tun0 AUTOCREATED
    192.168.1.173 hostname1 hostname1.bigcorp.com		# vpn-slice-tun0 AUTOCREATED

or

    # With vpnc, you *must* specify an absolute path for the disconnect hook
    # to work correctly, due to a bug which I reported:
    #   http://lists.unix-ag.uni-kl.de/pipermail/vpnc-devel/2016-August/004199.html
    $ sudo vpnc config_file \
           --script '/path/to/vpn-slice 192.168.1.0/24 hostname1 hostname2'

There are many command-line options to alter the behavior of
`vpn-slice`; try `vpn-slice --help` to show them all.

Running with `--verbose` makes it explain what it is doing, while running with
`--dump` shows the environment variables passed in by the caller.

# Inspiration

[**@jagtesh**](https://github.com/jagtesh)'s
[split-tunnelling tutorial gist](https://gist.github.com/jagtesh/5531300) taught me the
basics of how to set up a split-tunnel VPN by wrapping the standard `vpnc-script`.

[**@apenwarr**](https://github.com/apenwarr)'s
[sshuttle](https://github.com/apenwarr/sshuttle) has the excellent
`--auto-hosts` and `--seed-hosts` options. These inspired the
automatic host lookup feature.

# License

GPLv3 or later.

## TODO

* Fix timing issues
* IPv6 support?
* Support OSes other than Linux?
