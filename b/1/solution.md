# 1. Настройка имён и IP-адресации на устройствах rtr-cod и rtr-a

## Вариант реализации:

### rtr-cod (ecorouter):

**Назначение имени на устройстве:**

- Для назначения имени устройства согласно требованиям задания используем следующие команды:
  1. переходим в режим администрирования (`enable`);
  2. переходим в режим конфигурации (`configure terminal`);
  3. задаём имя устройству (`hostname <NAME>`);
  4. задаём доменное имя (`ip domain-name <DOMAIN_NAME>`);
  5. сохраняем конфигурацию (`write memory`).

```
ecorouter>enable
ecorouter#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ecorouter(config)#hostname rtr-cod
rtr-cod(config)#ip domain-name cod.ssau205.region
rtr-cod(config)#exit
Building configuration...
rtr-cod#
```

- Проверить имя устройства можно командой `show hostname` из режима администрирования (`enable`):

```
rtr-cod#show hostname
rtr-cod
rtr-cod#
rtr-cod#
```

- Проверить доменное имя устройства можно командой `show running-config | include domain-name` из режима администрирования (`enable`):

```
rtr-cod#show running-config | include domain-name
ip domain-name cod.ssau205.region
rtr-cod#
```

**Назначение IP-адресов на устройстве:**

- Основные понятия касающиеся EcoRouter:

  1. **Порт (port)** – это устройство в составе EcoRouter, которое работает на физическом уровне (L1);

  2. **Интерфейс (interface)** – это логический интерфейс для адресации, работает на сетевом уровне (L3);

  3. **Service Instance (ServiceInstance, SI, Сервисный интерфейс)** является логическим субинтерфейсом, работающим на канальном уровне (L2) и связывает L1, L2 и L3 уровни:
    - Данный вид интерфейса необходим для соединения физического порта с интерфейсами L3, интерфейсами bridge, портами;
    - Используется для гибкого управления трафиком на основании наличия меток VLANов в фреймах, или их отсутствия;
    - Сквозь сервисный интерфейс проходит весь трафик, приходящий на порт.

- Таким образом, для того чтобы назначить IPv4-адрес на EcoRouter, необходимо придерживаться следующего алгоритма в общем виде:
  1. Создать интерфейс с произвольным именем и назначить на него IPv4-адрес;
  2. В режиме конфигурирования порта создать сервис-инстанс с произвольным именем:
    - указать (инкапсулировать) что будет обрабатываться нетегированный или не тегированный трафик;
    - указать в какой интерфейс (ранее созданный) нужно отправлять обработанные кадры.

- Посмотреть физические порты можно командой `show port brief` из режима администрирования (`enable`):
  - порт **te0** направлен в сторону BM meg;
  - порт **te1** направлен в сторону rtr-a:

```
rtr-cod#show port brief
Name     Physical    Admin    LACP    Description
te0      --          up       --      --
te1      --          up       --      --
rtr-cod#
```

- Создадим интерфейс с именем **isp** и назначим на него IP-адрес **178.207.179.4/29**, также зададим для данного интерфейса описание (description - опциональный, необязательный параметр):

```
rtr-cod(config)#interface isp
rtr-cod(config-if)#ip address 178.207.179.4/29
rtr-cod(config-if)#description "Connecting to an ISP provider"
rtr-cod(config-if)#exit
rtr-cod(config)#
```

- Создадим интерфейс с именем **fw-cod** и назначим на него IP-адрес **172.16.1.1/30**, также зададим для данного интерфейса описание (description - опциональный, необязательный параметр):

```
rtr-cod(config)#interface fw-cod
rtr-cod(config-if)#ip address 172.16.1.1/30
rtr-cod(config-if)#description "Connecting to fw-cod"
rtr-cod(config-if)#exit
rtr-cod(config)#
```

