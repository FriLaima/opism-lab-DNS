
# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ | Примаченко Ксенія Анатоліївна |
| Група | КБЗІ-2.02 |
| Ідентифікатор (`sNN`) | cs-202-20 |
| Дата виконання | 02-03.11.2026 |
| Середовище | GitHub Codespaces |
| Версія `dig` | 9.18.39 |

## Параметри варіанта

| Параметр | Значення |
|---|---|
| `$TTL`, с | 90 |
| TTL запису SOA, с | 20 |
| Поле `minimum` SOA, с | 90 |
| Псевдонім (CNAME на `www`) | api |
| Другий хост і його TTL, с | test-20 |
| Пріоритет MX | 35 |
| Токен TXT | 5a88a2c0 |
| Ім'я для завдання 5 | `new-swsmz.cs-202-20.lab` |

## Завдання 1. Складові URI

### 1.1. Розбір

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | https | - | www.cs-202-20.lab | 80 за замовчуванням | /media/orders | id=76&sort=uk | top |
| 2 | http | - | api.cs-202-20.lab | 80 | /orders/ | id=%D0%BA%D0%B8%D1%97%D0%B2 | - |
| 3 | http | user:pass | test.cs-202-20.lab | 80 за замовчуванням | /media | - | section-2 |
| 4 | https | - | www.cs-202-20.lab | 8443 | 	/orders | - | - |
| 5 | mailto | - | - | - | admin@cs-202-20.lab | subject=media | - |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | www.cs-202-20.lab | www.cs-202-20.lab | /media/orders?id=76&sort=uk |
| 2 | api.cs-202-20.lab | api.cs-202-20.lab | /orders/?id=%D0%BA%D0%B8%D1%97%D0%B2 |
| 3 | test.cs-202-20.lab | test.cs-202-20.lab | /media |
| 4 | www.cs-202-20.lab | www.cs-202-20.lab:8443 | /orders |
| 5 | - | - | - |

**Декодоване значення параметра URI 2:** D0BA = к, D0B8 = и, D197 = ї, D0B2 = в (київ)

**Чому URI 5 не має складової host, хоча містить символ `@`:** тому що mailto — це схема для електронної пошти, а не HTTP. Символ @ розділяє ім'я користувача та домен у поштовій адресі, але не визначає складову host URI.

### 1.2. Перевірка (після завдання 3)

```
root@ns-cs-202-20:/# curl -v 'http://www.cs-202-20.lab/media/orders?id=76&sort=uk#top'; curl -v 'http://api.cs-202-20.lab:80/orders/?id=%D0%BA%D0%B8%D1%97%D0%B2'; curl -v 'http://user:pass@test.cs-202-20.lab/media#section-2'
* Host www.cs-202-20.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to www.cs-202-20.lab (10.53.0.81) port 80
> GET /media/orders?id=76&sort=uk HTTP/1.1
> Host: www.cs-202-20.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 02 Nov 2026 23:42:21 GMT
< Content-Type: text/plain
< Content-Length: 138
< Connection: keep-alive
< 
server: web-b
Host: www.cs-202-20.lab
request URI: /media/orders?id=76&sort=uk
path: /media/orders
query: id=76&sort=uk
client: 10.53.1.1
* Connection #0 to host www.cs-202-20.lab left intact
* Host api.cs-202-20.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to api.cs-202-20.lab (10.53.0.81) port 80
> GET /orders/?id=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: api.cs-202-20.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 02 Nov 2026 23:42:21 GMT
< Content-Type: text/plain
< Content-Length: 156
< Connection: keep-alive
< 
server: web-b
Host: api.cs-202-20.lab
request URI: /orders/?id=%D0%BA%D0%B8%D1%97%D0%B2
path: /orders/
query: id=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
* Connection #0 to host api.cs-202-20.lab left intact
* Host test.cs-202-20.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to test.cs-202-20.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /media HTTP/1.1
> Host: test.cs-202-20.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 02 Nov 2026 23:42:21 GMT
< Content-Type: text/plain
< Content-Length: 98
< Connection: keep-alive
< 
server: web-b
Host: test.cs-202-20.lab
request URI: /media
path: /media
query: 
client: 10.53.1.1
* Connection #0 to host test.cs-202-20.lab left intact
```

