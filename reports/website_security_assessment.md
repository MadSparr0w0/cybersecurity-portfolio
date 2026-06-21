# Sanitized Security Assessment Report — Веб сайт организации

**Объект проверки:** `[REDACTED_WEBSITE]`  
**IP-адрес:** `[REDACTED_IP]`  
**Формат проверки:** внешний black-box security assessment  

> **Sanitized version for portfolio.** Из отчёта удалены или заменены домен, IP-адрес, название организации, внешние домены, точные версии, отдельные внутренние endpoints/actions и другие детали, которые могли бы идентифицировать объект проверки или облегчить эксплуатацию. Документ оставлен как демонстрация формата работы: scope → methodology → findings → impact → remediation.

---

## 1. Краткое резюме

В ходе внешней проверки сайта `<SITE>` были выявлены несколько значимых проблем безопасности. Наиболее критичным риском является публичная доступность инфраструктурных сервисов на выделенном сервере сайта, включая FTP, SSH/SFTP и MariaDB/MySQL. Также выявлены устаревшие компоненты WordPress/PHP, раскрытие административного административный username `<REDACTED_USERNAME>`, слабые атрибуты cookies, неполная конфигурация HTTP security headers и публично доступные unauthenticated AJAX actions, связанные с функциональностью записи на приём.

Проверка выполнялась без подбора паролей, без попыток входа в FTP/SSH/MySQL, без изменения данных и без создания записей в продуктивной системе. State-changing action `<REDACTED_STATE_CHANGING_ACTION>` был обнаружен в клиентском коде, но не тестировался без отдельного письменного разрешения, чтобы не создавать запись в системе.

---

## 2. Scope и ограничения

### 2.1 Scope

В рамках проверки рассматривались:

- основной домен `https://<SITE>/`;
- публичные страницы сайта;
- публичные HTTP endpoints;
- WordPress REST API;
- клиентский JavaScript;
- HTTP headers;
- sitemap/robots;
- публично доступные сервисы на IP-адресе, связанном с сайтом.

### 2.2 Ограничения проверки

Проверка выполнялась в режиме внешнего black-box assessment без административного доступа к серверу, WordPress, базе данных и внутренней инфраструктуре.

Не выполнялись:

-  brute force / password spraying / подбор паролей;
- эксплуатация уязвимостей с изменением данных;
- DoS/DDoS и нагрузочное тестирование;
-  попытки входа в FTP/SSH/SFTP/MySQL без выданных тестовых учётных данных;
- доступ к персональным данным;
-  изменение, создание или удаление данных в продуктивной системе;

---

## 3. Methodology

Проверка включала:

- анализ HTTP headers и Set-Cookie;
- анализ HTML/JavaScript;
- проверку HTTP → HTTPS redirect chains;
- проверку WordPress REST API и стандартных WordPress endpoints;
- проверку публичных AJAX actions через `admin-ajax.php`;
- проверку sitemap/robots и устаревших ссылок;
- внешнюю инвентаризацию TCP-сервисов;
- безопасные проверки FTP/SSH/MySQL без массовой аутентификации;
- фиксацию positive/negative checks для неподтверждённых уязвимостей.

---

## 4. Сводная таблица findings

| ID | Finding | Severity | Status |
|---|---|---:|---|
| F01 | Раскрытие устаревшего технологического стека | Medium | Confirmed |
| F02 | Недостаточные security attributes у session cookies | Medium | Confirmed |
| F03 | Неполная конфигурация HTTP security headers | Medium | Confirmed |
| F04 | Публичное раскрытие WordPress REST API и маршрутов | Low | Confirmed |
| F05 | Публичная доступность FTP/SSH/SFTP/MySQL на выделенном сервере | High | Confirmed |
| F06 | Sitemap раскрывает большую карту устаревших страниц и HTTP-ссылки | Low | Confirmed |
| F07 | Использование небезопасных HTTP-ссылок на HTTPS-сайте | Low / Medium | Confirmed |
| F08 | Неработающие ссылки в разделе для абитуриентов | Low / Informational | Confirmed |
| F09 | Публично доступные unauthenticated AJAX actions плагина записи на приём | Medium | Confirmed, limited impact confirmed |
| F10 | WordPress user enumeration через author archive | Medium | Confirmed |
| F11 | Публичная доступность служебных WordPress endpoints и файлов установки | Low / Medium | Confirmed |
| F12 | Reflective CORS policy with credentials для WordPress REST API | Low / Medium | Confirmed |

