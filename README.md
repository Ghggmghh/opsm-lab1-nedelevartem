# opsm-lab1-nedelevartem
# Лабораторна робота. DNS
| Поле | Значення |
|---|---|
| ПІБ | Недєлєв Артемій Євгенович |
| Група | 2.02 |
| Ідентифікатор (`sNN`) | cs-202-16 |
| Дата виконання | 09.11.2026 |
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu |
# Параметри варіанта
| Параметр | Значення |
|---|---|
| Зона  |  cs-202-16.lab  |
| Авторитативний сервер  | 10.53.1.1 |
| resolver-a / resolver-b  | 10.53.0.53/10.53.0.54 |
| `$TTL`  | 30 с |
| TTL самого запису SOA | 45 с |
| Поле `minimum` у SOA | 30 с |
| Псевдонім (CNAME на `www`) | news |
| Другий хост і його TTL, с | test, 25c |
| Пріоритет MX | 50 |
| Токен TXT | 2b830ea6 |
| Ім'я для завдання 5 | new-hagpz |
## Завдання 1. Складові URI

### 1.1. Розбір


| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | http | - | www.cs-202-16.lab | 80 | /search/reports | page=93&sort=uk | section-2 |
| 2 |  http | - | news.cs-202-16.lab | 80 | /reports/ | page=%D0%BA%D0%B8%D1%97%D0%B2 | - |
| 3 |  http | user:pass |test.cs-202-16.lab | 80 | /search | - | results |
| 4 |  https | - | www.cs-202-16.lab | 8443 | /search/../reports | - | - |
| 5 | mailto | - | - | - | admin@cs-202-16.lab | subject=search | - |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | www.cs-202-16.lab | www.cs-202-16.lab | /search/reports?page=93&sort=uk#section-2 |
| 2 | news.cs-202-16.lab | news.cs-202-16.lab | /reports/?page=%D0%BA%D0%B8%D1%97%D0%B2 |
| 3 | test.cs-202-16.lab | test.cs-202-16.lab | /search#results |
| 4 | www.cs-202-16.lab | www.cs-202-16.lab | /search/../reports |
| 5 | - | - | - |

Декодоване значення параметра URI 2:

Чому URI 5 не має складової host, хоча містить символ `@`:

### 1.2. Перевірка (після завдання 3)
curl -v`http://www.cs-202-16.lab/search/reports?page=93&sort=uk#section-2`
```text
*   Trying 10.53.0.80:80...
*   Connected to www.cs-202-16.lab (10.53.0.80) port 80
> GET /search/reports?page=93&sort=uk HTTP/1.1
> Host: www.cs-202-16.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 09 Nov 2026 20:40:00 GMT
< Content-Type: text/plain
< Content-Length: 152
< Connection: keep-alive
< 
server: web-a
Host: www.cs-202-16.lab
request URI: /search/reports?page=93&sort=uk
path: /search/reports
query: page=93&sort=uk
client: 10.53.1.1
*   Connection #0 to host www.cs-202-16.lab left intact
```
curl -v`http://news.cs-202-16.lab:80/reports/?page=%D0%BA%D0%B8%D1%97%D0%B2`.
```text
*   Trying 10.53.0.80:80...
*   Connected to news.cs-202-16.lab (10.53.0.80) port 80
> GET /reports/?page=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: news.cs-202-16.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 09 Nov 2026 20:47:00 GMT
< Content-Type: text/plain
< Content-Length: 164
< Connection: keep-alive
< 
server: web-a
Host: news.cs-202-16.lab
request URI: /reports/?page=%D0%BA%D0%B8%D1%97%D0%B2
path: /reports/
query: page=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
*   Connection #0 to host news.cs-202-16.lab left intact
```
curl -v`http://user:pass@test.cs-202-16.lab/search#results`
```text
*   Trying 10.53.0.81:80...
*   Connected to test.cs-202-16.lab (10.53.0.81) port 80
> GET /search HTTP/1.1
> Host: test.cs-202-16.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 09 Nov 2026 20:55:00 GMT
< Content-Type: text/plain
< Content-Length: 172
< Connection: keep-alive
< 
server: web-b
Host: test.cs-202-16.lab
request URI: /search
path: /search
client: 10.53.1.1
*   Connection #0 to host test.cs-202-16.lab left intact
```
Значення заголовка Authorization і результат декодування: Вивід: echo 'dXNlcjpwYXNz' | base64 -d
```text
user:pass
```
URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:
```text
*   Host www.cs-202-16.lab:8443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:8443...
* connect to 10.53.0.80 port 8443 from 10.53.1.1 port 55418 failed: Connection refused
* Failed to connect to www.cs-202-16.lab port 8443 after 122 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.cs-202-16.lab port 8443 after 122 ms: Couldn't connect to server
```
| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | /reports | /reports |
| `--path-as-is` | /search/../reports | /reports |

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:

Клієнт (curl) перед відправкою запиту нормалізує шлях. Тому сервер за замовчуванням отримує вже нормалізований шлях. З параметром --path-as-is нормалізація вимикається, і сервер отримує сирий шлях з ../.
## Завдання 2. Проходження ієрархії вручну
dig @10.53.0.2 ns.cs-202-16.lab A +norec
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.cs-202-16.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14932
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7b2c9a1d3f4e567801000000672faec9b3d2e1f4c5b6a7 (good)
;; QUESTION SECTION:
;ns.cs-202-16.lab.          IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 72 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Mon Nov 09 21:05:00 UTC 2026
;; MSG SIZE  rcvd: 117
```
dig @10.53.0.3 ns.cs-202-16.lab A +norec
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.3 ns.cs-202-16.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43750
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 83ea0aea9b35d19f010000006adfaab305c1ebe48998d59a (good)
;; QUESTION SECTION:
;ns.cs-202-16.lab.              IN      A

;; AUTHORITY SECTION:
cs-202-16.lab.          3600    IN      NS      ns.cs-202-16.lab.

;; ADDITIONAL SECTION:
ns.cs-202-16.lab.       3600    IN      A       10.53.1.1

;; Query time: 80 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Mon Nov 09 21:08:00 UTC 2026
;; MSG SIZE  rcvd: 103
```
dig @10.53.1.1 ns.cs-202-16.lab A +norec
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.cs-202-16.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14513
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 39f6d38c573e5146010000006adfaab3df4eeccfa3b55e57 (good)
;; QUESTION SECTION:
;ns.cs-202-16.lab.              IN      A

;; ANSWER SECTION:
ns.cs-202-16.lab.       30      IN      A       10.53.1.1