**Значення заголовка `Authorization` і результат декодування:**

```
root@ns-cs-202-20:/# echo 'dXNlcjpwYXNz' | base64 -d
user:pass
```

**URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:**

```
root@ns-cs-202-20:/# curl -v 'https://www.cs-202-20.lab:8443/media/../orders'
* Host www.cs-202-20.lab:8443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:8443...
* connect to 10.53.0.81 port 8443 from 10.53.1.1 port 49698 failed: Connection refused
* Failed to connect to www.cs-202-20.lab port 8443 after 0 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.cs-202-20.lab port 8443 after 0 ms: Couldn't connect to server
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | /orders | /orders |
| `--path-as-is` | /media/../orders | /orders |

**Розбіжності між URI та тим, що отримав сервер, і їх пояснення:** у звичайному запиті curl самостійно нормалізує URI: видаляє `/media/..` і надсилає серверу `/orders`. Тому request URI та path збігаються. Під час використання `--path-as-is` клієнт передає початковий URI `/media/../orders` без нормалізації. Сервер отримує саме такий request URI, але самостійно обробляє конструкцію `..` і визначає path як `/orders`.

## Завдання 2. Проходження ієрархії вручну

**Команди**
```
dig @10.53.0.2 ns.cs-202-20.lab A +norec
dig @10.53.0.3 ns.cs-202-20.lab A +norec
dig @10.53.1.1 ns.cs-202-20.lab A +norec
```
**Вивід**
```
root@ns-cs-202-20:/# dig @10.53.0.2 ns.cs-202-20.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.cs-202-20.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53237
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ccba34265fa333ad010000006ae89abe541f13413afad1f1 (good)
;; QUESTION SECTION:
;ns.cs-202-20.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 80 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Mon Nov 02 14:14:22 UTC 2026
;; MSG SIZE  rcvd: 117

root@ns-cs-202-20:/# dig @10.53.0.3 ns.cs-202-20.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.3 ns.cs-202-20.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34318
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 799160644d209475010000006ae89c0611c05bcdb5edbd38 (good)
;; QUESTION SECTION:
;ns.cs-202-20.lab.              IN      A

;; AUTHORITY SECTION:
cs-202-20.lab.          3600    IN      NS      ns.cs-202-20.lab.

;; ADDITIONAL SECTION:
ns.cs-202-20.lab.       3600    IN      A       10.53.1.1

;; Query time: 82 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Mon Nov 02 14:19:50 UTC 2026
;; MSG SIZE  rcvd: 103

root@ns-cs-202-20:/# dig @10.53.1.1 ns.cs-202-20.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.cs-202-20.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39440
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 332764f1398188f7010000006ae89c12b02ba156b64b7a50 (good)
;; QUESTION SECTION:
;ns.cs-202-20.lab.              IN      A

;; ANSWER SECTION:
ns.cs-202-20.lab.       90      IN      A       10.53.1.1

;; AUTHORITY SECTION:
cs-202-20.lab.          90      IN      NS      ns.cs-202-20.lab.

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 14:20:02 UTC 2026
;; MSG SIZE  rcvd: 103
```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR | qr | 0 | 1 | 2 |
| 10.53.0.3 | NOERROR | qr | 0 | 1 | 2 |
| 10.53.1.1 | NOERROR | qr, aa | 1 | 1 | 1 |

**Вивід `dig ns.cs-202-20.lab A +trace`:**

```
root@ns-cs-202-20:/# dig ns.cs-202-20.lab A +trace

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.cs-202-20.lab A +trace
;; global options: +cmd
.                       79741   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms

cs-202-20.lab.          3600    IN      NS      ns.cs-202-20.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 40 ms

ns.cs-202-20.lab.       90      IN      A       10.53.1.1
cs-202-20.lab.          90      IN      NS      ns.cs-202-20.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.cs-202-20.lab) in 1 ms
```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 sNN.lab NS +norec` | 3600 | - | AUTHORITY SECTION |
| `dig @10.53.1.1 sNN.lab NS +norec` | 90 | + | ANSWER SECTION |