---

# 5. Findings

---

## F01 — Раскрытие устаревшего технологического стека

**Severity:** Medium  
**Status:** Confirmed

### Description

В HTTP-ответах и HTML-коде сайта раскрываются сведения о технологическом стеке, включая веб-сервер, версию PHP, WordPress и отдельные компоненты/плагины. Также присутствуют старые JavaScript-библиотеки.

### Evidence

В ответах и HTML-коде зафиксированы следующие признаки:

- `Server: Apache`;
- `X-Powered-By: PHP/[REDACTED_VERSION]`;
- `meta name="generator" content="WordPress [REDACTED_VERSION]"`;
- WordPress assets с `ver=[REDACTED_VERSION]`;
- `contact-form-7` version `[REDACTED_VERSION]`;
- `jquery.fancybox` version `[REDACTED_VERSION]`;
- `wp-pagenavi` version `[REDACTED_VERSION]`;
- `NextGEN version [REDACTED_VERSION]`;
- `jquery [REDACTED_VERSION]`, `jquery [REDACTED_VERSION]`, `jquery-migrate [REDACTED_VERSION]`;
- комментарии W3 Total Cache с упоминанием memcache.

Также на страницах подключается устаревшая версия jQuery:

```text
https://ajax.googleapis.com/ajax/libs/jquery/[REDACTED_VERSION]/jquery.min.js
```

### Impact

Раскрытие точных версий упрощает подбор известных уязвимостей под конкретные версии PHP, WordPress core, плагинов и JavaScript-библиотек. Использование устаревших компонентов повышает вероятность наличия известных CVE и несовместимостей с современными мерами защиты.

### Remediation

- Обновить PHP до поддерживаемой версии.
- Обновить WordPress core, тему и плагины.
- Удалить или обновить устаревшие JS-библиотеки.
- Отключить раскрытие `X-Powered-By`.
- Минимизировать публичное раскрытие версий компонентов.

---

## F02 — Недостаточные security attributes у session cookies

**Severity:** Medium  
**Status:** Confirmed

### Description

Сайт устанавливает session cookies без явно заданных атрибутов `Secure`, `HttpOnly` и `SameSite`.

### Evidence

В ответах сервера наблюдались cookies вида:

```http
Set-Cookie: _wp_session=[REDACTED]; expires=...; Max-Age=...; path=/
Set-Cookie: PHPSESSID=[REDACTED]; path=/
```

В указанных `Set-Cookie` заголовках не наблюдались атрибуты:

- `Secure`;
- `HttpOnly`;
- `SameSite`.

Дополнительно cookie устанавливается даже на HTTP redirect response:

```http
GET http://<SITE>/

HTTP/1.1 301 Moved Permanently
X-Powered-By: PHP/[REDACTED_VERSION]
Set-Cookie: _wp_session=[REDACTED]; expires=...; Max-Age=1800; path=/
Location: https://<SITE>/
```

Аналогичное поведение наблюдалось на `admin-ajax.php`:

```http
Set-Cookie: _wp_session=[REDACTED]; expires=...; Max-Age=1800; path=/
```

### Impact

Отсутствие `HttpOnly` повышает риск кражи cookies при XSS. Отсутствие `Secure` снижает защиту при ошибках HTTPS/HTTP-конфигурации и особенно проблемно, когда cookie устанавливается в HTTP redirect response. Отсутствие `SameSite` ослабляет защиту от некоторых межсайтовых сценариев.

### Remediation

- Установить для session cookies атрибуты `Secure`, `HttpOnly`, `SameSite=Lax` или `SameSite=Strict`.
- Проверить настройки PHP session, WordPress и плагинов, которые создают cookies.
- Убедиться, что все cookies передаются только по HTTPS.
- Не устанавливать session cookies на HTTP-ответах до перенаправления на HTTPS.

---

## F03 — Неполная конфигурация HTTP security headers

**Severity:** Medium  
**Status:** Confirmed

### Description

В проверенных HTTP-ответах отсутствует часть важных browser security headers. При этом сайт раскрывает `X-Powered-By: PHP/[REDACTED_VERSION]`. Заголовки применяются непоследовательно: на части endpoints присутствуют X-Frame-Options и X-Content-Type-Options, но на части публичных страниц они отсутствуют.

