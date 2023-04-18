# ipvs管理工具ipvsadm





| List IPVS services                             | `ipvsadm -l -n`                          |
| ---------------------------------------------- | ---------------------------------------- |
| Set up a new IPVS service                      | `ipvsadm -A -t 1.2.3.4:80 -s rr`         |
| Change scheduling mode to Round Robin          | `ipvsadm -E -t 1.2.3.4:80 -s rr`         |
| Change scheduling mode to Weighted Round Robin | `ipvsadm -E -t 1.2.3.4:80 -s wrr`        |
| Delete virtual service                         | `ipvsadm -d -t 1.2.3.4:80 -r 172.17.0.3` |
| Delete real server                             | `ipvsadm -D -t 1.2.3.4:80`               |
| Show stats                                     | `ipvsadm -L -n --stats --rate`           |
| Clear the whole table                          | `ipvsadm -C`                             |