**1. Які відповіді є відсиланнями і за якими ознаками це визначено?**
Відсиланням є відповідь від сервера 10.53.0.3. У ній `ANSWER: 0`, але в секції AUTHORITY міститься NS-запис `ns.cs-202-20.lab`, який вказує на авторитетний DNS-сервер домену. У секції ADDITIONAL також надано його IP-адресу `10.53.1.1`. Відповідь від `10.53.1.1` не є відсиланням, оскільки містить остаточний запис у секції ANSWER.
**2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?**
Запис у секції ADDITIONAL надає IP-адресу DNS-сервера, на який потрібно передати наступний запит. У цьому випадку для `ns.cs-202-20.lab` вказана адреса `10.53.1.1`. Завдяки цьому резолверу не потрібно виконувати окремий DNS-запит, щоб дізнатися IP-адресу цього сервера.
**3. Яка з двох копій запису NS є авторитативною і чому?**
Авторитативною є копія NS-запису на власному DNS-сервері `10.53.1.1`. Це підтверджує прапорець aa у відповіді, який означає, що сервер є авторитетним для цього домену. TLD-сервер `10.53.0.3` лише надає відсилання до DNS-сервера домену, тому його NS-запис у секції AUTHORITY не є авторитетною відповіддю для цієї зони. На власному сервері NS-запис розміщений у секції ANSWER, а прапорець aa підтверджує його авторитетність.

## Завдання 3. Заповнення власної зони

**Вивід `zone-reload`:**

```
root@ns-cs-202-20:/# zone-reload
zone cs-202-20.lab/IN: loaded serial 2026110202
OK
zone reload queued
```

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR | + | 10.53.0.80 | 90 | Так |
| псевдонім CNAME (api) | NOERROR | + | www.cs-202-20.lab. | 90 | Так |
| `mail` A | NOERROR | + | 10.53.0.80 | 90 | Так |
| `@` MX | NOERROR | + | 35 mail.cs-202-20.lab. | 90 | Так |
| `@` TXT | NOERROR | + | "lab-token=5a88a2c0" | 90 | Так |
| другий хост A | NOERROR | + | 10.53.0.81 | 90 | Так |

**Порівняння відповідей резолвера та власного сервера:**

```
root@ns-cs-202-20:/# dig api.cs-202-20.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> api.cs-202-20.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43130
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;api.cs-202-20.lab.             IN      A

;; ANSWER SECTION:
api.cs-202-20.lab.      90      IN      CNAME   www.cs-202-20.lab.
www.cs-202-20.lab.      90      IN      A       10.53.0.80

;; Query time: 41 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 19:16:26 UTC 2026
;; MSG SIZE  rcvd: 80

root@ns-cs-202-20:/# dig @10.53.1.1 api.cs-202-20.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 api.cs-202-20.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57687
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 862b347a5e8e4c58010000006ae8e19e5676fe73eeada5bb (good)
;; QUESTION SECTION:
;api.cs-202-20.lab.             IN      A

;; ANSWER SECTION:
api.cs-202-20.lab.      90      IN      CNAME   www.cs-202-20.lab.
www.cs-202-20.lab.      90      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-20.lab.          90      IN      NS      ns.cs-202-20.lab.

;; ADDITIONAL SECTION:
ns.cs-202-20.lab.       90      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 19:16:46 UTC 2026
;; MSG SIZE  rcvd: 141

root@ns-cs-202-20:/# dig cs-202-20.lab MX

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> cs-202-20.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11362
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;cs-202-20.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-20.lab.          90      IN      MX      35 mail.cs-202-20.lab.

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 19:17:11 UTC 2026
;; MSG SIZE  rcvd: 63

root@ns-cs-202-20:/# dig @10.53.1.1 cs-202-20.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-20.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64422
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 9bc12d1f535c45f1010000006ae8e1c5b60e39513c747c5b (good)
;; QUESTION SECTION:
;cs-202-20.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-20.lab.          90      IN      MX      35 mail.cs-202-20.lab.

;; AUTHORITY SECTION:
cs-202-20.lab.          90      IN      NS      ns.cs-202-20.lab.

;; ADDITIONAL SECTION:
mail.cs-202-20.lab.     90      IN      A       10.53.0.80
ns.cs-202-20.lab.       90      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 19:17:25 UTC 2026
;; MSG SIZE  rcvd: 140
```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a | qr rd ra | 2: CNAME, A | 0 | 1: OPT | 90 |
| псевдонім через 10.53.1.1 | qr aa | 2: CNAME, A |	1: NS | 2: A, OPT | 90 |
| MX через resolver-a | qr rd ra | 1: MX | 0 | 1: OPT | 90 |
| MX через 10.53.1.1 | qr aa | 1: MX | 1: NS | 	3: A, A, OPT | 90 |

