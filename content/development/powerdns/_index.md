+++
title = "Install and configure powerdns to serve HTTP API"
type = "chapter"
weight = 7
+++

MAAS is full of stateful services, and this is a very big problem. Ideally, MAAS is composed by multiple stateless and statefull services and they are not boundled together. 

One of the biggest and most annoying, complicated and overlooked problem is DNS. Currently MAAS uses BIND, which is a very good piece of software of course, but the way MAAS is configuring it is very messy, buggy and hard to maintain.

My wish is to move out DNS to powerdns, and have MAAS only creating the records through HTTP API. 

So, here are some notes to setup powerdns and configure it to serve HTTP API. This is just the first step of the POC journey. 

## Preerequisites

Deploy a LXD container with Ubuntu 24.04 and install the following packages:
```
sudo apt-get update 
sudo apt-get install -y pdns-server sqlite3 pdns-backend-sqlite3
```

Createe a database for powerdns

```
sudo mkdir /var/lib/powerdns
sudo sqlite3 /var/lib/powerdns/pdns.sqlite3 < /usr/share/doc/pdns-backend-sqlite3/schema.sqlite3.sql
sudo chown -R pdns:pdns /var/lib/powerdns
```

## Configure powerdns

Edit the configuration `/etc/powerdns/pdns.conf`

```

launch=gsqlite3
gsqlite3-database=/var/lib/powerdns/pdns.sqlite3

local-port=5300

api=yes
api-key=changeme

webserver=no
webserver-address=0.0.0.0
webserver-allow-from=0.0.0.0/0
webserver-port=8081
```

## Test!

Check the zones 

```
curl -X GET -H "X-API-Key: changeme" http://10.0.1.16:8081/api/v1/servers/localhost/zones
```

Create a new zone

```
curl -X POST \
  -H "X-API-Key: changeme" \
  -H "Content-Type: application/json" \
  -d '{
        "name": "mytestzone.com.",
        "kind": "Native",
        "masters": [],
        "nameservers": ["ns1.mytestzone.com."],
        "soa_edit_api": "INCEPTION-INCREMENT"
      }' \
  http://10.0.1.16:8081/api/v1/servers/localhost/zones

```

Create a new record 

```
curl -X PATCH \
  -H "X-API-Key: changeme" \
  -H "Content-Type: application/json" \
  -d '{
        "rrsets": [{
          "name": "pippo.mytestzone.com.",
          "type": "A",
          "ttl": 3600,
          "changetype": "REPLACE",
          "records": [{
            "content": "192.168.1.1",
            "disabled": false
          }]
        }]
      }' \
  http://10.0.1.16:8081/api/v1/servers/localhost/zones/mytestzone.com.
```

another one 

```
curl -X PATCH \
  -H "X-API-Key: changeme" \
  -H "Content-Type: application/json" \
  -d '{
        "rrsets": [{
          "name": "fuffa.mytestzone.com.",
          "type": "A",
          "ttl": 3600,
          "changetype": "REPLACE",
          "records": [{
            "content": "192.168.1.2",
            "disabled": false
          }]
        }]
      }' \
  http://10.0.1.16:8081/api/v1/servers/localhost/zones/mytestzone.com.
```


and then check!

```
$ dig @10.0.1.16 -p 5300 pippo.mytestzone.com.

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> @10.0.1.16 -p 5300 pippo.mytestzone.com.
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57829
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;pippo.mytestzone.com.		IN	A

;; ANSWER SECTION:
pippo.mytestzone.com.	3600	IN	A	192.168.1.1

;; Query time: 0 msec
;; SERVER: 10.0.1.16#5300(10.0.1.16) (UDP)
;; WHEN: Fri May 09 23:29:33 UTC 2025
;; MSG SIZE  rcvd: 65
```

```
$ dig @10.0.1.16 -p 5300 fuffa.mytestzone.com.

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> @10.0.1.16 -p 5300 fuffa.mytestzone.com.
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29537
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;fuffa.mytestzone.com.		IN	A

;; ANSWER SECTION:
fuffa.mytestzone.com.	3600	IN	A	192.168.1.2

;; Query time: 0 msec
;; SERVER: 10.0.1.16#5300(10.0.1.16) (UDP)
;; WHEN: Fri May 09 23:29:18 UTC 2025
;; MSG SIZE  rcvd: 65

```