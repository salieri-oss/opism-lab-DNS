# DNS lab stand (ОПІСМ, Topic 4)
<!--
Бланк звіту до лабораторної роботи «DNS».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Коментарі такого вигляду в GitHub не відображаються; видаляти їх не обов'язково.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.

# Лабораторна робота. DNS

|Поле|Значення|
|-|-|
|ПІБ|Василіогло Микола Сергійович|
|Група|F5-2.01|
|Ідентифікатор (`sNN`)|4|
|Дата виконання|27 10 2026|

Параметри варіанта
<!-- Переписати з картки stand/cards/sNN.md. -->
Параметр	Значення
`$TTL`, с	90
TTL запису SOA, с	15
Поле `minimum` SOA, с	20
Псевдонім (CNAME на `www`)	docs
Другий хост і його TTL, с	edge, 20
Пріоритет MX	40
Токен TXT	lab-token=ad25406f
Ім'я для завдання 5	new-fvikr.s04.lab
Завдання 1. Складові URI
1.1. Розбір
<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->
№	scheme	userinfo	host	port	path	query	fragment
1	http	-	www.s04.lab	80(за замовч.)	/search/reports	limit=95&sort=x	section-2
2	http	-	docs.s04.lab	80(за замовч.)	/reports/	limit=%D0%BA%D0%B8%D1%97%D0%B2	-
3	http	user:pass	edge.s04.lab	80(за замовч.)	/search	-	top
4	https	-	www.s04.lab	8443	/search/../reports	-	-
5	mailto	-	-	-	admin@s04.lab	subject=search	-

№	Ім'я, що запитується в DNS	Заголовок `Host`	Ціль запиту (request target)
1	www.s04.lab	www.s04.lab	/search/reports?limit=95&sort=x
2	docs.s04.lab	docs.s04.lab	/reports/?limit=%D0%BA%D0%B8%D1%97%D0%B2
3	edge.s04.lab	edge.s04.lab	/search
4	www.s04.lab	www.s04.lab:8443	/reports
5	s04.lab	-	-
Декодоване значення параметра URI 2:київ
Чому URI 5 не має складової host, хоча містить символ `@`:схема mailto не використовує ієрх. синтаксис тому уся адреса є непрозоорим шляхом
1.2. Перевірка (після завдання 3)
<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->
```text
> GET /search/reports?limit=95\&sort=x HTTP/1.1

> Host: www.s04.lab

> User-Agent: curl/8.5.0

> Accept: \*/\*

> 

server: web-a

Host: www.s04.lab

request URI: /search/reports?limit=95\&sort=x

path: /search/reports

query: limit=95\&sort=x

client: 10.53.1.1

> GET /reports/?limit=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1

> Host: docs.s04.lab

> User-Agent: curl/8.5.0

> Accept: \*/\*

> 

server: web-a

Host: docs.s04.lab

request URI: /reports/?limit=%D0%BA%D0%B8%D1%97%D0%B2

path: /reports/

query: limit=%D0%BA%D0%B8%D1%97%D0%B2

client: 10.53.1.1

> GET /search HTTP/1.1

> Host: edge.s04.lab

> Authorization: Basic dXNlcjpwYXNz

> User-Agent: curl/8.5.0

> Accept: \*/\*

> 

server: web-b

Host: edge.s04.lab

request URI: /search

path: /search

query: 

client: 10.53.1.1
```
Значення заголовка `Authorization` і результат декодування:
```text
> Authorization: Basic dXNlcjpwYXNz

echo 'dXNlcjpwYXNz' | base64 -d

user:pass
```
URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:
```text
curl 'http://www.s04.lab/search/../reports'

curl --path-as-is 'http://www.s04.lab/search/../reports'
```
Запит	`request URI`, отриманий сервером	`path`, отриманий сервером
звичайний	/reports	/reports
`--path-as-is`	/search/../reports	/reports
Розбіжності між URI та тим, що отримав сервер, і їх пояснення:
Фрагмент (#section-2, #top) клієнт ніколи не відправляє на сервер; userinfo (user:pass) не передається в рядку запиту чи заголовку Host, а перетворюється клієнтом у заголовок Authorization: Basic; відносна навігація Завдання 2. Проходження ієрархії вручну
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.s04.lab A +norec

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14260

;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: 883350b07db8a81a010000006aea1739d5b80ea0b38ea598 (good)

;; QUESTION SECTION:

;ns.s04.lab.                    IN      A



;; AUTHORITY SECTION:

lab.                    172800  IN      NS      ns1.nic.lab.



;; ADDITIONAL SECTION:

ns1.nic.lab.            172800  IN      A       10.53.0.3



;; Query time: 81 msec

;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)

