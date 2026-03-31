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

<img width="839" height="459" alt="image" src="https://github.com/user-attachments/assets/f5e7bc34-1f78-4404-a6b8-ae5ab4d9d724" />

<img width="915" height="533" alt="image" src="https://github.com/user-attachments/assets/8e9eaf86-ca3a-409f-939d-0adf5530927b" />



После настройки второго VLAN`а ping должен работать между хостами testClient1, testServer1 и между хостами testClient2, testServer2.

<img width="932" height="574" alt="image" src="https://github.com/user-attachments/assets/5d294a1b-a6d3-4ead-906e-d14408181d0f" />


<img width="1130" height="502" alt="image" src="https://github.com/user-attachments/assets/c88d2ec3-4311-4d26-b72e-240e62f5b537" />

# Настройка LACP между хостами inetRouter и centralRouter


Изначально необходимо на обоих хостах добавить конфигурационные файлы для интерфейсов eth1 и eth2:

<img width="1103" height="517" alt="image" src="https://github.com/user-attachments/assets/a7e7a3b0-9649-4968-81d9-3855a6c2ce31" />

После настройки интерфейсов eth1 и eth2 нужно настроить bond-интерфейс, для этого создадим файл /etc/sysconfig/network-scripts/ifcfg-bond0

<img width="888" height="330" alt="image" src="https://github.com/user-attachments/assets/2c313e56-53e7-49d9-b180-08a7659da416" />

На некоторых версиях RHEL/CentOS перезапуск сетевого интерфейса не запустит bond-интерфейс, в этом случае рекомендуется перезапустить хост.

После настройки агрегации портов, необходимо проверить работу bond-интерфейса, для этого, на хосте inetRouter (192.168.255.1) запустим ping до centralRouter (192.168.255.2):

<img width="1961" height="618" alt="image" src="https://github.com/user-attachments/assets/24ba077b-c3f7-470f-85e6-08afac19b060" />


Не отменяя ping подключаемся к хосту centralRouter и выключаем там интерфейс eth1: 