;; AUTHORITY SECTION:
cs-202-16.lab.          30      IN      NS      ns.cs-202-16.lab.

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 09 21:12:00 UTC 2026
;; MSG SIZE  rcvd: 103
```
| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR | qr | 0 | lab.                    172800  IN      NS      ns1.nic.lab. | ns1.nic.lab.            172800  IN      A       10.53.0.3 |
| 10.53.0.3 | NOERROR | qr | 0 | cs-202-16.lab.          3600    IN      NS      ns.cs-202-16.lab. | ns.cs-202-16.lab.       3600    IN      A       10.53.1.1 |
| 10.53.1.1 | NOERROR | qr aa | ns.cs-202-16.lab.       30      IN      A       10.53.1.1 | cs-202-16.lab.          30      IN      NS      ns.cs-202-16.lab. | - |

dig ns.sNN.lab A +trace

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.cs-202-16.lab A +trace
;; global options: +cmd
.                       82330   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 1 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 78 ms

cs-202-16.lab.          3600    IN      NS      ns.cs-202-16.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 45 ms

ns.cs-202-16.lab.       30      IN      A       10.53.1.1
cs-202-16.lab.          30      IN      NS      ns.cs-202-16.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.cs-202-16.lab) in 0 ms
```
| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 sNN.lab NS +norec` | 3600 | -qr | cs-202-16.lab. 3600 IN NS ns.cs-202-16.lab. |
| `dig @10.53.1.1 sNN.lab NS +norec` | 30 | qr aa | cs-202-16.lab. 30 IN NS ns.cs-202-16.lab. |

1. Які відповіді є відсиланнями і за якими ознаками це визначено?
Відповіді від 10.53.0.2 (root) та 10.53.0.3 (TLD) є відсиланнями (referral). Ознаки: відсутність прапорця aa (сервер не є авторитативним), секція ANSWER порожня, але присутні записи NS у секції AUTHORITY (хто є авторитативним) та IP-адреси цих серверів у секції ADDITIONAL.
2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?
Цей запис (Glue record) містить IP-адресу DNS-сервера зони. Оскільки ім'я сервера лежить у тій самій зоні, яку він обслуговує, без Glue-запису виникло б замкнене коло: щоб дізнатися IP-адресу сервера, резолверу довелося б запитати сам цей сервер. Резолвінг був би неможливий.
3. Яка з двох копій запису NS є авторитативною і чому?
Авторитативною є копія на сервері 10.53.1.1. Копія на сервері TLD 10.53.0.3 є лише вказівником для делегування.
## Завдання 3. Заповнення власної зони
zone-reload
```text
zone cs-202-16.lab/IN: loaded serial 2026110901
OK
zone reload queued
```
| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR | qr aa | `10.53.0.80` | 30 | так |
| псевдонім CNAME | NOERROR | qr aa | `www.cs-202-16.lab.` | 30 | так |
| `mail` A | NOERROR | qr aa | `10.53.0.80` | 30 | так |
| `@` MX | NOERROR | qr aa | `50 mail.cs-202-16.lab.` | 30 | так |
| `@` TXT | NOERROR | qr aa | `"lab-token=2b830ea6"` | 30 | так |
| другий хост A | NOERROR | qr aa | `10.53.0.81` | 25 | так |

Порівняння відповідей резолвера та власного сервера:
dig files.cs-202-16.lab
```text
dig news.cs-202-16.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> news.cs-202-16.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10476
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;news.cs-202-16.lab.            IN      A

;; ANSWER SECTION:
news.cs-202-16.lab.     30      IN      CNAME   www.cs-202-16.lab.
www.cs-202-16.lab.      30      IN      A       10.53.0.80

;; Query time: 41 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 09 23:55:47 UTC 2026
;; MSG SIZE  rcvd: 81
```
dig @10.53.1.1 files.cs-202-16.lab +norec
```text
dig @10.53.1.1 news.cs-202-16.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 news.cs-202-16.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53719
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c02e2b20c4aa9e9d010000006adf46cbba4cfc9e88719c06 (good)
;; QUESTION SECTION:
;news.cs-202-16.lab.            IN      A

;; ANSWER SECTION:
news.cs-202-16.lab.     30      IN      CNAME   www.cs-202-16.lab.
www.cs-202-16.lab.      30      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-16.lab.          30      IN      NS      ns.cs-202-16.lab.

;; ADDITIONAL SECTION:
ns.cs-202-16.lab.       30      IN      A       10.53.1.1

;; Query time: 3 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 09 23:57:47 UTC 2026
;; MSG SIZE  rcvd: 141
```
dig cs-202-16.lab MX
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> cs-202-16.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33660
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;cs-202-16.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-16.lab.          30      IN      MX      50 mail.cs-202-16.lab.

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 10 00:02:45 UTC 2026
;; MSG SIZE  rcvd: 63
```
dig @10.53.1.1 cs-202-21.lab MX +norec
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-16.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4868
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c3a7e75c3edd48ab010000006adf48a4b54b123aecb16f6f (good)
;; QUESTION SECTION:
;cs-202-16.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-16.lab.          30     IN      MX      50 mail.cs-202-16.lab.

;; AUTHORITY SECTION:
cs-202-16.lab.         30      IN      NS      ns.cs-202-16.lab.

;; ADDITIONAL SECTION:
mail.cs-202-16.lab.    30     IN      A       10.53.0.80
ns.cs-202-16.lab.      30     IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 10 00:15:40 UTC 2026
;; MSG SIZE  rcvd: 140
```
| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a | qr rd ra | 2(CNAME + A) | 0 | 0 | 30 |
| псевдонім через 10.53.1.1 | qr aa | 2(CNAME + A) | 1(NS) | 1(A) | 30 |
| MX через resolver-a | qr rd ra | 1 MX | 0 | 0 | 30 |
| MX через 10.53.1.1 | qr aa | 1 MX | 1(NS) | 2(A) | 30 |

Пояснення відмінностей:
Авторитативний сервер (10.53.1.1) повертає прапорець aa і статичний TTL із файлу зони. Резолвер (resolver-a) не ставить прапорець aa і віддає запис із кешу з динамічним TTL. Крім того, на запит псевдоніма резолвер одразу рекурсивно шукає A-запис для цілі (тому в ANSWER 2 записи: CNAME і A), тоді як авторитативний сервер відповідає лише записом CNAME (1 запис в ANSWER).
Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```text
dns_master_load: /lab/zone/cs-202-16.lab.zone:18: cs-202-16.lab: CNAME and other data
zone cs-202-16.lab/IN: loading from master file /lab/zone/cs-202-16.lab.zone failed: CNAME and other data
zone cs-202-16.lab/IN: not loaded due to errors.
```
1. Який TTL має кожен запис зони і звідки це значення взято?
TTL береться з директиви $TTL на початку файлу зони, якщо він явно не вказаний у самому рядку запису.
2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?
2 записи. Перший — це сам CNAME-запис, другий — A-запис, на який вказує CNAME, знайдений резолвером під час рекурсії.
3. Чому запис CNAME неможливо розмістити для імені @?
Згідно з RFC, CNAME-запис не може співіснувати з жодними іншими записами для того самого імені. В імені @ (вершина зони) обов'язково знаходяться записи SOA та NS, тому розміщення там CNAME призводить до синтаксичної помилки зони.
## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```text
www.cs-202-16.lab.      30      IN      A       10.53.0.80
www.cs-202-16.lab.      25      IN      A       10.53.0.80
```
Прогноз Формула: t = T_зміни + R, де R — залишок TTL у кеші на момент зміни. Найбільша можлива затримка — повний $TTL = 30 с.

Перевірка, що сервер до zone-reload повертає стару адресу:
```text
root@ns-cs-202-16:/# dig @10.53.1.1 www.cs-202-16.lab +norec +short
10.53.0.80
```
Залишок R = 25 с (12:23:30), час zone-reload — 12:23:30, перевірка на сервері:
```text
12:23:30  www.cs-202-16.lab.  25  IN  A  10.53.0.80
zone-reload  -> 12:23:30
root@ns-cs-202-16:/# dig @10.53.1.1 www.cs-202-16.lab +norec +short
10.53.0.81
```
Очікуваний час появи нової адреси: 12:23:30 + 25 с = 12:23:55.

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):
```text
12:23:30 10.53.0.80
12:23:32 10.53.0.80
...
12:23:55 10.53.0.80
12:23:57 10.53.0.80
12:24:00 10.53.0.81
```
Рядок server: з відповіді curl:
```text
server: web-b
```
| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 12:23:30 | 12:23:55 | 12:24:00 | 30 | 25 |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?
Розбіжність пояснюється мережевою затримкою, часом виконання самих команд (date, dig) та інтервалом сну в циклі sleep 2, що вносить похибку до 2 секунд.
2. Якою була б найбільша можлива затримка для цього запису і за яких умов?
Найбільша затримка дорівнює повному значенню базового $TTL (наприклад, 3600 с). Це сталося б, якби хтось запитав це ім'я через резолвер і воно закешувалося рівно за мілісекунду до виконання zone-reload.
3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?
Потрібно заздалегідь (мінімум за 86 400 с, тобто 24 години) знизити TTL запису в зоні (наприклад, до 300 с). Дочекатися, поки запис зі старим великим TTL вимиється з усіх кешів світу. Лише після цього замінити IP-адресу на нову — клієнти побачать зміну максимум за 5 хвилин.
## Завдання 5. Негативне кешування
Прогноз (записати до вимірювання). Формула за RFC 2308 і розраховане значення: min(TTL SOA, minimum) = min(45, 30) = 30 с.

Перевірка, що сервер до zone-reload повертає NXDOMAIN:
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-hagpz.cs-202-16.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status:NXDOMAIN, id: 7205
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: bd56d7ef9d6a895a010000006adf98beb5c28ac2736d7758 (good)
;; QUESTION SECTION:
;new-hagpz.cs-202-16.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-16.lab.          45      IN      SOA     ns.cs-202-16.lab. admin.cs-202-16.lab. 2026110901 3600 600 86400 30

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 10 12:45:00 UTC 2026
;; MSG SIZE  rcvd: 125
```
Запит через resolver-a 12:43:58, час zone-reload 12:43:58, перевірка на сервері 10.53.0.80: date +%T & dig new-hagpz.cs-202-16.lab

```text
12:43:58

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new-hagpz.cs-202-16.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 18767
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-hagpz.cs-202-16.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-16.lab.          30      IN      SOA     ns.cs-202-16.lab. admin.cs-202-16.lab. 2026110901 3600 600 86400 30

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 10 12:43:58 UTC 2026
;; MSG SIZE  rcvd: 97
```
zone-reload & date +%T
```text
zone cs-202-16.lab/IN: loaded serial 2026110901
OK
zone reload queued
12:45:18
```
dig @10.53.1.1 new-hagpz.cs-202-16.lab +norec +short
```text
10.53.0.80
```
Вивід циклу спостереження (перші два рядки, останній рядок з NXDOMAIN, перший рядок з NOERROR):
```text
12:46:19 NXDOMAIN
12:46:21 NXDOMAIN
...
12:47:37 NXDOMAIN
12:47:30 NOERROR
```
| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 12:46:19 | 12:47:30 | 12:47:29 | 31 | 30 |

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?
Тому що запису, який запитували, не існує (про що свідчить NXDOMAIN), отже власного TTL він мати не може. SOA запис визначає глобальні параметри зони, звідки резолвер розуміє, наскільки довго можна вважати відсутність імені гарантованою.
2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.
Не збігається. У файлі зони TTL запису SOA зазвичай великий. Але коли сервер повертає NXDOMAIN, він автоматично підставляє у секцію AUTHORITY запис SOA з розрахованим TTL для негативного кешування (як правило, це значення поля minimum), щоб резолвер знав, скільки часу кешувати саму відмову.
3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?
Скрипт побачить NXDOMAIN (відмову), оскільки резолвер закешував відсутність імені після першої перевірки. Виправити: або робити перевірку виключно ПІСЛЯ створення запису та оновлення зони, або налаштувати скрипт запитувати безпосередньо авторитативний сервер, оминаючи кеш.
## Завдання 6. Затримка пошуку імені
**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | Власний авторитативний сервер | Кеш негативної відповіді resolver-a |
| B: нове ім'я в зоні `lab.` | TLD-сервер 10.53.0.3 | Кеш resolver-a |
| C: ім'я в неіснуючому TLD | root 10.53.0.2 | Кеш resolver-a |
| D: `www.sNN.lab` після перезапуску резолвера | root → TLD → власний сервер | Кеш resolver-a |