- Проверить назначенные IP-адреса на интерфейсы можно командой `show ip interface brief` из режима администрирования (`enable`):
  - созданные интерфейсы пока не добавлены в какие-либо Service Instance, а значит не привязаны к к порту, отсюда и статус **down**

```
rtr-cod#show ip interface brief
Interface        IP-Address           Status      VRF
isp              178.207.179.4/29     down        default
fw-cod           172.16.1.1/30        down        default
rtr-cod#
```

- В режиме конфигурирования порта **te0** необходимо создать **service-instance** с произвольным именем, например **te0/isp**:
  - также необходимо указать (инкапсулировать) что будет обрабатываться не тегированный трафик (**untagged**);
  - указать в какой интерфейс (ранее созданный с именем **isp**) нужно отправлять обработанные кадры.

```
rtr-cod(config)#port te0
rtr-cod(config-port)#service-instance te0/isp
rtr-cod(config-service-instance)#encapsulation untagged
rtr-cod(config-service-instance)#connect ip interface isp
rtr-cod(config-service-instance)#exit
rtr-cod(config-port)#exit
rtr-cod(config)#
```

- В режиме конфигурирования порта **te1** необходимо создать **service-instance** с произвольным именем, например **te1/fw-cod**:
  - также необходимо указать (инкапсулировать) что будет обрабатываться не тегированный трафик (**untagged**);
  - указать в какой интерфейс (ранее созданный с именем **fw-cod**) нужно отправлять обработанные кадры.

```
rtr-cod(config)#port te1
rtr-cod(config-port)#service-instance te1/fw-cod
rtr-cod(config-service-instance)#encapsulation untagged
rtr-cod(config-service-instance)#connect ip interface fw-cod
rtr-cod(config-service-instance)#exit
rtr-cod(config-port)#exit
rtr-cod(config)#write memory
Building configuration...
rtr-cod(config)#
```

- Проверить назначенные IP-адреса на интерфейсы можно командой `show ip interface brief` из режима администрирования (`enable`):
  - созданные интерфейсы пока не добавлены в какие-либо Service Instance, а значит не привязаны к к порту, отсюда и статус **down**

```
rtr-cod#show ip interface brief
Interface        IP-Address           Status      VRF
isp              178.207.179.4/29     up          default
fw-cod           172.16.1.1/30        up          default
rtr-cod#
```

- Проверить созданные Service Instance можно командой `show service-instance brief` из режима администрирования (`enable`):

```
rtr-cod#show service-instance brief
Total instances: 2
A - Active;
N - Not connected;
D - Down;
PORT    SERVICE INSTANCE    ENDPOINT      ENDPOINT      BRIEF
NAME    NAME                TYPE          NAME          DESCRIPTION
---------------------------------------------------------------------
te0     te0/isp             iface         isp           -
te1     te1/fw-cod          iface         fw-cod        -
rtr-cod#
```

- IP адрес шлюза по умолчанию на данном устройстве на текущий момент не задётся (будет рассмотрено далее, в соответствующем разделе), т.к. по условиям задания:
  - Маршрутизатор ЦОД должен получать маршруту по умолчанию по BGP
  - Рукой создадим маршрута по умолчанию ЗАПРЕЩЕНО!
- Но связность с Интернет провайдером ВР проверить стоит:

```
rtr-cod#ping 178.207.179.1
Ping 178.207.179.1 (178.207.179.1) 56(84) bytes of data.
64 bytes from 178.207.179.1: icmp_seq=1 ttl=64 time=30.9 ms
64 bytes from 178.207.179.1: icmp_seq=2 ttl=64 time=30.8 ms
64 bytes from 178.207.179.1: icmp_seq=3 ttl=64 time=23.8 ms

--- 178.207.179.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 23.768/85.055/193.319/76.414 ms
rtr-cod#
```

### rtr-a (ecorouter):

**Назначение имени на устройстве:**

- Реализация аналогично rtr-cod, за исключением соответствующего доменного имени:
  - имя устройства должно быть:

```
rtr-a>show hostname
rtr-a
rtr-a#
rtr-a#
```

  - доменное имя на устройстве должно быть:

```
rtr-a#show running-config | include domain-name
ip domain-name office.ssau205.region
rtr-a#
rtr-a#
```

**Назначение IP-адресов на устройстве:**

- Реализация аналогично rtr-cod, за исключением того, что на базе физического порта **te1** должны быть созданы интерфейсы и Service Instance с целью обработки тегированного трафика для предоставления возможности маршрутизации между VLAN (рассмотрено далее)
  - должен быть создан интерфейс для подключения к Интернет провайдеру **isp**

```
rtr-a#show ip interface brief
Interface        IP-Address           Status      VRF
isp              178.207.179.28/29    down        default
rtr-a#
```

- должен быть создан Service Instance на порту **te0**
  - на созданный Service Instance реализована обработка не тегированного трафика
  - в созданный Service Instance добавлен интерфейс для подключения к Интернет провайдеру

```
rtr-a#show ip interface brief
Interface        IP-Address           Status      VRF
isp              178.207.179.28/29    up          default
rtr-a#show service-instance brief
Total instances: 1
A - Active;
N - Not connected;
D - Down;
PORT    SERVICE INSTANCE    ENDPOINT      ENDPOINT      BRIEF
NAME    NAME                TYPE          NAME          DESCRIPTION
---------------------------------------------------------------------
te0     te0/isp             iface         isp           -
rtr-a#
```

- В отличие от rtr-cod, на rtr-a по заданию нет никаких требований про настройку BGP, а значит для доступа в сеть Интернет, маршрут по умолчанию можно задать вручную:
  - переходим в режим конфигурации (`configure terminal`)
  - с помощью команды `ip route <IP_NETWORK/PREFIX> <NEXT_HOP_IP_ADDRESS>` задаём маршрут по умолчанию (шлюз)

```
rtr-a(config)#ip route 0.0.0.0/0 178.207.179.25
rtr-a(config)#
```

- Проверить назначенный маршрут по умолчанию можно командой `show ip route static` из режима администрирования (`enable`):

```
rtr-a#show ip route static
IP Route Table for VRF "default"
Gateway of last resort is 178.207.179.25 to network 0.0.0.0

S*   0.0.0.0/0 [1/0] via 178.207.179.25, isp
rtr-a#
```

- Так же стоит проверить доступ в сеть Интернет:

```
rtr-a#ping 77.88.8.8
PING 77.88.8.8 (77.88.8.8) 56(84) bytes of data.
64 bytes from 77.88.8.8: icmp_seq=1 ttl=51 time=164 ms
64 bytes from 77.88.8.8: icmp_seq=2 ttl=51 time=90.5 ms
64 bytes from 77.88.8.8: icmp_seq=3 ttl=51 time=33.6 ms
64 bytes from 77.88.8.8: icmp_seq=4 ttl=51 time=37.6 ms

--- 77.88.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 33.605/68.814/163.518/31.719 ms
rtr-a#
```

- Реализуем создание под-интерфейсов для дальнейшего маршрутизации между VLAN-ами:
  - Создаём интерфейсы с произвольным именем для каждого VLAN-а и назначаем на них IP-адреса:

```
rtr-a(config)#interface v1100
rtr-a(config-if)#ip address 172.20.10.254/24
rtr-a(config-if)#description "VLAN - SRV"
rtr-a(config-if)#exit
rtr-a(config)#interface v1200
rtr-a(config-if)#ip address 172.20.20.254/24
rtr-a(config-if)#description "VLAN - CLI"
rtr-a(config-if)#exit
rtr-a(config)#interface v1300
rtr-a(config-if)#ip address 172.20.30.254/24
rtr-a(config-if)#description "VLAN - MGMT"
rtr-a(config-if)#exit
rtr-a(config)#
```