;; WHEN: Tue Nov 03 17:17:45 UTC 2026

;; MSG SIZE  rcvd: 111





; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.3 ns.s04.lab A +norec

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20177

;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: 5fad459802b28524010000006aea1739259be7abda35588e (good)

;; QUESTION SECTION:

;ns.s04.lab.                    IN      A



;; AUTHORITY SECTION:

s04.lab.                3600    IN      NS      ns.s04.lab.



;; ADDITIONAL SECTION:

ns.s04.lab.             3600    IN      A       10.53.1.1



;; Query time: 41 msec

;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)

;; WHEN: Tue Nov 03 17:17:45 UTC 2026

;; MSG SIZE  rcvd: 97





; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.s04.lab A +norec

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49173

;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: 1ddf0411723df8c1010000006aea17399372fccfa5ad1a7b (good)

;; QUESTION SECTION:

;ns.s04.lab.                    IN      A



;; ANSWER SECTION:

ns.s04.lab.             90      IN      A       10.53.1.1



;; AUTHORITY SECTION:

s04.lab.                90      IN      NS      ns.s04.lab.



;; Query time: 0 msec

;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)

;; WHEN: Tue Nov 03 17:17:45 UTC 2026

;; MSG SIZE  rcvd: 97
```
Сервер	status	flags	ANSWER	AUTHORITY	ADDITIONAL
10.53.0.2	NOERROR	qr	0	1	2
10.53.0.3	NOERROR	qr	0	1	2
10.53.1.1	NOERROR	qr aa	1	1	1
Вивід `dig ns.s04.lab A +trace`:
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.s04.lab A +trace

;; global options: +cmd

.                       85922   IN      NS      a.root-servers.net.

;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms



lab.                    172800  IN      NS      ns1.nic.lab.

;; Received 111 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms



s04.lab.                3600    IN      NS      ns.s04.lab.

;; Received 97 bytes from 10.53.0.3#53(ns1.nic.lab) in 42 ms



ns.s04.lab.             90      IN      A       10.53.1.1

s04.lab.                90      IN      NS      ns.s04.lab.

;; Received 97 bytes from 10.53.1.1#53(ns.s04.lab) in 1 ms
```
Запит	TTL запису NS	Прапорець `aa`	Секція, у якій міститься NS
`dig @10.53.0.3 s04.lab NS +norec`	3600	ні	AUTHORITY
`dig @10.53.1.1 s04.lab NS +norec`	90	так	ANSWER
Які відповіді є відсиланнями і за якими ознаками це визначено?
Відповіді від серверів 10.53.0.2 та 10.53.0.3. Ознаки: статус NOERROR, відсутність прапорця Authoritative Answer, порожня секція ANSWER та наявність записів типу NS у секції AUTHORITY, які вказують на сервери нижчого рівня делегування.
Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?
Це glue-запис, який містить IP-адресу авторитативного сервера зони s04.lab. Оскільки ім'я сервера ns.s04.lab знаходиться всередині самої зони s04.lab, без glue-запису виникла б циклічна залежність, щоб дізнатися IP-адресу ns.s04.lab, резолверу довелося б звернутися до ns.s04.lab, чию адресу він ще не знає.
Яка з двох копій запису NS є авторитативною і чому?
Авторитативною є копія від власного сервера 10.53.1.1 (містить прапорець aa, запис повертається в секції ANSWER). Запис на TLD-сервері є лише делегуючим записом у батьківській зоні на точці розрізу
Завдання 3. Заповнення власної зони
Вивід `zone-reload`:
```text
zone s04.lab/IN: loaded serial 2026102702

OK

zone reload queued
```
<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->
Запис	status	Прапорець `aa`	Значення в ANSWER	TTL	Відповідає картці (так/ні)
`www` A	NOERROR	aa	10.53.0.80	90	так
псевдонім CNAME	NOERROR	аа	www.s04.lab	90	так
`mail` A	NOERROR	аа	10.53.0.80	90	так
`@` MX	NOERROR	аа	40 mail.s04.lab.	90	так
`@` TXT	NOERROR	аа	"lab-token=ad25406f"	90	так
другий хост A	NOERROR	аа	10.53.0.81	20	так
Порівняння відповідей резолвера та власного сервера:
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 docs.s04.lab A

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34865

