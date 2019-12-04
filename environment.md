# Nebulas node environment

## Introduction

We are glad to release Nebulas Mainnet here. Please join and enjoy Nebulas Mainnet.

```
https://github.com/nebulasio/go-nebulas/tree/master
```

## Hardware configuration

The nodes of the nebulas have certain requirements for machine performance. We recommend the performance of the machine has the following requirements:

```
CPU: >= 4cores(recommand 8 cores)
RAM: >= 16G
Disk: >= 600G SSD
```

## Environment

```
System: Ubuntu 18.04(recommand), other Linux is ok.
NTP: Ensure machine time synchronization
```

#### NTP
Install the NTP service to keep system time in sync.

Ubuntu install steps:

```
#install
sudo apt install ntp
#start ntp service
sudo service ntp restart
# check ntp status
ntpq -p
```

Centos install steps:

```
#install
sudo yum install ntp
#start ntp service
sudo service ntp restart
# check ntp status
ntpq -p
```


## Contribution

Feel free to join Nebulas Mainnet. If you did find something wrong, please [submit a issue](https://github.com/nebulasio/go-nebulas/issues/new) or [submit a pull request](https://github.com/nebulasio/go-nebulas/pulls) to let us know, we will add your name and url to this page soon.
