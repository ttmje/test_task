Задача:
На любой криптовалютной бирже на выбор с помощью подручных инструментов определить, из каких данных формируется:
Order book
Лента сделок
График цены (snapshot + обновления)

Пояснение:
1) Нужно определить, что это за данные, показать их, откуда они получаются и каким образом (с помощью каких запросов, через какие протоколы + описать общую механику)
2) Нужно предоставить запросы, которые делает терминал, чтобы получить снэпшоты ордербука, сделок и графика цены
3) Нужно предоставить запросы, которые шлёт терминал, чтобы получать обновления ордербука, сделок и графика цены
(Речь про реальные данные, а не ссылки на спецификацию биржи)


На примере Binance. 
===================
### 1. Order Book 
Используется HTTP GET запрос к REST API

URL для запроса: 
https://www.binance.com/api/v3/depth?symbol=MATICUSDT&limit=1000

Где https://www.binance.com/api/v3/ протокол и адрес запроса, depth - API метод, symbol и limit параметры, передаваемые в метод.

В ответе получаем json: 

    {
    "lastUpdateId":3418949369,
    "bids":[],
    "asks":[]
    }

 
### 2. Лента Сделок
Используется HTTP GET запроос к REST API

URL для запроса: 
https://www.binance.com/api/v1/aggTrades?limit=80&symbol=MATICUSDT

Где https://www.binance.com/api/v3/ протокол и адрес запроса, aggTrades - API метод, symbol и limit параметры, передаваемые в метод.
В ответе массив json'ов

    {
    "a":187795858,
    "p":"0.65900000",
    "q":"939.50000000",
    "f":257997003,
    "l":257997004,
    "T":1654027533067,
    "m":false,
    "M":true
    }
    
Где: a - trade id, p - цена, q - количество, T - unix timestamp, m - мэйкер ли, M - лучшая цена, f,l - first/last trade id

### 3. График цены
Используется HTTP GET запрос к REST API

URL для запроса: 
https://www.binance.com/api/v3/klines?limit=1000&symbol=MATICUSDT&interval=4h

Где https://www.binance.com/api/v3/ протокол и адрес запроса, klines - API метод, symbol,limit и interval параметры, передаваемые в метод.

В ответе массив json'ов:

    [
    1639641600000,
    "2.11400000",
    "2.20200000",
    "2.10100000",
    "2.19100000",
    "23493796.70000000",
    1639655999999,
    "50559594.80660000",
    100473,
    "11668089.60000000",
    "25116483.76040000",
    "0"
    ]
Где: 1639641600000 - время открытия свечи, 2.11400000 - цена открытия, 2.20200000 - максимальная цена, 2.10100000 - минимальная цена, 2.19100000 - цена закрытия свечи, 23493796.70000000 - объем сделок, 1639655999999 - unix timestamp закрытия, 50559594.80660000 - объем валюты, 100473 - количетсво сделок, 11668089.60000000 - Taker buy base asset volume, 25116483.76040000 - Taker buy quote asset volume


### Отправляемые запросы для обновления информации: 

###### Order Book:
Request URL: wss://stream.binance.com/stream

Request Method: GET

    {"stream":"maticusdt@depth","data":{"e":"depthUpdate","E":1654106410153,"s":"MATICUSDT","U":3422047096,"u":3422047105,"b":[["0.61100000","113963.30000000"],        ["0.60900000","156236.80000000"],["0.60200000","153153.90000000"],["0.59500000","17428.40000000"]],"a":[["0.61300000","159993.30000000"],    ["0.61400000","187105.10000000"],["1.22700000","2208.30000000"]]}}	


###### Лента сделок: 
Request URL: wss://stream.binance.com/stream

Request Method: GET

    {"stream":"maticusdt@aggTrade","data":{"e":"aggTrade","E":1654105997099,"s":"MATICUSDT","a":187833790,"p":"0.61000000","q":"7442.70000000","f":258080367,"l":258080376,"T":1654105997097,"m":true,"M":true}}	

###### График цены
Request URL: wss://stream.binance.com/stream

Request Method: GET

    {"stream":"maticusdt@kline_1m","data":{"e":"kline","E":1654106301743,"s":"MATICUSDT","k":    {"t":1654106280000,"T":1654106339999,"s":"MATICUSDT","i":"1m","f":258080590,"L":258080593,"o":"0.61200000","c":"0.61300000","h":"0.61300000","l":"0.61200000","v":"1277.70000000","n":4,"x":false,"q":"782.73950000","V":"787.10000000","Q":"482.49230000","B":"0"}}}	

### Общий механизм: 

Что касается загрузки первичной информации на странице. Используя REST API сервер предоставляет доступ к своим данным клиенту по конкретному URL. В данном случае используются GET запросы для получения информации. Обобщенно: отправили GET запрос, получили ответ от сервера. В запрос передаем URL, метод и его параметры. 

Для динамического обновления информации на странице используется websocket:
Устанавливается соединение, сервер передает информацию клиенту по мере поступления. 