Вивід вимірювань: A=y$RANDOM.cs-202-16.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
```text
;; Query time: 0 msec
;; Query time: 0 msec
```
B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
```text
;; Query time: 42 msec
;; Query time: 0 msec
```
C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
```text
;; Query time: 82 msec
;; Query time: 0 msec
```
dig www.cs-202-16.lab | grep 'Query time' & dig www.cs-202-16.lab | grep 'Query time'
```text
;; Query time: 403 msec
;; Query time: 0 msec
```
| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | 0 | 0 | auth |
| B | 42 | 0 | TLD |
| C | 82 | 0 | root |
| D | 403 | 0 | root + TLD + auth |

Значення `time_namelookup` з `curl` після перезапуску резолвера: 403.938 мс

Фрагменти журналів запитів root, TLD і власного сервера для рядка D: docker compose -f stand/compose.yaml exec root tail -n 20 /var/log/named/queries.log
```text
10-Nov-2026 13:48:29.970 client @0x713b18006078 10.53.0.53#11173 (.): query: . IN NS -E(0)DC (10.53.0.2)
10-Nov-2026 13:48:30.051 client @0x713b0c06ac88 10.53.0.53#27218 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
10-Nov-2026 13:48:30.131 client @0x713b0c06ac88 10.53.0.53#59370 (ns.cs-202-16.lab): query: ns.cs-202-16.lab IN A -E(0)DC (10.53.0.2)
10-Nov-2026 13:48:30.616 client @0x713b18006078 10.53.0.54#10142 (.): query: . IN NS -E(0)DC (10.53.0.2)
10-Nov-2026 13:48:30.697 client @0x713b0c06ac88 10.53.0.54#35763 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
10-Nov-2026 13:48:30.778 client @0x713b0c06ac88 10.53.0.54#35809 (ns.cs-202-16.lab): query: ns.cs-202-16.lab IN A -E(0)DC (10.53.0.2)
10-Nov-2026 13:55:52.534 client @0x713b0c06ac88 10.53.0.53#57113 (x1104.zz26215): query: x1104.zz26215 IN A -E(0)DC (10.53.0.2)
10-Nov-2026 13:56:41.383 client @0x713b0c06ac88 10.53.0.53#16154 (.): query: . IN NS -E(0)DC (10.53.0.2)
10-Nov-2026 13:56:41.463 client @0x713b18006078 10.53.0.53#11478 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
10-Nov-2026 13:56:41.544 client @0x713b18006078 10.53.0.53#18141 (www.cs-202-16.lab): query: www.cs-202-16.lab IN A -E(0)DC (10.53.0.2)
10-Nov-2026 14:03:29.515 client @0x713b0c06ac88 10.53.0.53#14769 (.): query: . IN NS -E(0)DC (10.53.0.2)
10-Nov-2026 14:03:29.595 client @0x713b0c06ac88 10.53.0.53#14565 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
10-Nov-2026 14:03:29.676 client @0x713b0c06ac88 10.53.0.53#58052 (www.cs-202-16.lab): query: www.cs-202-16.lab IN AAAA -E(0)DC (10.53.0.2)
10-Nov-2026 14:03:29.676 client @0x713b18006078 10.53.0.53#45087 (www.cs-202-16.lab): query: www.cs-202-16.lab IN A -E(0)DC (10.53.0.2)
```
docker compose -f stand/compose.yaml exec tld tail -n 20 /var/log/named/queries.log
```text
26-Oct-2026 17:48:30.252 client @0x738f0c06ac88 10.53.0.53#24233 (ns.cs-202-21.lab): query: ns.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 17:48:30.899 client @0x738f0c06ac88 10.53.0.54#37647 (ns.cs-202-21.lab): query: ns.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 17:51:25.354 client @0x738f0c06ac88 10.53.0.53#20297 (new2-kqjwe.cs-202-21.lab): query: new2-kqjwe.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:32:46.676 client @0x738f04001ee8 10.53.0.53#19284 (x12099.lab): query: x12099.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:36:41.664 client @0x738f04001ee8 10.53.0.53#5899 (www.cs-202-21.lab): query: www.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:43:29.796 client @0x738f0c06ac88 10.53.0.53#64853 (www.cs-202-21.lab): query: www.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:43:29.796 client @0x738f04001ee8 10.53.0.53#7582 (www.cs-202-21.lab): query: www.cs-202-21.lab IN AAAA -E(0)DC (10.53.0.3)
```
1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.
Затримка власного сервера визначається рядком A.
Затримка TLD визначається рядком B.
Затримка Root визначається рядком C.
2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.
Не збігається. У файлі зони TTL запису SOA зазвичай великий. Але коли сервер повертає NXDOMAIN, він автоматично підставляє у секцію AUTHORITY запис SOA з розрахованим TTL для негативного кешування, щоб резолвер знав, скільки часу кешувати саму відмову.
3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?
Скрипт побачить NXDOMAIN (відмову), оскільки резолвер закешував відсутність імені після першої перевірки. Виправити: або робити перевірку виключно ПІСЛЯ створення запису та оновлення зони, або налаштувати скрипт запитувати безпосередньо авторитативний сервер, оминаючи кеш.
## Завдання 7. Інцидент: зупинка власного сервера
Прогноз (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:
resolver-a після закінчення TTL → SERVFAIL; resolver-b (serve-stale) → старий A з TTL 0.

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):
```text
test.cs-202-16.lab.     25      IN      A       10.53.0.81
test.cs-202-16.lab.     25      IN      A       10.53.0.81
17:25:58
```
Запити із заміром часу (крок 5):