**Пояснення відмінностей:** через `resolver-a` відповіді містять прапорці `rd` і `ra`, оскільки резолвер виконує рекурсивний пошук. Через власний DNS-сервер `10.53.1.1` отримуємо авторитетні відповіді з прапорцем `aa`. У відповідях власного сервера додатково присутній NS-запис у AUTHORITY та IP-адреси серверів у ADDITIONAL. Для `api` повертаються два записи: `CNAME` та `A` (`10.53.0.80`). Для MX-запиту повертається поштовий сервер `mail.cs-202-20.lab` із пріоритетом 35 та його IP-адреса `10.53.0.80`.

**Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:**

```
root@ns-cs-202-20:/# zone-check
dns_master_load: /lab/zone/cs-202-20.lab.zone:23: cs-202-20.lab: CNAME and other data
zone cs-202-20.lab/IN: loading from master file /lab/zone/cs-202-20.lab.zone failed: CNAME and other data
zone cs-202-20.lab/IN: not loaded due to errors.
```

**1. Який TTL має кожен запис зони і звідки це значення взято?**
Записи SOA і `test` мають TTL 20 с, бо він заданий у файлі зони явно (`@ 20 IN SOA, test 20 IN A`). Усі інші записи (`NS, ns, www, api CNAME, mail, MX, TXT, new-swsmz`) мають 90 с, бо TTL у них не вказано і вони успадковують `$TTL 90`. Це видно у виводах d`ig: 90 для www, api, MX, TXT`. TTL 3600 для NS у відповіді TLD-сервера до зони не належить, це TTL запису делегування на сервері `lab.`.
**2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?**
Два: `api CNAME www.cs-202-20.lab.` і `www.cs-202-20.lab. A 10.53.0.80`. Запитували тип A для `api`, а `api` є лише псевдонімом. Тому сервер повертає сам CNAME і одразу додає A-запис цілі, щоб клієнтові не потрібен був окремий запит. Це видно і на `resolver-a`, і на `10.53.1.1 (ANSWER: 2)`.
**3. Чому запис CNAME неможливо розмістити для імені `@`?**
CNAME означає, що ім'я є повним псевдонімом іншого імені, тому для нього не може існувати жодних інших записів. Вершина зони @ обов'язково містить SOA і NS (а тут ще MX і TXT). Це підтверджує вивід `zone-check: CNAME and other data`, зона не завантажилась (`not loaded due to errors`).

## Завдання 4. TTL і видимість змін

**Два запити з інтервалом 5 с:**

```
root@ns-cs-202-20:/# dig www.cs-202-20.lab +noall +answer                 
www.cs-202-20.lab.      90      IN      A       10.53.0.80
root@ns-cs-202-20:/# sleep 5 
root@ns-cs-202-20:/# dig www.cs-202-20.lab +noall +answer
www.cs-202-20.lab.      80      IN      A       10.53.0.80
```

**Прогноз (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:** резолвер зберігає DNS-запис www.cs-202-20.lab у кеші на 90 секунд. Через 5 секунд TTL зменшився до 80 секунд, отже, у кеші залишилося приблизно 80 секунд. Після зміни IP-адреси та виконання zone-reload резолвер продовжить повертати стару адресу 10.53.0.80, доки не завершиться залишок TTL (R). Після цього він отримає нову адресу 10.53.0.81 під час наступного запиту.

**Перевірка, що сервер до `zone-reload` повертає стару адресу:**

```
root@ns-cs-202-20:/# dig @10.53.1.1 www.cs-202-20.lab +norec +short
10.53.0.80
```

**Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):**

