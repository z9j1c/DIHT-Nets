## OSPF

#### Тестовая сеть

![OSPF Nets](ospf-net.png)

#### Настройка R1

```
conf t
int lo0
ip addr 5.5.5.5 255.255.255.255
no shutdown

int fa0/0
ip addr 1.1.1.1 255.255.255.0
ip ospf network point-to-point
no shutdown
exit

router ospf 200
router-id 1.1.1.1
log-adjacency-changes detail
network 1.1.1.0 0.0.0.255 area 0
network 5.5.5.5 0.0.0.0 area 0
```

```
Настройка loopback'а

Интерфейс fa0/0
ip addr
Выбираем режим ospf point-to-point (то есть не будет ни Designated, ни Backup роутера)
...

Подъём и вход в режим настройки OSPF, process-id 200 (может быть несколько OSPF процессов на одном роутере с разными id)
Установка router-id (формат тот же, что у ip addr, но не путайте их)
Включаем подробное логгирование
Добавляем подсеть и её инвертированную маску, указываем номер области, в которой находится подсеть, - 0
-- // --
```

#### Настройка R2

```
conf t
int fa0/1
ip addr 1.1.1.2 255.255.255.0
ip ospf network point-to-point
no shutdown

int fa0/0
ip addr 2.2.2.1 255.255.255.0
ip ospf network point-to-point
no shutdown
exit

router ospf 200
router-id 2.2.2.2
log-adjacency-changes detail
network 1.1.1.0 0.0.0.255 area 0
network 2.2.2.0 0.0.0.255 area 0
```

#### Настройка R3

```
conf t
int lo0
ip addr 6.6.6.6 255.255.255.255
no shutdown

int fa0/1
ip addr 2.2.2.2 255.255.255.0
ip ospf network point-to-point
no shutdown
exit

router ospf 200
router-id 3.3.3.3
log-adjacency-changes detail
network 6.6.6.6 0.0.0.0 area 0
network 2.2.2.2 0.0.0.255 area 0
```