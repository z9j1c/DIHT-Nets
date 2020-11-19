## NAT demo

Лучше включите виртуальную машину (а то были случаи).  В этом  демо мы настроим все 3 типа NAT.

Слева имеем локальную сеть. Справа от свича "глобальная сеть".

### Статический NAT

```
conf t

int fa0/0
ip nat inside

int fa0/1
ip nat outside

exit

ip nat inside source static 1.1.1.2 2.2.2.102
ip nat inside source static 1.1.1.3 2.2.2.103
```


#### Динамический NAT

```
ip nat pool pool-dyn 2.2.2.50 2.2.2.100 netmask 255.255.255.0
access-list 1 permit 1.1.1.1 0.0.0.255
ip nat inside source list 1 pool-dyn
```

- Указываем промежуток адресов для внешнег общения
- Нужно указывать access-list'ы, от каких внутренних машин принимать подключения
- Включаем NAT для access-list'a и пула внешних адресов