### Evidence

В проверенных ответах не наблюдались следующие заголовки:

- `Content-Security-Policy`;
- `Strict-Transport-Security`;
- `Permissions-Policy`;
- `X-Frame-Options` на части публичных страниц;
- `X-Content-Type-Options` на части публичных страниц.

Присутствовал заголовок:

```http
X-Powered-By: PHP/[REDACTED_VERSION]
```

На `admin-ajax.php` часть headers присутствовала:

```http
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
```

Это указывает не на полное отсутствие, а на неединообразную конфигурацию заголовков.

### Impact

Отсутствие CSP снижает устойчивость к XSS. Отсутствие HSTS ослабляет защиту от downgrade/HTTP-сценариев. Неполный и непоследовательный набор security headers ухудшает защиту пользователей в браузере.

### Remediation

- Добавить `Content-Security-Policy` после тестирования совместимости.
- Добавить `Strict-Transport-Security` после проверки корректности HTTPS на всех ресурсах.
- Добавить `Permissions-Policy`.
- Единообразно включить `X-Frame-Options` или CSP `frame-ancestors`.
- Единообразно включить `X-Content-Type-Options: nosniff`.
- Отключить `X-Powered-By`.

---

## F04 — Публичное раскрытие WordPress REST API и маршрутов

**Severity:** Low  
**Status:** Confirmed

### Description

Endpoint `/wp-json/` публично доступен и раскрывает информацию о WordPress REST API, namespaces и маршрутах.

### Evidence

В `/wp-json/` были доступны сведения о namespaces и routes, включая:

- `wp/v2`;
- `oembed/1.0`;
- `contact-form-7/v1`;
- маршруты для posts, pages, media и contact forms;
- методы `GET`, `POST`, `PUT`, `PATCH`, `DELETE` в schema для отдельных маршрутов.

При этом endpoint `/wp-json/wp/v2/users` был ограничен и возвращал `401`, что является положительным контролем.

### Impact

Само по себе наличие REST API является нормальным поведением WordPress. Однако раскрытие маршрутов и названий помогает атакующему быстрее составить карту сайта и искать уязвимые плагины/endpoints.

### Remediation

- Ограничить ненужные REST routes.
- Проверить, что чувствительные endpoints требуют авторизации.
- Оставить ограничение `/wp-json/wp/v2/users`.
- Убедиться, что state-changing REST actions защищены nonce/permission checks.

---

## F05 — Публичная доступность FTP/SSH/SFTP/MySQL на выделенном сервере сайта

**Severity:** High  
**Status:** Confirmed  
**Confidence:** High

### Description

На IP-адресе `<IP>`, обслуживающем сайт `<SITE>`, публично доступны административные и database-сервисы. 

### Evidence

Внешняя инвентаризация показала следующие открытые сервисы:

```text
21/tcp   open  ftp        [REDACTED_FTP_SERVICE]
22/tcp   open  ssh        [REDACTED_SSH_SERVICE]
80/tcp   open  http       [REDACTED_HTTP_SERVICE]
443/tcp  open  ssl/https
2222/tcp open  ssh        SFTP service
3306/tcp open  mysql      [REDACTED_DATABASE_SERVICE]
```

MariaDB/MySQL раскрывает информацию о протоколах, версии и плагин аутентификации `mysql_native_password`.

```text
3306/tcp open  mysql   [REDACTED_DATABASE_SERVICE]
Protocol: 10
Version: [REDACTED_DB_VERSION]
Auth Plugin Name: mysql_native_password
```

Дополнительно были выполнены безопасные проверки конфигурации.

SSH на `22/tcp` разрешает password authentication:

```text
22/tcp open ssh
Supported authentication methods:
  publickey
  password
```

SSH-сервис поддерживает legacy алгоритмы, включая SHA-1 на  KEX/MAC, `ssh-rsa` и режимы шифрования CBC:

```text
diffie-hellman-group-exchange-sha1
diffie-hellman-group14-sha1
ssh-rsa
aes256-cbc
aes128-cbc
hmac-sha1
```

Попытки аутентификации, подбор паролей и изменение данных не выполнялись.

### Impact

