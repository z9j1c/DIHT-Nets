## LSA – [https://en.wikipedia.org/wiki/Link-state_advertisement](Link State Advertisement)

Stub Network` или `Stub segment` -- локальная подсеть, в которой нет других роутеров, поддерживающих `LSA`. Весь трафик в эту подсеть по `LSA` будет проходить через входной роутер.

#### Первый тип сообщений `LSA`

Роутер сообщает о своём присутствии и делится информацией о других известных ему роутерах и `Metric`-стоимостью досутпа к ним. `Link-state ID` уникален для каждого роутера.

#### Второй тип сообщений `LSA`

Выделяется один `designated router (DR)`, обладающий полной иноформацией о его `broadcast segment'е`. `Link-state ID` один для всего сегмета и равен `ID` `designated router'а`.

**Далее говорим только о первом типе сообщения LSA**

#### LSDB – Link State DB

Каждый роутер локально хранит карту сети и рассчитывает оптимальные маршруты. Поэтому не получится использовать этот протокол для интеренета.

#### Соседи (neighbours)

Как задаются `router IDs`? Можно руками, можно автоматически.  Если автоматически, то роутер смотрит на самый высокий `ip`-адрес на `loopback` инетерфейсе. Если `loopback` интерефейс не доступен, то берётся какой-нибудь другой интерефейс.

Двухсторонний `handshake`. // Перерисовать диаграмму //

#### Обмен данными

- Установка соединения
- Делимся базовой информацией из своих `LSBD`
- Аксепты, что инфа получена
- Запросы детальной информации о конкретных роутерах
- Аксепты


#### Designated rounter & Backup DR

// картинка //

R3 подключён к внешней сети.  Предположим, что R3 отключают от внешней сети,  надо рассказать об этом.  Он R8 и R2 отправляет update-сообщение. R8 и R2 должны дальше распространить инфу об обновлении.
Получается много трафика, нехорошо. Выделим designated (берут того, у кого выше ip) и backup роутеры. Designated хорошо бы иметь связь с каждым роутером в сети.


#### Маршрут и его стоимость
Для каждого типа пропускной способности установлена своя стоиомть соединения.

Выбирается путь с минимальной стоимостью. RT5 суммирует стоимости путей и рассказывает о том, что через него можно куда-то дойти с суммарной стоимостью.

#### Зоны

Зона начинается и заканчивается на одном роутере, она не может проходить между роутерами.

###  Основной минус OSPF

Каждый роутер знает обо всех роутерах в сети. Это создаёт повышенную нагрузку на сеть. Хотелось бы иметь протокол, в котором каждый роутер знает только о соседях, при этом
были хорошие метрики стоимости маршрута. Возможное решение -- `EIGRP`.

## EIGRP -- [Enhanced Interior Gateway Routing Protocol](https://en.wikipedia.org/wiki/Enhanced_Interior_Gateway_Routing_Protocol)

Придуман в `CISCO`, использовался в их внутренних сетях. Не очень популярен, в основном, работает на роутерах `CISCO`. Сложен в объяснении.

# `BGP`

До этого ввсе рассматриваемые нами протоколы были `Internal Gateway Protocol`, они заточены под сети, в которых известна топология.

`BGP` -- единственный `External Gateway Protocol`, он заточен под сети, в которых присутствуют различные автономные системы. `BGP` настраивается на `edgerouter'ах`, создавая линки между провайдерами.
Метрики `BGP` похожы на метрики `RIP`: считается кол-во `hop'ов` между `autonomuos system'ами`, `hop'ы` внутри `AS` не считаются. Работает поверх `TCP`.

`EGP` не предполагает обязально быстрого распространения изменений.


```$ show ip bgp```

`Next Hop` - сосед, через которого нужно пойти первым шагом для доступа к адресу.
`Path` -- все `AS ID`, через которые придётся пройти.


### Типы сообщений `BGP`

* `Open` -- Установка соседства. Cоседи отправляют друг другу всю свою таблицу. "The OPEN message is used to establish a BGP adjacency. Both sides negotiate session capabilities before a BGP peering establishes. The OPEN message contains the BGP version number, ASN of the originating router, Hold Time, BGP Identifier, and other optional parameters that establish the session capabilities."
* `Keepalive` -- "я ещё живой и работаю" (BGP does not rely on the TCP connection state to ensure that the neighbors are still alive. Keepalive messages are exchanged every one-third of the Hold Timer agreed upon between the two BGP routers. Cisco devices have a default Hold Time of 180 seconds, so the default Keepalive interval is 60 seconds. If the Hold Time is set for zero, no Keepalive messages are sent between the BGP neighbors.)
* `Update` -- "The Update message advertises any feasible routes, withdraws previously advertised routes, or can do both. The Update message includes the Network Layer Reachability Information (NLRI) that includes the prefix and associated BGP PAs when advertising prefixes. Withdrawn NLRIs include only the prefix. An UPDATE message can act as a Keepalive to reduce unnecessary traffic."
* `Notification` --  "Indicates an error condition to a BGP neighbor"
* `Route Refresh` -- [ошибки](https://www.inetdaemon.com/tutorials/internet/ip/routing/bgp/operation/messages/notification.shtml#:~:text=NOTIFICATION%20messages%20are%20used%20to,send%20an%20update%20or%20keepalive)

### [Состояния `BGP`](https://networklessons.com/bgp/bgp-neighbor-adjacency-states)
* `Idle` -- ожидание стартового события (прихода нового роутера с `BGP`)
* `Connect` -- ожидание трёхэтапного `TCP` рукопожатия, из этого состояния переходим в `OpenSent`
* `Active` -- попробуем другое `TCP` рукопожатие для общения с `BGP` соседом
* `OpenSent` -- "In this state BGP will be waiting for an Open message from the remote BGP neighbor. The Open message will be checked for errors, if something is wrong then BGP will respond with a Notification message and jumps back to the Idle state. This is also the moment where BGP decides whether we use EBGP or IBGP (since we check the AS number). If everything is OK then BGP starts sending keepalive messages and resets its keepalive timer. At this moment, the hold time is negotiated (lowest value is picked) between the two BGP routers. In case the TCP session fails, BGP will jump back to the Active state."
* `OpenConfirm` -- "BGP waits for a keepalive message from the remote BGP neighbor. When we receive the keepalive, we can move to the established state and the neighbor adjacency will be completed. When this occurs, it will reset the hold timer. If we receive a notification message from the remote BGP neighbor then we fall back to the Idle state. BGP will keep sending keepalive messages."
* `established` -- "The BGP neighbor adjacency is complete and the BGP routers will send update packets to exchange routing information. Every time we receive a keepalive or update message, the hold timer will be resetted. In case we receive a notification message we will jump back to the Idle state."

``
