# Unbound DNS - Authoritative, validating, recursive caching DNS 

install it via apt

```
apt update && apt install unbound
```

get the root-hint, this is a file - that tells unbound where it finds the root DNS servers

```
wget https://www.internic.net/domain/named.root -O /var/unbound/etc/root.hints
```

and enable a crontab that automaticly updates it

```
echo "" >> /etc/crontab
echo "20 4 * * * root bash -c 'sleep $((1 + RANDOM % 3600)) && wget https://www.internic.net/domain/named.root -O /var/unbound/etc/root.hints'"
```

now get the latest version of the trust anchors

```
unbound-anchor
```

and create a crontab entry that automaticly updates it

```
echo "" >> /etc/crontab
echo "05 4 31 */2 * root bash -c 'sleep $((1 + RANDOM % 3600)) && unbound-anchor && systemctl restart unbound'
```

now we create the /etc/unbound.conf file

```
server:
    verbosity: 1

    # network config
    interface: 127.0.0.1
    port: 53
    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes

    access-control: 10.0.0.0/8 allow
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.0.0/16 allow

    # root-hints
    root-hints: "/var/unbound/etc/root.hints"

    # security settings
    hide-identity: yes
    hide-version: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: yes

    # cache settings
    cache-min-ttl: 1800
    cache-max-ttl: 86400
    prefetch: yes

    # threads
    num-threads: 4

    # the number of slabs to use for cache and must be a power of 2 times the
    # number of num-threads set above. more slabs reduce lock contention, but
    # fragment memory usage.
    msg-cache-slabs: 8
    rrset-cache-slabs: 8
    infra-cache-slabs: 8
    key-cache-slabs: 8

    # cache size
    rrset-cache-size: 256m
    msg-cache-size: 128m

    # udp buffer
    so-rcvbuf: 1m

    # dnssec related settings
    private-address: 192.168.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    unwanted-reply-threshold: 10000
    do-not-query-localhost: no

    # root.key directory
    auto-trust-anchor-file: "/var/unbound/etc/root.key"

  # and everything else goes into the public dns resolver bin
  forward-zone:
   name: "."
   forward-addr: 1.1.1.1@53#one.one.one.one
   forward-addr: 8.8.8.8@53#dns.google
   forward-addr: 9.9.9.9@53#dns.quad9.net
   forward-addr: 46.182.19.48@53#Digitalcourage e.V.
   forward-addr: 8.8.4.4@53#dns.google
   forward-addr: 149.112.112.112@53#dns.quad9.net
```

(re)start and enable unbound

```
systemctl restart unbound && systemctl enable unbound
```

and now unbound should be up&running! :)