;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

;; QUESTION SECTION:

;docs.s04.lab.                  IN      A



;; ANSWER SECTION:

docs.s04.lab.           90      IN      CNAME   www.s04.lab.

www.s04.lab.            90      IN      A       10.53.0.80



;; Query time: 41 msec

;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)

;; WHEN: Tue Nov 03 17:44:47 UTC 2026

;; MSG SIZE  rcvd: 75





; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 docs.s04.lab A +norec

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36776

;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: f1301cf1c42e9803010000006aea1d8f68925388c8627d33 (good)

;; QUESTION SECTION:

;docs.s04.lab.                  IN      A



;; ANSWER SECTION:

docs.s04.lab.           90      IN      CNAME   www.s04.lab.

www.s04.lab.            90      IN      A       10.53.0.80



;; AUTHORITY SECTION:

s04.lab.                90      IN      NS      ns.s04.lab.



;; ADDITIONAL SECTION:

ns.s04.lab.             90      IN      A       10.53.1.1



;; Query time: 1 msec

;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)

;; WHEN: Tue Nov 03 17:44:47 UTC 2026

;; MSG SIZE  rcvd: 136





; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 s04.lab MX

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24852

;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

;; QUESTION SECTION:

;s04.lab.                       IN      MX



;; ANSWER SECTION:

s04.lab.                90      IN      MX      40 mail.s04.lab.



;; Query time: 0 msec

;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)

;; WHEN: Tue Nov 03 17:44:47 UTC 2026

;; MSG SIZE  rcvd: 57





; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 s04.lab MX +norec

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25465

;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: 1487741f47dab6b6010000006aea1d8f749149b7c3469650 (good)

;; QUESTION SECTION:

;s04.lab.                       IN      MX



;; ANSWER SECTION:

s04.lab.                90      IN      MX      40 mail.s04.lab.



;; AUTHORITY SECTION:

s04.lab.                90      IN      NS      ns.s04.lab.



;; ADDITIONAL SECTION:

mail.s04.lab.           90      IN      A       10.53.0.80

ns.s04.lab.             90      IN      A       10.53.1.1



;; Query time: 0 msec

;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)

;; WHEN: Tue Nov 03 17:44:47 UTC 2026

;; MSG SIZE  rcvd: 134
```
Запит	flags	ANSWER (кількість, типи)	AUTHORITY (кількість, типи)	ADDITIONAL (кількість, типи)	TTL
псевдонім через resolver-a	qr rd ra	2 (CNAME, A)	0	1 (OPT)	<= 90
псевдонім через 10.53.1.1	qr aa	2 (CNAME, A)	1 (NS)	2 (A, OPT)	90
MX через resolver-a	qr rd ra	1 (MX)	0	1 (OPT)	<= 90
MX через 10.53.1.1	qr aa	1 (MX)	1 (NS)	2 (A OPT)	90
Пояснення відмінностей: Власний сервер 10.53.1.1 є авторитативним для зони s04.lab, тому повертає прапорець aa , а також додає секцію AUTHORITY із записом NS ns.s04.lab та секцію ADDITIONAL з A-записами для ns.s04.lab . Рекурсивний резолвер  відповідає з кешу, тому замість aa встановлює прапорці rd Recursion Desired і ra Recursion Available, а секції AUTHORITY та адресні записи в ADDITIONAL опускає. TTL у всіх відповідях становить 90 с, оскільки запити до резолвера були виконані одразу в момент кешування (до спливання першої секунди).
Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:
```text
dns\_master\_load: /lab/zone/s04.lab.zone:22: s04.lab: CNAME and other data

