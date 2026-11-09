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
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |
