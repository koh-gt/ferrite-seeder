ferrite-seeder
===============

ferrite-seeder is a crawler for the Ferritecoin network, which exposes a list
of reliable nodes via a built-in DNS server.

Features:
* regularly revisits known nodes to check their availability
* bans nodes after enough failures, or bad behaviour
* accepts nodes down to v3.0.0 to request new IP addresses from,
  but only reports good post-v3.1.3 nodes.
* keeps statistics over (exponential) windows of 2 hours, 8 hours,
  1 day and 1 week, to base decisions on.
* very low memory (a few tens of megabytes) and cpu requirements.
* crawlers run in parallel (by default 192 threads simultaneously).

REQUIREMENTS
------------

```bash
sudo apt-get install build-essential libboost-all-dev libssl-dev libcurl4-openssl-dev libconfig++-dev
```

USAGE
-----

Assuming you want to run a dns seed on dnsseed.example.com, you will
need an authorative NS record in example.com's domain record, pointing
to for example vps.example.com:

$ dig -t NS dnsseed.example.com

;; ANSWER SECTION
dnsseed.example.com.   86400    IN      NS     vps.example.com.

On the system vps.example.com, you can now run dnsseed:

```
./dnsseed -h dnsseed.ferritecoin.com -n vps.ferritecoin.com -m system@ferritecoin.org
```

If you want the DNS server to report SOA records, please provide an
e-mail address (with the @ part replaced by .) using -m.

#### Namecheap
A Record     vps        118.8.8.8              Automatic
NS Record    dnsseed    vps.ferritecoin.com    Automatic

COMPILING
---------

Compiling will require boost and ssl.  On debian systems, these are provided
by `libboost-dev` and `libssl-dev` respectively.

```bash
make
```

This will produce the `dnsseed` binary.

TESTING
-------

It's sometimes useful to test `dnsseed` locally to ensure it's giving good
output (either as part of development or sanity checking). You can inspect
`dnsseed.dump` to inspect all nodes being tracked for crawling, or you can
issue DNS requests directly. Example:

$ dig @:: -p 15353 dnsseed.example.com
       ^       ^    ^
       |       |    |__ Should match the host (-h) argument supplied to dnsseed
       |       |
       |       |_______ Port number (example uses the user space port; see below)
       |
       |_______________ Explicitly call the DNS server on localhost


RUNNING AS NON-ROOT
-------------------

Typically, you'll need root privileges to listen to port 53 (name service).

One solution is using an iptables rule (Linux only) to redirect it to
a non-privileged port:

$ iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 15353

If properly configured, this will allow you to run dnsseed in userspace, using
the -p 15353 option.

Another solution is allowing a binary to bind to ports < 1024 with setcap (IPv6 access-safe)

$ setcap 'cap_net_bind_service=+ep' /path/to/dnsseed