zone s04.lab/IN: loading from master file /lab/zone/s04.lab.zone failed: CNAME and other data

zone s04.lab/IN: not loaded due to errors.
```
Який TTL має кожен запис зони і звідки це значення взято?
Записи www, docs, mail, MX, TXT та NS мають TTL 90 с — це значення за замовчуванням із директиви $TTL 90 на початку файлу зони. Запис SOA має явно заданий TTL 15 с, а запис хоста edge — явно заданий у рядку запису TTL 20 с.
Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?
секція ANSWER містить 2 записи: сам запис CNAME (docs.s04.lab -> www.s04.lab) та цільовий запис A (www.s04.lab -> 10.53.0.80), оскільки резолвер автоматично розгортає ланцюжок CNAME до кінцевої адреси шуканого типу.
Чому запис CNAME неможливо розмістити для імені `@`?
За стандартом DNS, якщо ім'я має запис CNAME, воно не може мати жодних інших записів інших типів. Оскільки корінь зони @ обов'язково містить записи SOA та NS, додавання CNAME на @ викликає конфлікт типів



Завдання 4. TTL і видимість змін
Два запити з інтервалом 5 с:
```text
www.s04.lab.            90      IN      A       10.53.0.80

www.s04.lab.            84      IN      A       10.53.0.80
```
Прогноз (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:
Перевірка, що сервер до `zone-reload` повертає стару адресу:
```text
root@ns-s04:/# dig @10.53.1.1 www.s04.lab +norec +short

10.53.0.80
```
Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):
```text
18:08:38

www.s04.lab.            90      IN      A       10.53.0.81

zone s04.lab/IN: loaded serial 2026102703

OK

zone reload up-to-date

18:08:39