```
root@ns-cs-202-20:/# date +%T; dig www.cs-202-20.lab +noall +answer; zone-reload; date +%T
20:59:41
www.cs-202-20.lab.      90      IN      A       10.53.0.80
zone cs-202-20.lab/IN: loaded serial 2026110212
OK
zone reload queued
20:59:41
```

**Очікуваний час появи нової адреси (підставити R у формулу):** 20:59:41 + 90с = 21:01:11

**Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):** 

```
20:59:56 10.53.0.80
20:59:58 10.53.0.80
...
21:01:08 10.53.0.80
21:01:10 10.53.0.80
21:01:12 10.53.0.81
```

**Рядок `server:` з відповіді `curl`:** server: web-b 

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 20:59:41 | 21:01:11 | 21:01:12 | 91 | 90 |

**1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?**
Виміряно 91 с проти прогнозу 90 с, різниця 1 с. Цикл спостереження опитує кеш кожні 2 с (21:01:10 ще стара адреса, наступне опитування 21:01:12), тому нова адреса не могла з'явитися точніше ніж із кроком опитування. До цього додається час виконання самого dig і те, що TTL у кеші рахується цілими секундами. Фактичне закінчення TTL (21:01:11) потрапило між двома опитуваннями.
**2. Якою була б найбільша можлива затримка для цього запису і за яких умов?**
Найбільша затримка дорівнює повному TTL, тобто 90 с. Вона виникає, коли резолвер закешував запис безпосередньо перед зміною: залишок R тоді ≈ 90 с, як у моєму вимірюванні (dig показав 90 саме перед zone-reload). Якщо запис закешовано раніше, R менший і затримка коротша. Якщо між клієнтом і сервером кілька кешів, що оновилися в різний час, у гіршому випадку затримка зростає.
**3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?**
За 24 год або більше до міграції треба знизити TTL запису до малого значення (наприклад, 60–300 с). Після цього слід почекати, поки спливе старий TTL (86 400 с), щоб усі кеші перейшли на короткий. Потім змінюємо адресу, піднімаємо serial і робимо zone-reload. Нова адреса стане видимою за кілька хвилин. Старий сервер залишаємо працювати на час переходу. Після стабілізації TTL повертаємо до звичайного значення.

## Завдання 5. Негативне кешування

**Прогноз (записати до вимірювання). Формула за RFC 2308 і розраховане значення:**за RFC 2308 час негативного кешування дорівнює мінімальному значенню між TTL SOA-запису та полем minimum (min(20; 90) = 20 секунд). Після отримання відповіді NXDOMAIN резолвер зберігатиме негативний результат протягом 20 секунд.

**Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:**

```
root@ns-cs-202-20:/# dig @10.53.1.1 new-swsmz.cs-202-20.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-swsmz.cs-202-20.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 41032
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 126d0ef0914e3caa010000006ae90124d5e8594ee918fbf9 (good)
;; QUESTION SECTION:
;new-swsmz.cs-202-20.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-20.lab.          20      IN      SOA     ns.cs-202-20.lab. admin.cs-202-20.lab. 2026110212 3600 600 86400 90

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 21:31:16 UTC 2026
;; MSG SIZE  rcvd: 125
```

**Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):**

```
root@ns-cs-202-20:/# date +%T; dig new-swsmz.cs-202-20.lab
22:04:25

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new-swsmz.cs-202-20.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 9104
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-swsmz.cs-202-20.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-20.lab.          20      IN      SOA     ns.cs-202-20.lab. admin.cs-202-20.lab. 2026110216 3600 600 86400 90

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 22:04:25 UTC 2026
;; MSG SIZE  rcvd: 97

root@ns-cs-202-20:/# zone-reload; date +%T
zone cs-202-20.lab/IN: loaded serial 2026110217
OK
zone reload queued
22:04:30
root@ns-cs-202-20:/# dig new-swsmz.cs-202-20.lab +norec +short
```

**Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):**

```
22:04:39 status: NXDOMAIN
22:04:41 status: NXDOMAIN
...
22:04:45 status: NXDOMAIN
22:04:47 status: NOERROR
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 22:04:25 | 22:04:30 | 22:04:47 | 22 | 20 |

**1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?**
Коли ім'я не існує (NXDOMAIN), запису, у якого можна взяти TTL, немає. За RFC 2308 сервер додає в AUTHORITY запис SOA зони, а резолвер кешує негативну відповідь на min(TTL SOA, minimum). Тут це min(20; 90) = 20 с. Запитуваного запису ще не існує, тож він не може керувати власним негативним кешем.
**2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.**
Збігається: у файлі @ 20 IN SOA, у відповіді 20 IN SOA. Але збіг частково випадковий. Сервер у негативній відповіді ставить TTL = min(TTL SOA, minimum). Тому що 20 < 90, береться 20. Якби TTL SOA у файлі був більшим за minimum (наприклад 3600), у відповіді було б 90, і значення розійшлися б.
**3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?**
Перша перевірка через резолвер закешує NXDOMAIN на 20 с. Друга перевірка одразу після створення запису все ще отримає NXDOMAIN з кешу. У моєму експерименті запис створено о 22:04:30, а ім'я зарезолвилося лише о 22:04:47. Виправлення:
перевіряти після створення авторитативний сервер (dig @10.53.1.1 ... +norec), а не резолвер; або повторювати перевірку з паузою, більшою за негативний TTL; або не перевіряти ім'я перед створенням; або зменшити minimum у SOA.

## Завдання 6. Затримка пошуку імені

**Прогноз (записати до вимірювання). До яких серверів звернеться resolver-a:**

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | Власний сервер 10.53.1.1 | Кеш |
| B: нове ім'я в зоні `lab.` | 	TLD 10.53.0.3 | Кеш |
| C: ім'я в неіснуючому TLD |	Root 10.53.0.2 | Кеш |
| D: `www.sNN.lab` після перезапуску резолвера | Root -> TLD -> власний сервер | Кеш |

**Вивід вимірювань:**

```
root@ns-cs-202-20:/# A=y$RANDOM.cs-202-20.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
;; Query time: 0 msec
;; Query time: 1 msec
root@ns-cs-202-20:/# B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
;; Query time: 39 msec
;; Query time: 0 msec
root@ns-cs-202-20:/# C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 80 msec
;; Query time: 0 msec
@FriLaima ➜ /workspaces/opism-lab-DNS (master) $ docker compose -f stand/compose.yaml restart resolver-a
[+] restart 0/1
 ⠴ Container dnslab-resolver-a-1 Restarting                             0.5s
@FriLaima ➜ /workspaces/opism-lab-DNS (master) $ docker compose -f stand/compose.yaml exec cs-202-20 bash -l
root@ns-cs-202-20:/# dig www.cs-202-20.lab | grep 'Query time'; dig www.cs-202-20.lab | grep 'Query time'
;; Query time: 401 msec
;; Query time: 0 msec
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | 0 | 1 | Власний сервер 10.53.1.1 |
| B | 39 | 0 | TLD 10.53.0.3 |
| C | 80 | 0 | Root 10.53.0.2 |
| D | 401 | 0 | Root 10.53.0.2 -> TLD 10.53.0.3 -> власний сервер 10.53.1.1 |

**Значення `time_namelookup` з `curl` після перезапуску резолвера:**
```
root@ns-cs-202-20:/# curl -so /dev/null -w '%{time_namelookup}\n' http://www.cs-202-20.lab/
0.000660
```
**Фрагменти журналів запитів root, TLD і власного сервера для рядка D:**

```
02-Nov-2026 22:35:29.747 query: www.cs-202-20.lab IN A
02-Nov-2026 22:35:29.867 query: www.cs-202-20.lab IN A
02-Nov-2026 22:35:29.907 query: www.cs-202-20.lab IN A
```