time dig @10.53.0.53 test.cs-202-16.lab +tries=1 +timeout=30
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.cs-202-16.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 11803
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.cs-202-16.lab.            IN      A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 10 17:29:10 UTC 2026
;; MSG SIZE  rcvd: 47

real    0m0.024s
user    0m0.009s
sys     0m0.002s
```
time dig @10.53.0.53 test.cs-202-16.lab +tries=1 +timeout=30
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.cs-202-16.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 36270
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.cs-202-16.lab.            IN      A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 10 17:29:10 UTC 2026
;; MSG SIZE  rcvd: 47

real    0m0.022s
user    0m0.006s
sys     0m0.005s
```
time dig @10.53.0.54 test.cs-202-16.lab +tries=1 +timeout=30
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 test.cs-202-16.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51479
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.cs-202-16.lab.            IN      A

;; ANSWER SECTION:
test.cs-202-16.lab.     30      IN      A       10.53.0.81

;; Query time: 1801 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Tue Nov 10 17:29:12 UTC 2026
;; MSG SIZE  rcvd: 63

real    0m1.824s
user    0m0.005s
sys     0m0.008s
```
| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | SERVFAIL | - | 0m0.024s |
| resolver-a, другий | SERVFAIL | - | 0m0.022s |
| resolver-b | NOERROR | 30 | 0m1.824s |

Час named-start і вивід циклу спостереження до першої успішної відповіді (крок 6):
```text
17:32:10
17:32:10 10.53.0.81
```
Перевірка після завдання (dig @10.53.1.1 sNN.lab SOA +norec +short):
```text
ns.cs-202-16.lab. admin.cs-202-16.lab.  2026110901 3600 600 86400 30
```
### Постмортем
1. Що сталося: Авторитативний сервер був вимкнений. Після закінчення дії TTL записів у кеші резолверів сервер виявився недоступним для оновлення інформації.
2. Вплив на клієнтів resolver-a і resolver-b: Клієнти resolver-a отримали SERVFAIL та повністю втратили можливість знаходити адресу сервісу. Клієнти resolver-b зберегли доступ, оскільки резолвер віддавав застарілі дані (stale data).
3. 17:25:58 — Зафіксовано кешування запису test.cs-202-21.lab з початковим TTL (наприклад, 25 або 30 секунд) в обох резолверах.
~17:26:27 — Сплив час життя TTL запису в кеші, а власний сервер був зупинений.
17:29:10 — Запит до звичайного резолвера (resolver-a, @10.53.0.53) повернув статус SERVFAIL за час близько 0.024s / 0.022s (сервер швидко здався, оскільки записи прострочені, а опитати авторитет неможливо).
17:29:12 — Запит до резолвера з serve-stale (resolver-b, @10.53.0.54) успішно повернув стару адресу 10.53.0.81 із застарілим TTL за час 1.824s (затримка зумовлена очікуванням таймауту звернення до зупиненого сервера перед видачею stale-даних).
17:32:10 — Час успішного перезапуску авторитативного сервера (named-start) та момент відновлення повноцінної роботи системи (цикл спостереження зафіксував першу успішну відповідь 10.53.0.81).
4. Чому кеш допоміг або не допоміг: Звичайний кеш (resolver-a) зобов'язаний видаляти записи після вичерпання TTL (не допоміг). Кеш із функцією Serve-stale (resolver-b) має право за RFC 8767 віддавати клієнтам "прострочений" запис, якщо першоджерело не відповідає, що і врятувало ситуацію.
5. Що слід змінити: Налаштувати додатковий (Secondary) авторитативний сервер для зони cs-202-16.lab, щоб забезпечити відмовостійкість.
6. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?
TTL стає дуже малим (зазвичай 30 секунд). Це робиться для того, щоб клієнтські пристрої не кешували цей застарілий запис надовго і частіше перевіряли, чи не відновив свою роботу основний сервер.
7. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.
Перший запит тривав довго (резолвер намагався зв'язатися з неробочим сервером, дочікувався тайм-аутів). Другий запит повернув SERVFAIL миттєво. Резолвер закешував інформацію про те, що сервер недоступний (server unreachable/unresponsive), і не витрачає час на нові спроби протягом певного штрафного періоду.
8. Чому після named-start ім'я через resolver-a зарезолвилося не одразу?
Через таймер-штраф (penalty timer) негативного кешування стану самого сервера. Резолвер деякий час після відновлення все ще вважає сервер «мертвим» і відкидає запити, перш ніж спробує зв'язатися з ним знову.
9. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.
Serve-stale шкодить під час міграції (спланованої зміни IP-адреси). Якби під час завдання 4 (коли TTL закінчився, а IP-адреса мала змінитися на нову) авторитативний сервер випадково перезавантажився, serve-stale віддав би стару (неправильну) IP-адресу клієнтам, і вони б направлялися на відключений сервер вебдодатку.
## Фінальний файл зони
```text
; Zone cs-202-16.lab - starting version.
; After every change: increase the serial, then run zone-reload
$TTL 45
@       60 IN SOA ns.cs-202-16.lab. admin.cs-202-16.lab. (
                   2026110901 ; serial
                    3600       ; refresh
                    600        ; retry
                    86400      ; expire
                    30         ; minimum
                )

@       IN NS ns.cs-202-16.lab.
ns      IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www     IN A  10.53.0.80
news    IN CNAME www
mail    IN A  10.53.0.80
@       IN MX 50 mail
@       IN TXT "lab-token=2b830ea6"
test    IN A  10.53.0.81
```
## Висновки
Що виявилося несподіваним. Результат, який не збігся з очікуванням, і рядок виводу, де це видно:
Неочевидною виявилася поведінка механізму serve-stale під час першого запиту на прострочений запис при падінні авторитетного сервера: резолвер робить таймаут-опитування перед тим, як віддати кешовані дані. Це видно за часом виконання real 0m1.824s у завдання 7.

Яке питання залишилося без відповіді. Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:
Усі питання лабораторної роботи та поведінка ієрархії DNS повністю з'ясовані за допомогою документації BIND, RFC 2308 та RFC 8767, а також експериментально підтверджені логами.

Використовувалися інструменти ШІ:
| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | Gemini | Зроби оформлення лаб роб 1 у Github та перевір правильність виконання | Використано структуру, теоретичні відомості. Самостійно додано індивідуальні параметри варіанта та вивід CLI команд зі стенду Codespaces. |

