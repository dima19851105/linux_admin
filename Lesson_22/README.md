# Домашнее задание №19

### Планируемая архитектура
построить следующую архитектуру

Сеть office1
- 192.168.2.0/26 - dev
- 192.168.2.64/26 - test servers
- 192.168.2.128/26 - managers
- 192.168.2.192/26 - office hardware

Сеть office2
- 192.168.1.0/25 - dev
- 192.168.1.128/26 - test servers
- 192.168.1.192/26 - office hardware


Сеть central
- 192.168.0.0/28 - directors
- 192.168.0.32/28 - office hardware
- 192.168.0.64/26 - wifi

Office1 ---\
-----> Central --IRouter --> internet
Office2----/

Итого должны получится следующие сервера
- inetRouter
- centralRouter
- office1Router
- office2Router
- centralServer
- office1Server
- office2Server

### Теоретическая часть
- Найти свободные подсети
- Посчитать сколько узлов в каждой подсети, включая свободные
- Указать broadcast адрес для каждой подсети
- проверить нет ли ошибок при разбиении

### Практическая часть
- Соединить офисы в сеть согласно схеме и настроить роутинг
- Все сервера и роутеры должны ходить в инет черз inetRouter
- Все сервера должны видеть друг друга
- у всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи
- при нехватке сетевых интервейсов добавить по несколько адресов на интерфейс

## Схема сети

![Схема сети](https://github.com/parshyn-dima/screens/blob/master/lesson19/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20v3.png)

## Теоретическая часть

| Подсеть            | Кол-во узлов в сети | broadcast адрес   |
|--------------------|---------------------|-------------------|
| 192.168.2.0/26     | 62                  | 192.168.2.63      |
| 192.168.2.64/26    | 62                  | 192.168.2.127     |
| 192.168.2.128/26   | 62                  | 192.168.2.191     |
| 192.168.2.192/26   | 62                  | 192.168.2.255     |
|--------------------|---------------------|-------------------|
| 192.168.1.0/25     | 126                 | 192.168.1.127     |
| 192.168.1.128/26   | 62                  | 192.168.1.191     |
| 192.168.1.192/26   | 62                  | 192.168.1.255     |
|--------------------|---------------------|-------------------|
| 192.168.0.0/28     | 14                  | 192.168.0.15      |
| 192.168.0.16/28(*) | 14                  | 192.168.0.31      |
| 192.168.0.32/28    | 14                  | 192.168.0.47      |
| 192.168.0.48/28(*) | 14                  | 192.168.0.63      |
| 192.168.0.64/26    | 62                  | 192.168.0.127     |
| 192.168.0.128/25(*)| 126                 | 192.168.0.255     |

**(*)** - Свободные подсети

## Практическая часть

Так как одним из условий задания является отключение интерфейсов NAT, чеpез которые можно подключиться к ВМ с помощью vagrant shh, настроил одну ВМ с интерфейсом через Host-only Adapter. Данная ВМ будет точкой входа для работы со стендом. По умолчанию Host-Only Adapter vboxnet0 активирован и имеет адрес 192.168.56.1. Проверить наcтройки можно VirtualBox -> File -> Host Network Manager.
Если сеть не 192.168.56.0/24, то необходимо изменить настройки данного хоста или изменить IP в Vagrantfile в следующей строке.

        box.vm.network :private_network, ip: "192.168.56.50"

![Настройки сети](https://github.com/parshyn-dima/screens/blob/master/lesson19/Network%20config.png)

На данную ВМ также устанавливается ansible.

Для проверки необходимо скачать из данного репозитория GitHub каталог Lesson_19 и перейти в него.
Далее необходимо выполнить в терминале

        vagrant up

После развертывания всех ВМ, необходимо по ssh подключаться к ВМ controller.

        ssh vagrant@192.168.56.50

Пароль стандартный *vagrant*
Далее необходимо выполнить команду

        bash /vagrant/scripts/provision.sh

Данный скрипт отключает NAT адаптер и запускает небольшой playbook, который отключает NAT интерфейсы (eth0) на всех устройствах кроме inetRouter.
После перезагрузки ВМ интерфейс eth0 восстанавливается соответственно для откючения необходимо снова запускать *provision.sh*

### Проверка работы стенда

Подключиться к ВМ **controller**

        ip route
        tracepath -n <IP>
        ping -c 5 <Внутренний IP>
        ping -c 5 <Внешний IP>

Для подключения к любой ВМ стенда можно использовать ssh.

        ssh <Внутренний IP>