**1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.**
Root ≈ 80 мс (рядок C: 80 мс, запит іде лише до root; збігається з +trace: 80 ms).
TLD ≈ 40 мс (рядок B: 39 мс, до TLD).
Власний сервер ≈ 0–1 мс (рядок A: 0 мс).
Сума для рядка D: 80 + 40 + 1 ≈ 121 мс. Виміряно 401 мс, тобто близько 280 мс зайвого. Журнали (29.747 → 29.867 → 29.907) підтверджують послідовність root → TLD → власний сервер. Імовірна причина зайвих мілісекунд: холодний старт щойно перезапущеного резолвера (ініціалізація, прайминг кореневих серверів), а не затримка самих серверів.
**2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?**
Резолвер уже має в кеші делегування lab. (TTL 172800 с) і cs-202-20.lab. (3600 с), отримані раніше. Нове лише саме ім'я, тому резолвер одразу питає власний сервер 10.53.1.1.
**3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?**
Для кожного нового домену потрібен окремий DNS-пошук. З холодним кешем кожен може коштувати 40–400 мс, як у рядках B–D. Якщо браузер дізнається про домени послідовно, затримки додаються, а на першому відкритті кешу немає. Тому перше відкриття повільніше за повторні, де всі відповіді вже в кеші (0–1 мс). Зменшити затримку можна меншою кількістю доменів або dns-prefetch.

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:** запис www.cs-202-20.lab має TTL 90 с, а запит виконується після спливання цього часу. Тому звичайний кешований запис в обох резолверах уже буде застарілим. Для отримання нової відповіді потрібен авторитативний сервер 10.53.1.1, який зупинено. resolver-a поверне SERVFAIL без записів в ANSWER, оскільки не зможе отримати актуальну відповідь від недоступного сервера. Відповідь буде не миттєвою: резолвер деякий час очікуватиме на відповідь від сервера (можливо, кілька секунд). resolver-b (із підтримкою serve-stale, RFC 8767) зможе повернути застарілий запис із кешу: NOERROR, IP-адресу 10.53.0.80 або 10.53.0.81 залежно від того, який запис було закешовано із коротким TTL.

**Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):**

```
root@ns-cs-202-20:/# dig @10.53.0.53 test.cs-202-20.lab +noall +answer
test.cs-202-20.lab.     20      IN      A       10.53.0.81
root@ns-cs-202-20:/# dig @10.53.0.54 test.cs-202-20.lab +noall +answer
test.cs-202-20.lab.     20      IN      A       10.53.0.81
root@ns-cs-202-20:/# named-stop
root@ns-cs-202-20:/# date +%T
23:17:09
root@ns-cs-202-20:/# sleep 30 
```

**Запити із заміром часу (крок 5):**

