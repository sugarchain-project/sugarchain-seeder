SUGARCHAIN-SEEDER
==============

Sugarchain-seeder is a crawler for the Sugarchain network, which exposes a list of reliable nodes via a built-in DNS server.

Features:
* regularly revisits known nodes to check their availability
* bans nodes after enough failures, or bad behaviour
* accepts nodes from Yumekawa v0.16.3.x (protocol version `70015`) to request new IP addresses.
* keeps statistics over (exponential) windows of 2 hours, 8 hours, 1 day and 1 week, to base decisions on.
* very low memory (a few tens of megabytes) and cpu requirements.
* crawlers run in parallel (by default 96 threads simultaneously).

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

USAGE
-----

Assuming you want to run a dns seed on `seed-testnet.sugarchain.org`, you will need an authorative NS record in `sugarchain.org`'s domain record, pointing to for example `ns-testnet.sugarchain.org`:

```bash
dig -t NS seed-testnet.sugarchain.org
```

```
;; ANSWER SECTION:
seed-testnet.sugarchain.org. 21599 IN	NS	ns-testnet.sugarchain.org.
```

On the system `ns.sugarchain.org`, you can now run dnsseed with root privileged to use port 53
```bash
sudo ./dnsseed --testnet -h seed-testnet.sugarchain.org -n ns-testnet.sugarchain.org -m sugarchain.dev.gmail.com
```

If you want the DNS server to report SOA records, please provide an e-mail address (with the @ part replaced by .) using -m.

Check port 53 should be opened
```bash
sudo netstat -nulp | grep 53
```

Check if it works
```bash
watch -n1 dig +short -t A seed-testnet.sugarchain.org @1.1.1.1
```

Run Yumekawa node with
```bash
./src/sugarchaind -testnet -dns=1 -dnsseed=1 -forcednsseed=1 -listen=1 -daemon
```


RUNNING AS NON-ROOT
-------------------

Typically, you'll need root privileges to listen to port 53 (name service). One solution is using an iptables rule (Linux only) to redirect it to a non-privileged port:

```bash
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
```

If properly configured, this will allow you to run dnsseed in userspace, using the -p 5353 option.
