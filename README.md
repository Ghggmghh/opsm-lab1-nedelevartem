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
dig @10.53.0.2 ns.cs-202-21.lab A +norec
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
dig @10.53.0.3 ns.cs-202-21.lab A +norec
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
dig @10.53.1.1 ns.cs-202-21.lab A +norec
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
dig files.cs-202-17.lab
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
dig @10.53.1.1 files.cs-202-21.lab +norec
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
dig cs-202-21.lab MX
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
root@ns-cs-202-21:/# dig @10.53.1.1 www.cs-202-21.lab +norec +short
10.53.0.80
```