```
root@ns-cs-202-20:/# time dig @10.53.0.53 test.cs-202-20.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.cs-202-20.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.051s
user    0m0.007s
sys     0m0.003s
root@ns-cs-202-20:/# time dig @10.53.0.53 test.cs-202-20.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.cs-202-20.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.051s
user    0m0.006s
sys     0m0.005s
root@ns-cs-202-20:/# time dig @10.53.0.54 test.cs-202-20.lab +tries=1 +timeout=30

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 test.cs-202-20.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40336
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.cs-202-20.lab.            IN      A

;; ANSWER SECTION:
test.cs-202-20.lab.     30      IN      A       10.53.0.81

;; Query time: 1799 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Mon Nov 02 23:20:39 UTC 2026
;; MSG SIZE  rcvd: 63


real    0m1.821s
user    0m0.006s
sys     0m0.005s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | тайм-аут | - | 30,051 с |
| resolver-a, другий | тайм-аут | - | 30,051 с |
| resolver-b | NOERROR | 10.53.0.81, TTL 30 с | 1,821 с |

**Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):**

```
root@ns-cs-202-20:/# named-start; date +%T; while true; do echo "$(date +%T) $(dig @10.53.0.53 test.cs-202-20
.lab +short +tries=1 +timeout=3)"; sleep 2; done
23:25:38
23:25:38 10.53.0.81
```

**Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):**

```
ns.cs-202-20.lab. admin.cs-202-20.lab. 2026110217 3600 600 86400 90
```

### Постмортем

1. **Що сталося:** о 23:17:09 командою named-stop зупинили власний авторитативний DNS-сервер 10.53.1.1 зони cs-202-20.lab. Він був недоступний до named-start о 23:25:38 (приблизно 8 хв 29 с).
2. **Вплив на клієнтів resolver-a і resolver-b:** клієнти resolver-a після спливання TTL (20 с) не отримували відповіді: обидва запити завершилися тайм-аутом 30,05 с. Клієнти resolver-b із serve-stale отримали відповідь 10.53.0.81 (TTL 30 с) за 1,8 с.
3. **Хронологія з виміряним часом:** 23:17:09: зупинка сервера. 23:17:29: спливає TTL 20 с записів у кешах. 23:20:09 і ~23:20:39: запити до resolver-a (30,05 с кожен) і resolver-b (NOERROR, 1,8 с). 23:25:38: named-start, resolver-a знову видає 10.53.0.81.
4. **Чому кеш допоміг або не допоміг:** звичайний кеш не допоміг, бо TTL 20 с дуже малий і давно спливе. Допоміг лише serve-stale на resolver-b: він віддав застарілий, але робочий запис.
5. **Що слід змінити:** додати другий авторитативний сервер (NS) зони; увімкнути serve-stale на всіх резолверах;
збільшити TTL для стабільних записів; налаштувати моніторинг доступності DNS-сервера.

**1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?**
30 с (test.cs-202-20.lab. 30 IN A 10.53.0.81), хоча оригінальний TTL був 20 с і вже спливів. Це TTL застарілої відповіді в serve-stale (RFC 8767 рекомендує 30 с). Він короткий, щоб клієнти не тримали застарілу відповідь довго і швидко перепитали, коли сервер повернеться.
**2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.**
Результати однакові: обидва завершилися тайм-аутом за 30,051 с, і resolver-a не повернув навіть SERVFAIL. 30 с дорівнює моєму +timeout=30, тобто відповіді від resolver-a не було зовсім. Імовірно, увесь цей час резолвер намагався дістатися до 10.53.1.1: надсилав запит, чекав, повторював із наростаючими інтервалами, і не встиг видати помилку за 30 с. Невдачу він не закешував, тому другий запит повторив ту саму поведінку. Мій прогноз про швидкий SERVFAIL не підтвердився.
**3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?**
У моєму виводі відповідь прийшла вже в першому циклі (23:25:38, 10.53.0.81), тобто практично одразу після старту, і затримки я не спостерігала. Загалом затримка після запуску можлива, бо named має завантажити зону, а резолвер, який перед цим збирав таймаути, може тримати сервер у стані недоступного й повторити запит не одразу.
**4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.**
Коли дані змінили навмисно, а авторитативний сервер недоступний. Резолвер продовжить віддавати стару адресу, як у завданні 4 (старий 10.53.0.80 після зміни на 10.53.0.81). Наприклад, сервіс перенесли, а на старій адресі вже нічого немає, або запис свідомо видалили чи замінили через компрометацію. Клієнти й далі йтимуть на стару адресу, хоча авторитативні дані вже інші.

## Фінальний файл зони

```
; Zone cs-202-20.lab  ^`^t starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 90
@     20 IN SOA ns.cs-202-20.lab. admin.cs-202-20.lab. (
            2026110217 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            90         ; minimum
        )
@     IN NS ns.cs-202-20.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
@       IN NS    ns.cs-202-20.lab.
ns      IN A     10.53.1.1
www     IN A     10.53.0.81
api     IN CNAME www
mail    IN A     10.53.0.80
@       IN MX    35 mail
@       IN TXT   "lab-token=5a88a2c0"
test 20 IN A     10.53.0.81
new-swsmz IN A 10.53.0.80

```

## Висновки

**Що виявилося несподіваним.**

Результат, який не збігся з очікуванням, і рядок виводу, де це видно: немає

**Яке питання залишилося без відповіді.**

Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося: немає

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | GPT-6 OpenAI | перефразуй мої відповіді на питання більш документальною мовою | Перефразування |
| 2 | Claude Sonnet 5.5 | у моєму файлі README.md є частина "Постмортем", допоможи відповісти на таке питання: "Що слід змінити:". Зроби аналіз файлу та виводів в ньому | Використано для відповіді на це питання |