10.53.0.81
```
Очікуваний час появи нової адреси (підставити R у формулу):
18:08:38+90с=18:10:08
Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):
```text
18:08:39 10.53.0.81
```
Рядок `server:` з відповіді `curl`:
server: web-b
Час зміни	Очікуваний час	Спостережений час	Виміряна затримка, с	Прогноз, с
18:08:39	18:10:08	18:08:39	0	90
Чим пояснюється розбіжність між прогнозом і виміряним значенням?
У кеші резолвера попередня відповідь зі старою адресою або вже встигла вичерпати свій TTL на момент команди, або резолвер виконав новий запит уже після завантаження нової зони, тому клієнт отримав оновлений запис
Якою була б найбільша можлива затримка для цього запису і за яких умов?
Найбільша можлива затримка дорівнює значенню TTL запису, тобто 90 секунд. Вона виникає за умови, якщо рекурсивний резолвер звернувся до авторитативного сервера і закешував стару адресу за секунду до виконання команди
Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?
Заздалегідь зменшити значення TTL запису в зоні до невеликого часу, наприклад 60–300 секунд, і дочекатися поки старий кеш вичерпається в усіх резолверах інтернету. Після цього змінити IP-адресу на нову, перевірити працездатність сервісу і повернути стандартне високе значення TTL назад
Завдання 5. Негативне кешування
Прогноз (записати до вимірювання). Формула за RFC 2308 і розраховане значення:
TTL neg ​ =min(SOA TTL,SOA.MINIMUM)
min(15,20)=15 с.
Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-fvikr.s04.lab A +norec

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 45095

;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
```
Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):
```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 new-fvikr.s04.lab A

; (1 server found)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 1923

;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1



;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

;; QUESTION SECTION:

;new-fvikr.s04.lab.             IN      A



;; AUTHORITY SECTION:

s04.lab.                15      IN      SOA     ns.s04.lab. admin.s04.lab. 2026102703 3600 600 86400 20



;; Query time: 137 msec

;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)

;; WHEN: Tue Nov 03 18:19:38 UTC 2026

;; MSG SIZE  rcvd: 91



zone s04.lab/IN: loaded serial 2026102704

OK

zone reload queued

18:19:38

10.53.0.80
```
Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):
```text
18:19:38 status: NXDOMAIN

18:19:39 status: NXDOMAIN

18:19:53 status: NXDOMAIN

18:19:54 status: NOERROR
```
Негативну відповідь закешовано	Запис створено	Ім'я резолвиться через resolver-a	Виміряно, с	Прогноз, с
18:19:38	8:19:38	8:19:54	16	15
<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->
Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?
Тому що запитуваного запису в зоні не існує, отже, він не має власного TTL
Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.
На авторитативному сервері та в початковий момент кешування на резолвері він дорівнює мінімуму з TTL запису SOA у файлі зони 15 с та поля minimum 20 с, тобто 15 с. Далі у відповідях рекурсивного резолвера цей TTL у секції AUTHORITY зменшується щосекунди від 15 до 0 у міру старіння негативного кешу.
Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?
Перша перевірка до створення запису призведе до того, що рекурсивний резолвер закешує негативну відповідь NXDOMAIN на час негативного TTL 15 с. Тому одразу після створення запису скрипт продовжить отримувати NXDOMAIN із кешу резолвера, хоча запис на авторитативному сервері вже існує.
Завдання 6. Затримка пошуку імені
Прогноз (записати до вимірювання). До яких серверів звернеться resolver-a:
Ім'я	Перший запит	Повторний запит
A: нове ім'я у власній зоні	власний сервер (10.53.1.1)	з кешу(жодного)
B: нове ім'я в зоні `lab.`	TLD (10.53.0.3)	з кешу(жодного)
C: ім'я в неіснуючому TLD	root (10.53.0.2)	з кешу(жодного)
D: `www.sNN.lab` після перезапуску резолвера	root (10.53.0.2), TLD (10.53.0.3), власний сервер (10.53.1.1)	з кешу(жодного)
Вивід вимірювань:
```text
--- A (x32484.s04.lab) ---
;; Query time: 1 msec
;; Query time: 0 msec
--- B (x27838.lab) ---
;; Query time: 40 msec
;; Query time: 0 msec
--- C (x23795.zz23783) ---
;; Query time: 80 msec
;; Query time: 0 msec
--- D (www.s04.lab) ---
;; Query time: 401 msec
;; Query time: 0 msec
```
Ім'я	Перший запит, мс	Повторний, мс	Сервери, до яких звернувся резолвер (за журналами)
A	1	0	власний сервер (10.53.1.1)
B	40	0	TLD (10.53.0.3)
C	80	0	root (10.53.0.2)
D	401	0	root (10.53.0.2), TLD (10.53.0.3), власний сервер (10.53.1.1)
Значення `time\\\_namelookup` з `curl` після перезапуску резолвера:404ms
Фрагменти журналів запитів root, TLD і власного сервера для рядка D: 
```text
=== ROOT LOG ===
04-Nov-2026 09:31:25.180 client @0x7b746c001f38 10.53.0.53#38440 (www.s04.lab): query: www.s04.lab IN A -E(0)DC (10.53.0.2)
04-Nov-2026 09:31:31.580 client @0x7b746406ac88 10.53.0.53#48451 (.): query: . IN NS -E(0)DC (10.53.0.2)
04-Nov-2026 09:31:31.660 client @0x7b746406ac88 10.53.0.53#33498 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
04-Nov-2026 09:31:31.740 client @0x7b746c001f38 10.53.0.53#44989 (www.s04.lab): query: www.s04.lab IN AAAA -E(0)DC (10.53.0.2)
04-Nov-2026 09:31:31.740 client @0x7b746406ac88 10.53.0.53#54772 (www.s04.lab): query: www.s04.lab IN A -E(0)DC (10.53.0.2)
=== TLD LOG ===
04-Nov-2026 09:30:15.106 client @0x70af58006078 10.53.0.53#60499 (x27838.lab): query: x27838.lab IN A -E(0)DC (10.53.0.3)
04-Nov-2026 09:31:25.301 client @0x70af58006078 10.53.0.53#51118 (www.s04.lab): query: www.s04.lab IN A -E(0)DC (10.53.0.3)
04-Nov-2026 09:31:31.861 client @0x70af58006078 10.53.0.53#56289 (www.s04.lab): query: www.s04.lab IN AAAA -E(0)DC (10.53.0.3)
04-Nov-2026 09:31:31.861 client @0x70af6006ac88 10.53.0.53#37565 (www.s04.lab): query: www.s04.lab IN A -E(0)DC (10.53.0.3)
=== S04 LOG ===
04-Nov-2026 09:30:15.085 client @0x789af806ac88 10.53.0.53#47015 (x32484.s04.lab): query: x32484.s04.lab IN A -E(0)DC (10.53.1.1)
04-Nov-2026 09:31:25.341 client @0x789af0005278 10.53.0.53#60503 (www.s04.lab): query: www.s04.lab IN A -E(0)DC (10.53.1.1)
04-Nov-2026 09:31:31.901 client @0x789af0005278 10.53.0.53#15464 (www.s04.lab): query: www.s04.lab IN A -E(0)DC (10.53.1.1)
04-Nov-2026 09:31:31.901 client @0x789af0008ca8 10.53.0.53#12392 (www.s04.lab): query: www.s04.lab IN AAAA -E(0)DC (10.53.1.1)
```
Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.
Із рядка C (запит до неіснуючого TLD, де відповідає лише root) випливає, що затримка root 10.53.0.2 становить 80 мс. Із рядка B (запит у зоні lab., де lab. уже закешовано, тому запит іде лише до TLD) випливає затримка TLD 10.53.0.3 — 40 мс. Із рядка A (запит до власної зони) випливає затримка власного сервера 10.53.1.1 — 1 мс. У рядку D після перезапуску резолвера час склав 401 мс, оскільки resolver-a окрім ітеративного ланцюжка також виконав початкову ініціалізацію кореневих підказок
Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?
Тому що делегування зони s04.lab. вже перебуває в кеші resolver-a після попередніх завдань. Резолвер одразу звертається до найближчого відомого авторитативного сервера 10.53.1.1.   
Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?
При холодному кеші клієнту потрібно розв'язати всі 5 доменних імен. Якщо вони в різних зонах, кожне ім'я вимагатиме проходження ієрархії DNS, що затримає початок завантаження ресурсів
Завдання 7. Інцидент: зупинка власного сервера
Прогноз (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:resolver-a (10.53.0.53): TTL запису edge дорівнює 20 с. Після зупинки сервера та спливання TTL резолвер спробує звернутися до 10.53.1.1, зазнає невдачі і поверне помилку SERVFAIL або завершиться за таймаутом (~30 с при +timeout=30). Повторний запит поверне SERVFAIL майже миттєво через кэшування сбою.
resolver-b (10.53.0.54): Працює в режимі serve-stale (RFC 8767). Після невдалої спроби зв'язку з 10.53.1.1 він поверне застарілий запис edge.s04.lab. IN A 10.53.0.81 з кэшу з фіксованим TTL (30 с) за час, необхідний для перевірки недоступності авторитативного сервера (~1–2 с).

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):
```text
edge.s04.lab.           20      IN      A       10.53.0.81
edge.s04.lab.           20      IN      A       10.53.0.81
09:45:13 root@ns-s04:/# named-stop
```
Запити із заміром часу (крок 5):
```text
root@ns-s04:/# time dig @10.53.0.53 edge.s04.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 edge.s04.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.035s
user    0m0.008s
sys     0m0.003s

root@ns-s04:/# time dig @10.53.0.53 edge.s04.lab +tries=1 +timeout=30

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 edge.s04.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 29290
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;edge.s04.lab.                  IN      A

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Wed Nov 04 09:46:34 UTC 2026
;; MSG SIZE  rcvd: 41

real    0m0.021s
user    0m0.005s
sys     0m0.006s

root@ns-s04:/# time dig @10.53.0.54 edge.s04.lab +tries=1 +timeout=30

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 edge.s04.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6836
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;edge.s04.lab.                  IN      A

;; ANSWER SECTION:
edge.s04.lab.           30      IN      A       10.53.0.81

;; Query time: 1799 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Wed Nov 04 09:46:40 UTC 2026
;; MSG SIZE  rcvd: 57

real    0m1.824s
user    0m0.006s
sys     0m0.007s
```
Запит	status або тайм-аут	ANSWER, TTL	`real`
resolver-a, перший	timed out	0	0m30.035s
resolver-a, другий	SERVFAIL	0	0m0.021s
resolver-b	NOERROR	edge.s04.lab. 30 IN A 10.53.0.81,	0m1.824s
Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):
```text
09:46:45
09:46:45
09:46:47
09:46:49
09:46:51
09:46:53
09:46:55
09:46:57
09:46:59
09:47:02
09:47:04 10.53.0.81
```
Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):
```text
ns.s04.lab. admin.s04.lab. 2026102704 3600 600 86400 20
```
Постмортем
Що сталося: Зупинився єдиний авторитативний DNS-сервер зони s04.lab (10.53.1.1).
Вплив на клієнтів resolver-a і resolver-b: Клієнти resolver-a повністю втратили можливість резолвити ім'я edge.s04.lab (отримували таймаут 30 с і помилки SERVFAIL). Клієнти resolver-b продовжували успішно отримувати адреси завдяки механізму serve-stale (RFC 8767).
Хронологія з виміряним часом:
09:45:13 — Зупинка сервера named (named-stop).

