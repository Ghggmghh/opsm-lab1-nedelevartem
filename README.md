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
