# otus_vlan_lacp
Научиться настраивать VLAN и LACP.

Описание домашнего задания
в Office1 в тестовой подсети появляется сервера с доп интерфейсами и адресами
в internal сети testLAN: 
- testClient1 - 10.10.10.254
- testClient2 - 10.10.10.254
- testServer1- 10.10.10.1 
- testServer2- 10.10.10.1

Равести вланами:
testClient1 <-> testServer1
testClient2 <-> testServer2

Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов

Формат сдачи ДЗ - vagrant + ansible

<img width="836" height="724" alt="image" src="https://github.com/user-attachments/assets/28e3dffd-2b81-4a87-bb5d-d25a457d0dca" />

# Устанавливаем необходимые утилиты

Установка пакетов на CentOS 8 Stream: 
yum install -y vim traceroute tcpdump net-tools 

Установка пакетов на Ubuntu 22.04:
apt install -y vim traceroute tcpdump net-tools 


```
TASK [Показать статус пакета traceroute] ************************************************************************************************************************************************************************
ok: [inetRouter] => {
    "msg": "Пакет traceroute установлен"
}
ok: [centralRouter] => {
    "msg": "Пакет traceroute установлен"
}
ok: [office1Router] => {
    "msg": "Пакет traceroute установлен"
}
ok: [testClient1] => {
    "msg": "Пакет traceroute установлен"
}
ok: [testClient2] => {
    "msg": "Пакет traceroute установлен"
}
ok: [testServer2] => {
    "msg": "Пакет traceroute установлен"
}
ok: [testServer1] => {
    "msg": "Пакет traceroute установлен"
}

TASK [Показать статус пакета tcpdump] ***************************************************************************************************************************************************************************
ok: [inetRouter] => {
    "msg": "Пакет tcpdump установлен"
}
ok: [centralRouter] => {
    "msg": "Пакет tcpdump установлен"
}
ok: [office1Router] => {
    "msg": "Пакет tcpdump установлен"
}
ok: [testClient1] => {
    "msg": "Пакет tcpdump установлен"
}
ok: [testClient2] => {
    "msg": "Пакет tcpdump установлен"
}
ok: [testServer2] => {
    "msg": "Пакет tcpdump установлен"
}
ok: [testServer1] => {
    "msg": "Пакет tcpdump установлен"
}

TASK [Показать статус пакета net-tools] *************************************************************************************************************************************************************************
ok: [inetRouter] => {
    "msg": "Пакет net-tools установлен"
}
ok: [centralRouter] => {
    "msg": "Пакет net-tools установлен"
}
ok: [office1Router] => {
    "msg": "Пакет net-tools установлен"
}
ok: [testClient1] => {
    "msg": "Пакет net-tools установлен"
}
ok: [testClient2] => {
    "msg": "Пакет net-tools установлен"
}
ok: [testServer2] => {
    "msg": "Пакет net-tools установлен"
}
ok: [testServer1] => {
    "msg": "Пакет net-tools установлен"
}

PLAY RECAP ******************************************************************************************************************************************************************************************************
centralRouter              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
inetRouter                 : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
office1Router              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
testClient1                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
testClient2                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
testServer1                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
testServer2                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
# Настройка VLAN на хостах

На хосте testClient1 требуется создать файл /etc/sysconfig/network-scripts/ifcfg-vlan1 со следующим параметрами:

VLAN=yes
#Тип интерфейса - VLAN
TYPE=Vlan
#Указываем физическое устройство, через которые будет работать VLAN
PHYSDEV=eth1
#Указываем номер VLAN (VLAN_ID)
VLAN_ID=1
VLAN_NAME_TYPE=DEV_PLUS_VID_NO_PAD
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
#Указываем IP-адрес интерфейса
IPADDR=10.10.10.254
#Указываем префикс (маску) подсети
PREFIX=24
#Указываем имя vlan
NAME=vlan1
#Указываем имя подинтерфейса
DEVICE=eth1.1
ONBOOT=yes

<img width="772" height="397" alt="image" src="https://github.com/user-attachments/assets/99a5dd61-2084-40c8-9e38-70333f103f7e" />

На хосте testServer1 создадим идентичный файл с другим IP-адресом


<img width="746" height="417" alt="image" src="https://github.com/user-attachments/assets/5cf53a39-83fa-41b4-aceb-e943caa0a6f2" />


После создания файлов нужно перезапустить сеть на обоих хостах:
systemctl restart NetworkManager


Проверим настройку интерфейса, если настройка произведена правильно, то с хоста testClient1 будет проходить ping до хоста testServer1:


<img width="1085" height="228" alt="image" src="https://github.com/user-attachments/assets/2b099660-e70c-48e6-8a2f-c8a0fde2140f" />


# Настройка VLAN на Ubuntu:

На хосте testClient2 требуется создать файл /etc/netplan/50-cloud-init.yaml со следующим параметрами:

# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    version: 2
    ethernets:
        enp0s3:
            dhcp4: true
        #В разделе ethernets добавляем порт, на котором будем настраивать VLAN
        enp0s8: {}
    #Настройка VLAN
    vlans:
        #Имя VLANа
        vlan2:
          #Указываем номер VLAN`а
          id: 2
          #Имя физического интерфейса
          link: enp0s8
          #Отключение DHCP-клиента
          dhcp4: no
          #Указываем ip-адрес
          addresses: [10.10.10.254/24]