Публичная доступность FTP/SSH/SFTP/MySQL существенно увеличивает внешнюю поверхность атаки выделенного сервера. Злоумышленник может проводить атаки на учётные данные, использовать известные уязвимости сервисов, пытаться получить административный доступ к серверу или прямой доступ к базе данных. Наиболее критичным является внешний доступ к MariaDB/MySQL.

Риск усиливается тем, что SSH допускает аутентификацию по паролю, а WordPress раскрывает имя административного пользователя `<REDACTED_USERNAME>`. Сами административные и database-сервисы не должны быть доступны из внешней сети без необходимости.

### Remediation

- Закрыть внешний доступ к `3306/tcp`, если он не требуется.
- Ограничить FTP/SSH/SFTP/MySQL через firewall, VPN или IP allowlist.
- Для SSH/SFTP использовать ключевую аутентификацию и отключить парольный вход.
- Для FTP рассмотреть отключение или замену на более безопасный административный канал.
- Отключить legacy SSH algorithms: SHA-1 KEX/MAC, CBC ciphers, `ssh-rsa`, если нет legacy-клиентов.
- Включить rate limiting и мониторинг попыток подключения.
- Проверить журналы FTP/SSH/MySQL на предмет подозрительной активности.

---

## F06 — Sitemap раскрывает большую карту устаревших публичных страниц и использует HTTP-ссылки

**Severity:** Low  
**Status:** Confirmed

### Description

Файл `/sitemap.xml` публично доступен, содержит большое количество URL и использует `http://` ссылки для части страниц и ресурсов.

### Evidence

В sitemap обнаружены:

- большой список публичных URL;
- устаревшие/legacy страницы;
- ссылки вида `http://<SITE>/...`;
- ссылка на stylesheet sitemap через `http://<SITE>/wp-content/plugins/companion-sitemap-generator/sitemap.xsl`.

### Impact

Sitemap облегчает инвентаризацию старых страниц и потенциально забытых разделов сайта. Наличие HTTP-ссылок указывает на неполную HTTPS-гигиену.

### Remediation

- Обновить sitemap.
- Удалить неактуальные/устаревшие URL.
- Заменить `http://` ссылки на `https://`.
- Настроить корректную генерацию sitemap через актуальный плагин.

---

## F07 — Использование небезопасных HTTP-ссылок на HTTPS-сайте

**Severity:** Low / Medium  
**Status:** Confirmed

### Description

На страницах, доступных по HTTPS, обнаружены абсолютные ссылки и ресурсы с использованием `http://`, а также обнаружена неоптимальная redirect-chain для `www` host.

### Evidence

В HTML-коде встречались ссылки вида:

```html
<img src="http://<SITE>/wp-content/uploads/...">
<a href="http://<SITE>/...">
<a href="http://<EXTERNAL_DOMAIN_1>/...">
<a href="http://<EXTERNAL_DOMAIN_2>/...">
```

Также HTTP-ссылки использовались в cookie/banner text и внутренних навигационных элементах.

Redirect-chain:

```http
GET http://<SITE>/
HTTP/1.1 301 Moved Permanently
Location: https://<SITE>/
Set-Cookie: _wp_session=[REDACTED]; expires=...; Max-Age=1800; path=/
```

```http
GET http://www.<SITE>/
HTTP/1.1 301 Moved Permanently
Location: http://<SITE>/
Set-Cookie: _wp_session=[REDACTED]; expires=...; Max-Age=1800; path=/
```

```http
GET https://www.<SITE>/
HTTP/2 301
Location: https://<SITE>/
Set-Cookie: _wp_session=[REDACTED]; expires=...; Max-Age=1800; path=/
```

Оптимальный вариант: `http://www.<SITE>/` должен перенаправлять сразу на `https://<SITE>/`, без промежуточного `http://<SITE>/`.

### Impact

HTTP-ссылки на HTTPS-сайте могут приводить к downgrade-переходам, mixed content предупреждениям, неконсистентному поведению браузера и снижению доверия к сайту. Установка cookie на HTTP redirect response дополнительно ухудшает HTTPS/cookie гигиену.

### Remediation

- Заменить внутренние абсолютные `http://` ссылки на `https://` или относительные URL.
- Исправить redirect-chain для `www` host
- Проверить шаблон WordPress и hardcoded links.
- Настроить HTTP → HTTPS redirect.
- После проверки добавить HSTS.

