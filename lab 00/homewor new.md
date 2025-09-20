Andrew Kotlov, [20.09.2025 18:44]
arkdown
# Лабораторная работа: Базовая настройка коммутатора Cisco

---

## Топология
![alt text](image.png)
---

## Таблица адресации

| Устройство | Интерфейс | IP-адрес / Префикс       |
|------------|-----------|--------------------------|
| S1         | VLAN 1    | 192.168.1.2 /24          |
| S1         | VLAN 99   |                          |
| PC-A       | NIC       | 192.168.1.10 /24         |

---

## Необходимые ресурсы

- Cisco 2960 (или эквивалент, PT: Switch 2960)
- PC-A (ПК)
- Прямой Ethernet-кабель (PC-A — Switch Fa0/6)
- Console-кабель (в PT — “Console”)

---

## Часть 1. Проверка конфигурации коммутатора по умолчанию

### 1. Подключение по консоли

1. В Packet Tracer соедините консольный порт PC-A с Console Switch. На PC-A: Desktop → Terminal, параметры: 9600, 8, none, 1, none.
2. На S1 "CLI" увидите приглашение Switch>.

> Вопрос 1  
> Почему нужно использовать консольное подключение для первоначальной настройки коммутатора? Почему нельзя подключиться к коммутатору через Telnet или SSH?

| Вопрос            | Ответ                                                                                                                                 |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| Почему консольное?| Telnet/SSH невозможны без настроек адреса и паролей, а консоль всегда доступна локально даже без предварительной настройки устройства. |

---

### 2. Проверка параметров по умолчанию

1. Перейдите в привилегированный режим:
    
    Switch> enable
    
2. Посмотрите текущую конфигурацию:
    
    Switch# show running-config
    
3. Сколько портов:
    
    Switch# show ip interface brief
    
4. Линии VTY:
    
    Switch# show running-config
    

| № | Вопрос                                   | Ответ                        |
|---|------------------------------------------|------------------------------|
| 2 | Сколько FastEthernet интерфейсов у 2960? | 24 (fa0/1 — fa0/24)          |
| 3 | Сколько GigabitEthernet?                 | 2 (gi0/1, gi0/2)             |
| 4 | Диапазон VTY линий?                      | 0–15                         |

---

### 3. Проверка startup-config

Switch# show startup-config

yaml

| Вопрос | Ответ                                               |
|--------|-----------------------------------------------------|
| Почему show startup-config может выдавать ошибку? | NVRAM пуста, нет стартовой конфигурации (настройки не сохранены) |

---

### 4. Изучение интерфейса SVI VLAN 1

Switch# show interface vlan 1
Switch# show ip interface vlan 1

yaml

| №  | Вопрос                                 | Ответ                                 |
|----|----------------------------------------|---------------------------------------|
| 6  | Назначен ли IP-адрес VLAN 1?           | Нет, по умолчанию не назначен         |
| 7  | Какой MAC-адрес у SVI?                 | Уникальный, вида XXXX.XXXX.XXXX       |
| 8  | Интерфейс VLAN 1 включен?              | Нет (если нет активных портов)        |
| 9  | Какие строки show ip interface vlan 1? | administratively down, без IP-адреса  |

---

### 5. Подключение PC-A к Fa0/6

- Подключите Ethernet-кабель с PC-A в Fa0/6.
- Повторите команду:
    
    Switch# show interface vlan 1
    

| Вопрос                         | Ответ                                                |
|--------------------------------|------------------------------------------------------|
| Какие выходные данные теперь?  | Статус "VLAN1 is up, line protocol is up"            |

---

### 6. Сведения о Cisco IOS

Switch# show version

yaml

Andrew Kotlov, [20.09.2025 18:44]
| Вопрос                         | Ответ                                |
|--------------------------------|--------------------------------------|
| Версия Cisco IOS?              | Ваша версия (например, 15.2(2))      |
| Имя файла образа?              | Например, c2960-lanbasek9-mz.150-2.SE.bin |

---

### 7. Свойства FastEthernet

Switch# show interface fa0/6

yaml

| №  | Вопрос                              | Ответ                                                  |
|----|-------------------------------------|--------------------------------------------------------|
| 13 | Включен ли интерфейс?               | Только если no shutdown и кабель вставлен              |
| 14 | Как включить интерфейс?             | interface fa0/6 <br/> no shutdown                     |
| 15 | MAC-адрес интерфейса?               | Уникален (пример: 001b.54ad.1a06)                      |
| 16 | Скорость/дуплекс?                   | Обычно auto; определяется согласованием                |

---

### 8. Флеш-память

Switch# show flash

yaml

| Вопрос                 | Ответ                            |
|------------------------|----------------------------------|
| Какое имя у файла IOS? | c2960-lanbasek9-mz.150-2.SE.bin  |

---

## Часть 2. Настройка базовых параметров

### 1. Настройте коммутатор

```bash
configure terminal
no ip domain-lookup
hostname S1
service password-encryption
enable secret class
banner motd #
Unauthorized access is strictly prohibited.
#
2. Назначьте SVI (VLAN 1) IP-адрес
bash
interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown
exit
3. Ограничьте консольный доступ
bash
line con 0
password cisco
login
logging synchronous
exit
4. Настройте VTY-доступ (Telnet)
bash
line vty 0 15
password cisco
login
exit
Вопрос Ответ
Для чего login? Требует ввести пароль при подключении по консоли/VTY
5. (Опционально) Задайте default gateway
bash
ip default-gateway 192.168.1.1
exit
6. Настройте IP на ПК
На PC-A: Desktop → IP Configuration
IP Address: 192.168.1.10
Subnet Mask: 255.255.255.0
Gateway: [оставьте пустым, если маршрутизация не требуется]
Часть 3. Проверка сетевых подключений
1. Просмотр конфигурации
lua
S1# show running-config
(Проверьте настройки hostname, secret, IP, консольных и VTY линий и баннера.)

Andrew Kotlov, [20.09.2025 18:44]
2. Проверьте VLAN 1
graphql
S1# show interface vlan 1
Вопрос Ответ
Пропускная способность VLAN 1? 100000 Kbit/sec
3. Пинг
На PC-A: Desktop → Command Prompt:
bash
ping 192.168.1.10    # проверка на самого себя
ping 192.168.1.2     # проверка связи с S1
4. Telnet-доступ
Desktop → Terminal → Telnet к 192.168.1.2, порт 23
Введите пароль: cisco
Команда:
bash
enable
Пароль: class
lua
show running-config
exitv
Итоговые команды конфигурации
bash
enable
configure terminal
no ip domain-lookup
hostname S1
service password-encryption
enable secret class
banner motd #
Unauthorized access is strictly prohibited.
#
interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown
exit
line con 0
password cisco
login
logging synchronous
exit
line vty 0 15
password cisco
login
exit
ip default-gateway 192.168.1.1
exit
copy running-config startup-config
Советы
Проверяйте соединения и IP-адреса.
Сохраняйте конфигурацию (copy running-config startup-config).
Не забудьте приложить скриншоты CLI, настроек ПК, успешного пинга и Telnet-подключения.
bash
# Лабораторная работа завершена! Удачи в обучении!
yaml

---

Скопируйте текст из контейнера выше и сохраните в файл с расширением `.md`. Всё красиво и структурировано!