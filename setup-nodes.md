Сетап виртуалок в Селектел

В Selectel.ru мы использовали VPC, настройки я сделал такие:

конфиг виртуалок

1 vCPU, 4 GB RAM, 10 GB disk - master1-3

1 vCPU, 4 GB RAM, 10 GB disk - ingress-1 + доп диск 10 GB для ceph

1 vCPU, 4 GB RAM, 20 GB disk + доп диск 10 GB для ceph - node1, node2

ОС: CentOS 7 x64 minimal с openssh.

Залить на них ssh ключ своего ПК/управляющего сервера/админбокса.

на виртуалках локальная сеть 172.20.100.0/24, адреса виртуалок 172.20.100.2-172.20.1007

роутер - виртуальный от Селектел, 172.20.100.1

1 плавающий ip, привязан к ingress-1 к локальному адрес 172.20.100.5
все виртуалки ходят в интернет через виртуальный роутер 172.20.100.1, через плавающий ip отдается только контент из кластера.

Делаю кластер:

hostname
ip's
root pass
master-1.k8s.vadimisakanov.ru
172.20.100.2


master-2.k8s.vadimisakanov.ru
172.20.100.3


master-3.k8s.vadimisakanov.ru
172.20.100.4


ingress-1.k8s.vadimisakanov.ru
172.20.100.5, 92.53.100.30


node-1.k8s.vadimisakanov.ru
172.20.100.6


node-2.k8s.vadimisakanov.ru
172.20.100.7




Делаем в виртуалках:
- ядро 4.х
- выключить firewalld
- выключить selinux
- выключить swap (хотя у вас его и так нет, скорее всего)
- разложить ssh-ключи “главной” управляющей машины по остальным виртуалкам; с этой машины будет производиться сетап
я делал такой скрипт:


	#!/bin/bash

sed -i "s/SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config
setenforce 0

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm 
yum --enablerepo=elrepo-kernel -y install kernel-ml
yum --enablerepo=elrepo-kernel -y swap kernel-headers -- kernel-ml-headers
yum --enablerepo=elrepo-kernel -y swap kernel-tools-libs -- kernel-ml-tools-libs
yum --enablerepo=elrepo-kernel -y install kernel-ml kernel-ml-devel kernel-ml-headers kernel-ml-tools kernel-ml-tools-libs kernel-ml-tools-libs-devel

grub2-set-default 0
grub2-mkconfig -o /boot/grub2/grub.cfg

yum update -y

reboot

exit 1

Далее
ssh-copy-id 172.20.100.[2..7]