---

## F08 — Неработающие ссылки в разделе для абитуриентов

**Severity:** Low / Informational  
**Status:** Confirmed

### Description

В разделе для абитуриентов и FAQ обнаружены ссылки, ведущие на несуществующие страницы.

### Evidence

Примеры:

- ссылка на `/<REDACTED_OLD_PAGE>/` возвращает `404 Not Found`;
- ссылка из раздела `/<REDACTED_SECTION>/` на `/<REDACTED_BROKEN_PAGE>/` ведёт на несуществующую страницу;
- рабочая страница с приказами доступна по другому адресу: `/<REDACTED_VALID_PAGE>/`.

### Impact

Пользователи, включая абитуриентов, могут не находить важные материалы и формы. Это ухудшает пользовательский путь и доступность информации.

### Remediation

- Исправить устаревшие ссылки.
- Настроить перенаправление со старых адресов на актуальные.
- Регулярно проверять сайт на некорректные ссылки.

---

## F09 — Публично доступные unauthenticated AJAX actions плагина записи на приём

**Severity:** Medium  
**Status:** Confirmed, limited impact confirmed

### Description

В клиентском JavaScript плагина `<REDACTED_PLUGIN>` обнаружены AJAX actions, связанные с функциональностью записи на приём:

- `<REDACTED_READ_ACTION>`;
- `<REDACTED_CHECK_ACTION>`;
- `<REDACTED_STATE_CHANGING_ACTION>`.

Проверка показала, что actions доступны через `/wp-admin/admin-ajax.php` без авторизации. Для `<REDACTED_READ_ACTION>` и `<REDACTED_CHECK_ACTION>` подтверждено получение ответов бизнес-логики. Для state-changing action `<REDACTED_STATE_CHANGING_ACTION>` подтверждена публичная достижимость обработчика без авторизации и без явного nonce/permission отказа.

Полноценная отправка валидных данных в `<REDACTED_STATE_CHANGING_ACTION>` не выполнялась, чтобы избежать создания реальной записи или изменения данных приложения.

### Evidence

Пример безопасных read/check запросов:

```http
POST /wp-admin/admin-ajax.php
Content-Type: application/x-www-form-urlencoded

action=<REDACTED_READ_ACTION>&datetime=<REDACTED_DATE_TIME>```

Ответ:

```text
HTTP/2 200 OK
[]
```

```http
POST /wp-admin/admin-ajax.php
Content-Type: application/x-www-form-urlencoded

action=<REDACTED_CHECK_ACTION>&datetime=<REDACTED_DATE_TIME>
```

Ответ:

```text
HTTP/2 200 OK
success_check
```

Проверка `<REDACTED_STATE_CHANGING_ACTION>` без авторизации и без параметров:

```http
POST /wp-admin/admin-ajax.php
Content-Type: application/x-www-form-urlencoded

action=<REDACTED_STATE_CHANGING_ACTION>
```

Ответ:

```http
HTTP/2 200 OK
X-Robots-Tag: noindex
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Set-Cookie: _wp_session=[REDACTED]; expires=...; Max-Age=1800; path=/

error_add
```

Дополнительно было выполнено сравнение с несуществующим AJAX action.

Несуществующий action:

```http
POST /wp-admin/admin-ajax.php
Content-Type: application/x-www-form-urlencoded

action=<NON_EXISTENT_ACTION>
```

Ответ:

```http
HTTP/2 400

0
```

Реальный action `<REDACTED_STATE_CHANGING_ACTION>` через GET:

```http
GET /wp-admin/admin-ajax.php?action=<REDACTED_STATE_CHANGING_ACTION>
```

Ответ:

```http
HTTP/2 200

error_add
```

Различие ответов подтверждает, что `<REDACTED_STATE_CHANGING_ACTION>` зарегистрирован на сервере и обрабатывается логикой плагина без предварительной авторизации. Успешное создание записи в рамках проверки не выполнялось.

### Impact

Неавторизованный пользователь может обращаться к внутренней бизнес-логике записи на приём. Для read/check actions это позволяет получать или проверять сведения о доступности временных слотов. Для `<REDACTED_STATE_CHANGING_ACTION>` подтверждена достижимость state-changing обработчика; успешное создание записи в рамках проверки не выполнялось.

