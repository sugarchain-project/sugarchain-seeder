SUGARCHAIN-SEEDER
==============

sugarchain-seeder is a crawler for the Sugarchain network, which exposes a list of reliable nodes via a built-in DNS server.

Features:
* regularly revisits known nodes to check their availability
* bans nodes after enough failures, or bad behaviour
* accepts nodes from Yumekawa v0.16.3.x (protocol version `70015`) to request new IP addresses.
* keeps statistics over (exponential) windows of 2 hours, 8 hours, 1 day and 1 week, to base decisions on.
* very low memory (a few tens of megabytes) and cpu requirements.
* crawlers run in parallel (by default `96` threads simultaneously).

INSTALLATION
------------

```bash
cd && \
sudo apt-get update && \
sudo apt-get install -y build-essential libboost-all-dev libssl-dev && \
git clone https://github.com/sugarchain-project/sugarchain-seeder.git && \
cd sugarchain-seeder && \
make -j$(nproc)
```

REMOVE
------

```
sudo killall dnsseed ; \
cd && sudo rm -f dnsseed.dat && sudo rm -f dnsseed.dump && sudo rm -f dnsstats.log && \
cd && cd sugarchain-seeder/ && sudo rm -f dnsseed.dat && sudo rm -f dnsseed.dump && sudo rm -f dnsstats.log && \
cd && rm -rf sugarchain-seeder/ && \
sudo netstat -nulp | grep 53
```

USAGE
-----

On the system `1ns.sugarchain.info`, you can now run dnsseed with root privileged to use port 53 (`UDP`)
```bash
sudo ./dnsseed -h 1seed.sugarchain.info -n 1ns.sugarchain.info -m cryptozeny.gmail.com
```

Assuming you want to run a dns seed on `1seed.sugarchain.info`, you will need an authorative NS record in `sugarchain.org`'s domain record, pointing to for example `1ns.sugarchain.info`. Set DNS TTL as `86400 (1 Day)` or Automatic.

```bash
dig -t NS 1seed.sugarchain.info
```

```
;; ANSWER SECTION:
1seed.sugarchain.info. 21599 IN	NS	1ns.sugarchain.info.
```

If you want the DNS server to report SOA records, please provide an e-mail address (with the `@` part replaced by `.`) using `-m`.

Check if port 53 opened
```bash
sudo netstat -nulp | grep 53

udp6       0      0 :::53                   :::*                                10949/dnsseed
```

Check if it works
```bash
watch -n1 dig +short -t A 1seed.sugarchain.info @1.1.1.1
```

Run Sugarchain node on another computer
```bash
./src/sugarchaind -dns=1 -dnsseed=1 -forcednsseed=1 -listen=1
```

CRON
----
Adding the following command with `sudo crontab -e` as `@reboot`. On amazon AWS EC2, run with `crontab -e` (without sudo because the username is ubuntu)

`1seed.sugarchain.info`
```bash
# dnsseed for sugarchain (mainnet)
@reboot sudo $HOME/sugarchain-seeder/dnsseed -h 1seed.sugarchain.info -n 1ns.sugarchain.info -m cryptozeny.gmail.com
```

(Optional) Reboot frequently (1)
```
# reboot every week on Monday(1) 01:01
# Minute | Hour | Day | Month | Day(Week) 
1 1 * * MON sudo /sbin/shutdown -r now
```

RUNNING AS NON-ROOT
-------------------

Typically, you'll need root privileges to listen to port 53 (name service). One solution is using an iptables rule (Linux only) to redirect it to a non-privileged port:

```bash
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
```

If properly configured, this will allow you to run dnsseed in userspace, using the -p 5353 option.

RESOURCE
--------
- https://github.com/team-exor/generic-seeder/blob/master/SETUP.md