09:45:38 — Вичерпано TTL запису (20 с) у кэші резолверів.

09:46:04 — Перший запит до resolver-a завершився таймаутом через 30.035 с.

09:46:34 — Другий запит до resolver-a за 0.021 с повернув SERVFAIL.

09:46:40 — Запит до resolver-b повертає stale-запис за 1.824 с.

09:46:45 — Запуск сервера named (named-start).

09:47:04 — resolver-a відновив видачу правильних відповідей.
Чому кеш допоміг або не допоміг: Звичайний кэш resolver-a не допоміг, оскільки після спливання TTL (20 с) запис вважається недійсним і видаляється. Кэш resolver-b допоміг завдяки підтриманню режиму serve-stale, який дозволяє використовувати прострочені дані з кэшу в разі недоступності авторитативного сервера.
Що слід змінити:Розгорнути щонайменше два незалежних авторитативних сервери (Primary та Secondary) і увімкнути режим serve-stale на рекурсивних резолверах мережі
Який TTL мав запис у відповіді resolver-b і навіщо такий TTL? Запис мав TTL 30 секунд. Це короткий фіксований TTL

Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.
Перший запит тривав 30.035 с через очікування таймауту повторних спроб зв'язку з недоступним сервером 10.53.1.1. Другий запит завершився за 0.021 с з помилкою SERVFAIL, оскільки резолвер закешував негативний результат спроби повторного резолвінгу (SERVFAIL cache) і не витрачав час на нові мережеві запити
Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?
resolver-a зарезолвив ім'я лише через 19 секунд (о 09:47:04), оскільки протягом цього часу діяв негативний кэш сбою (SERVFAIL cache), який заборгував повторні звернення до сервера до вичерпання свого внутрішнього таймера.
У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.
Serve-stale шкодить під час планової зміни IP-адреси сервісу (як у Завданні 4). Якщо авторитативний сервер виходить з ладу одразу після зміни адреси в зоні, резолвер з serve-stale віддаватиме клієнтам стару невалідна адресу з кэшу замість повідомлення про помилку, що призведе до спроб клієнтів підключитися до непрацюючого вузла.
Фінальний файл зони
<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->
```text
; Zone s04.lab — starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 90
@     15 IN SOA ns.s04.lab. admin.s04.lab. (
            2026102704 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            20         ; minimum
        )
@     IN NS ns.s04.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www     IN  A       10.53.0.81
docs    IN  CNAME   www
mail    IN  A       10.53.0.80
@       IN  MX      40 mail
@       IN  TXT     "lab-token=ad25406f"
edge    20  IN  A   10.53.0.81

new-fvikr IN A 10.53.0.80
```
Висновки
Що виявилося несподіваним. Результат, який не збігся з очікуванням, і рядок виводу, де це видно:
Яке питання залишилося без відповіді. Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:
Декларація використання ШІ
Використовувалися інструменти ШІ: <!-- так / ні --> так
<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->
№	Інструмент	Промпт (дослівно)	Що використано у звіті і що змінено самостійно
1	Gemini	Відповіси на завдання у 5,6	Дослідив відповіді та записав їх своїми словами
2	Gemini	напиши прогнози до 7 6 5 4	Записано прогнози