При отсутствии серверных nonce/CSRF проверок, проверок прав, ограничений частоты отправки и строгой валидации это может привести к спаму заявками, несанкционированному созданию записей или нарушению расписания приёмной комиссии.

### Remediation

- Добавить серверную проверку nonce/CSRF-token для всех AJAX действий.
- Для `<REDACTED_STATE_CHANGING_ACTION>` обязательно проверять права, nonce и корректность источника запроса.
- Запретить редактирование записей через GET.
- Добавить ограничение по частоте отправки  на `/wp-admin/admin-ajax.php`.
- Валидировать все входные параметры на сервере.
- Не полагаться на клиентскую JavaScript-валидацию.
- Логировать массовые или аномальные обращения к actions записи на приём.
- Проверить возможность эксплуатации `<REDACTED_STATE_CHANGING_ACTION>` на постановочные-копии, а не на действительные записи.

---

## F10 — WordPress user enumeration через author archive

**Severity:** Medium  
**Status:** Confirmed

### Description

Сайт раскрывает имя пользователя WordPress через стандартный механизм author archive. Запрос к `/?author=1` перенаправляет пользователя на `/author/<REDACTED_USERNAME>/`, что раскрывает существование учётной записи с административный username `<REDACTED_USERNAME>`.

### Evidence

Запрос:

```http
GET /?author=1
```

Ответ:

```http
HTTP/2 301
Location: https://<SITE>/author/<REDACTED_USERNAME>/
X-Redirect-By: WordPress
```

После перехода страница содержит признаки author archive:

```html
<title>[REDACTED_AUTHOR_ARCHIVE_TITLE]</title>
<body class="archive author author-<REDACTED_USERNAME> author-1 sidebars">
```

Запросы `/?author=2` и `/?author=3` возвращали 404.

### Impact

Раскрытие username облегчает целенаправленные атаки на учетные данные, перебор паролей и попытки входа в административные интерфейсы. Риск усиливается тем, что на выделенном сервере также публично доступны SSH/FTP/SFTP/MySQL.

### Remediation

- Отключить author enumeration.
- Запретить редиректы `/?author=<id>` на публичные username.
- Скрыть author archive для технических и административных аккаунтов.
- Переименовать административную учётную запись `<REDACTED_USERNAME>`.
- Использовать MFA для административного доступа.

---

## F11 — Публичная доступность служебных WordPress endpoints и файлов установки

**Severity:** Low / Medium  
**Status:** Confirmed

### Description

На сайте публично доступны стандартные WordPress-файлы и служебные endpoints, раскрывающие информацию о WordPress-инсталляции и её состоянии.

### Evidence

Обнаружено:

```text
/readme.html               200 OK
/license.txt               200 OK
/wp-config-sample.php      500 Database Error
/wp-admin/install.php      200 OK, “Уже установлен”
/wp-admin/upgrade.php      200 OK, “Обновление не требуется”
/wp-admin/setup-config.php 500, сообщает что wp-config.php уже существует
```

`/readme.html` содержит стандартную документацию WordPress и ссылки на install/upgrade пути. `/wp-admin/install.php` подтверждает, что WordPress уже установлен. `/wp-admin/upgrade.php` сообщает, что обновление базы данных не требуется.

### Impact

Данные endpoints не дают доступа сами по себе, но раскрывают внутреннее состояние WordPress-инсталляции и помогают атакующему строить карту системы. В сочетании с устаревшим стеком, раскрытым административный username `<REDACTED_USERNAME>` и открытыми инфраструктурными сервисами это повышает общий риск.

### Remediation

- Ограничить доступ к `/readme.html`, `/license.txt`, `/wp-admin/install.php`, `/wp-admin/upgrade.php`, `/wp-admin/setup-config.php`.
- Удалить ненужные стандартные файлы после установки.
- Проверить права доступа и web server rules.
- Убедиться, что installation/setup endpoints недоступны извне.

---

## F12 — Reflective CORS policy with credentials для WordPress REST API

**Severity:** Low / Medium  
**Status:** Confirmed

### Description

WordPress REST API отражает произвольный `Origin` в `Access-Control-Allow-Origin` и одновременно разрешает credentials через `Access-Control-Allow-Credentials: true`.

### Evidence

Запрос с произвольным Origin:

```http
GET /wp-json/
Origin: https://evil.example
```

Ответ:

```http
HTTP/2 200 OK
Access-Control-Allow-Origin: https://evil.example
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: OPTIONS, GET, POST, PUT, PATCH, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Vary: Origin
```

### Impact

Такая CORS-конфигурация может позволить стороннему сайту выполнять cross-origin REST API запросы от имени пользователя, если браузер отправляет cookies и если соответствующие REST endpoints недостаточно защищены nonce/permission проверок. Само по себе это не подтверждает обход авторизации, но увеличивает риск при наличии уязвимых REST путей или плагинов.

### Remediation

- Ограничить CORS только доверенными origin.
- Не использовать `Access-Control-Allow-Credentials: true` для произвольных origin.
- Ограничить разрешенные методы до реально необходимых.
- Проверить, что state-changing REST endpoints требуют nonce и проверку прав доступа.
- Проверить CORS-настройки WordPress ядра, темы, плагинов и reverse proxy/web сервера.

---
# 6. Negative / positive checks

Следующие проверки не подтвердили наличие уязвимости:

| Проверка | Результат |
|---|---|
| `/xmlrpc.php` | возвращает 404; XML-RPC не подтвердился как доступный endpoint |
| `/wp-json/wp/v2/users` | возвращает 401; REST users endpoint ограничен |
| Search XSS | отражение найдено, но payload HTML-encoded; XSS не подтверждён |
| HTTP TRACE | `405 Not Allowed`; TRACE не включён |
| Dangerous HTTP methods | Nmap показал только `GET`, `HEAD`, `POST`, `OPTIONS` |
| TLS cipher suites | слабые TLS версии/ciphers не выявлены; observed least strength: A |
| Directory listing | `Index of` не подтверждён; `/wp-content/uploads/` возвращал `403 Forbidden` |
| common sensitive files | `.env`, debug.log, backups/dumps/phpinfo по типовым путям не подтвердились |

---

# 7. Приоритеты исправления

## Срочно

1. Ограничить внешний доступ к MySQL/MariaDB `3306/tcp`.
2. Ограничить внешний доступ к FTP/SSH/SFTP через VPN/firewall/IP белых списков.
3. Отключить SSH парольную аутентефикацию, если возможно, и перейти на ключи + MFA/VPN.
4. Обновить PHP, WordPress ядра, тему и плагины.
5. Проверить логи FTP/SSH/MySQL на попытки несанкционированного доступа.

## В ближайшее время

6. Исправить cookies: `Secure`, `HttpOnly`, `SameSite`.
7. Добавить заголовки безопасности: CSP, HSTS, Permissions-Policy, X-Frame-Options/frame-ancestors, X-Content-Type-Options.
8. Исправить reflective CORS policy с credentials.
9. Отключить перечисление авторов в WordPress и переименовать административного пользователя `<REDACTED_USERNAME>`.
10. Защитить AJAX actions плагина `<REDACTED_PLUGIN>` nonce/CSRF-token, проверку прав и частоту отправки.
11. Отключить legacy SSH алгоритмы.

## Планово

12. Удалить/ограничить служебные WordPress-файлы и endpoints.
13. Исправить HTTP-ссылки на HTTPS-страницах и redirect-chain для `www`.
14. Обновить sitemap и убрать устаревшие URL.
15. Исправить нерабочие ссылки в разделе для абитуриентов.

---

# 9. Итоговая оценка

Общий уровень риска оценивается как **High** из-за публичной доступности административных и database-сервисов на выделенном сервере сайта. Остальные findings усиливают риск за счёт устаревшего стека, раскрытия административный username `<REDACTED_USERNAME>`, слабых cookie attributes, неполных security headers и unauthenticated AJAX actions.

Наиболее важная цепочка риска:

```text
Публичные SSH/FTP/MySQL
+ SSH password authentication
+ раскрытый WordPress username `<REDACTED_USERNAME>`
+ устаревший WordPress/PHP/JS стек
+ unauthenticated AJAX/REST attack surface
= повышенная вероятность targeted credential attacks и эксплуатации известных уязвимостей
```

Ключевая рекомендация: сначала закрыть внешний доступ к инфраструктурным сервисам и обновить технологический стек, затем исправить WordPress hardening issues и browser/client-side protections.