- Проверить назначенные IP-адреса на интерфейсы можно командой `show ip interface brief` из режима администрирования (`enable`):
  - созданные интерфейсы пока не добавлены в какие-либо Service Instance, а значит не привязаны к к порту, отсюда и статус **down**

```
rtr-a#show ip interface brief
Interface        IP-Address           Status      VRF
isp              178.207.179.28/29    up          default
v1100            172.20.10.254/24     down        default
v1200            172.20.20.254/24     down        default
v1300            172.20.30.254/24     down        default
rtr-a#
```

- на базе физического интерфейса **te1** для каждого VLAN-а создадим **service-instance** с инкапсуляцией соответствующих тегов (VID) и подключением необходимых интерфейсов:

### Пояснение к листингу команд указанному ниже

Операции над метками в сервисных интерфейсах:

Есть три варианта операций над метками: **удаление существующей метки/меток**, **добавление новой метки (меток)** и **трансляция метки/меток из одного значения в другое**.

Пояснение команд:

- Указание номера обрабатываемого VLAN выполняется на service-instance с помощью команды `encapsulation dot1q <VID> exact`:
  - опция **exact** показывает, что над это правилом попадут кадры только с меткой данной <VID>
  - слово **exact** писать не обязательно, так как это поведение по умолчанию и в выводе `show run` это слово не отображается
- Указание выполняемой операции выполняется на service-instance с помощью команды `rewrite pop <N>`:
  - опция **1** показывает, что снимаем только одну, верхнюю метку, на L3 кадр должен поступать без признаков VLAN

```
rtr-a(config)#port te1
rtr-a(config-port)#service-instance te1/v1100
rtr-a(config-service-instance)#encapsulation dot1q 100
rtr-a(config-service-instance)#rewrite pop 1
rtr-a(config-service-instance)#connect ip interface v1100
rtr-a(config-service-instance)#exit
rtr-a(config-port)#service-instance te1/v1200
rtr-a(config-service-instance)#encapsulation dot1q 200
rtr-a(config-service-instance)#rewrite pop 1
rtr-a(config-service-instance)#connect ip interface v1200
rtr-a(config-service-instance)#exit
rtr-a(config-port)#service-instance te1/v1300
rtr-a(config-service-instance)#encapsulation dot1q 300
rtr-a(config-service-instance)#rewrite pop 1
rtr-a(config-service-instance)#connect ip interface v1300
rtr-a(config-service-instance)#exit
rtr-a(config-port)#exit
rtr-a(config)#write memory
Building configuration...
rtr-a(config)#
```

- Проверить назначенные IP-адреса на интерфейсы можно командой `show ip interface brief` из режима администрирования (`enable`):

```
rtr-a#show ip interface brief
Interface        IP-Address           Status      VRF
isp              178.207.179.28/29    up          default
v1100            172.20.10.254/24     up          default
v1200            172.20.20.254/24     up          default
v1300            172.20.30.254/24     up          default
rtr-a#
```

- Проверить созданные Service Instance можно командой `show service-instance brief` из режима администрирования (`enable`):

```
rtr-a#show service-instance brief
Total instances: 4
A - Active;
N - Not connected;
D - Down;
PORT    SERVICE INSTANCE    ENDPOINT      ENDPOINT      BRIEF
NAME    NAME                TYPE          NAME          DESCRIPTION
---------------------------------------------------------------------
te0     te0/isp             iface         isp           -
te1     te1/v1100           iface         v1100         -
te1     te1/v1200           iface         v1200         -
te1     te1/v1300           iface         v1300         -
rtr-a#
```

---

Последнее изменение: понедельник, 10 ноября 2025, 17:15

---

**Предыдущий элемент курса**
Настройка виртуальной машины ISP

**Следующий элемент курса**
2. Подход к настройке fw-cod

---

### Обратная связь
Подпишитесь

Вы используете гостевой доступ (Вход)
Сводка хранения данных

Тема оформления сайта разработана
conecti.me
