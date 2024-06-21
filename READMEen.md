# padde
Passive Aggressive Dns Done Easy - PADDE

Quick and dirty passive dns

The solution currently consists of three components.

* Clickhouse installed locally https://clickhouse.com
* Suricata set up to log dns events https://suricata.io
* Taylor (in this repo), a simple Go deamon that reads the JSON log from surricata and pushes into the local clickhouse base
* Clickhouse takes care of aggregating data in the background based on the table definition

This is a POC, but works and has been tested on a probe that has peak up to 30Gb/s

TODO:
* Add command line support to Taylor to set all parameters to clickhousebase
* Create "something" that can query the base, either an API in taylor or a separate daemon
* systemd unit file to start taylor

## Installation

1. Install clickhouse locally on the surricata probe
2. Create the database

```
$ echo "CREATE DATABASE PADDE" | clickhouse-cli
$ clickhouse-cli < padde_log.sql
```
3. Compile taylor
```
CGO_ENABLED=0 go build taylor.go
```
Recommend to compile on Ubuntu 22.04 or similar. Clickhouse lib. requires newer Go.
The binary runs on RHEL 8.

4. Start taylor with the right parameters
```
$ taylor -filename /var/log/surricata/eve-dns.json -skip TXT,DNSKEY
```

5. Read data from the database
```
echo "SELECT * frorm toad.log" | clickhouse-cli
```

```
SELECT *
FROM padde.log
WHERE query LIKE 'github.com'
LIMIT 1

Query id: 4e669149-6531-4c2c-8b72-9dec7acb820e

┌─query──────┬─answer───────┬─qtype─┬──────first─┬───────last─┬─count─┐
│ github.com │ 140.82.112.3 │ A     │ 1649436366 │ 1649926564 │    18 │
└────────────┴──────────────┴───────┴────────────┴────────────┴───────┘
```

## Miscellaneous

The solution is running at UiO, tested with feed up to 30Gb/s. Taylor is multi-threaded (test on 128 cores), and reads >100Gb log data in minutes.

![image](https://user-images.githubusercontent.com/10460977/168348222-d64c0258-31bf-4843-9088-aad95bb41d7c.png)
