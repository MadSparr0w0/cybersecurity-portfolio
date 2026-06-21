 # Nmap

- enumeration
- open / closed / filtered
- `-sV`, `-sC`, `-Pn`, `-p-`

Enumeration в nmap - процесс сканирования сети на открытые порты, службы, версии, 
уязвимости в целевых хостах

Open (Открыт), Closed (Закрыт) и Filtered (Фильтруется) - статусы портов. 
Open означает, что приложение ожидает подключения, 
Closed — порт доступен, но не используется, 
а Filtered — брандмауэр блокирует запросы.

Ключи `-sV`, `-sC`, `-Pn`, `-p-`

- -sV — определение версий сервисов
- -sC — запуск стандартных NSE-скриптов - набор базовых скриптов для дополнительной проверки сервисов: баннеры, SSL, HTTP-заголовки, SMB-инфо и прочее.
- -Pn - запуск nmap без проверки на пинг
- -p- - сканировать все TCP-порты (1-65535)

Сканируем github
```
└─$ nmap 140.82.121.3       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-23 13:08 -0400
Nmap scan report for lb-140-82-121-3-fra.github.com (140.82.121.3)
Host is up (0.016s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

Как можно заметить, при обычном сканировании nmap обнаружил самые обычные порты 23, 80, 443,
но что если запустить со всеми ключами

``` sudo nmap -sV -sC -p- -Pn -T4  
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-24 01:53 -0400
Nmap scan report for lb-140-82-121-4-fra.github.com (140.82.121.4)
Host is up (0.15s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE SERVICE   VERSION
22/tcp  open  ssh       (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-a59182e
80/tcp  open  http
|_http-title: Did not follow redirect to https://lb-140-82-121-4-fra.github.com/
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 301 Moved Permanently
|     Content-Length: 0
|     Location: https:///
|     connection: close
|   RTSPRequest, X11Probe: 
|     HTTP/1.1 400 Bad Request
|     cache-control: no-cache
|     content-type: text/html; charset=utf-8
|     strict-transport-security: max-age=31536000; includeSubDomains; preload
|     x-content-type-options: nosniff
|     x-frame-options: deny
|     x-xss-protection: 0
|     content-security-policy: default-src 'none'; style-src 'unsafe-inline'
|     <html>
|     <head>
|     <meta content="origin" name="referrer">
|     <title>Bad request &middot; GitHub</title>
|     <meta name="viewport" content="width=device-width">
|     <style type="text/css" media="screen">
|     body {
|     background-color: #f6f8fa;
|     color: rgba(0, 0, 0, 0.5);
|     font-family: -apple-system,BlinkMacSystemFont,Segoe UI,Helvetica,Arial,sans-serif,Apple Color Emoji,Segoe UI Emoji,Segoe UI Symbol;
|     font-size: 14px;
|     line-height: 1.5;
|_    margin: 50px auto; max-width: 600px; text-align: center; padding: 0 24p
443/tcp open  ssl/https
|_ssl-date: TLS randomness does not represent time
|_http-title: Did not follow redirect to https://github.com/
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 301 Moved Permanently
|     Content-Length: 0
|     Location: https://github.com/nice%20ports%2C/Tri%6Eity.txt%2ebak
|     Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
|     connection: close
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 301 Moved Permanently
|     Content-Length: 0
|     Location: https://github.com/
|     Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
|     connection: close
|   tor-versions: 
|     HTTP/1.1 400 Bad Request
|     cache-control: no-cache
|     content-type: text/html; charset=utf-8
|     strict-transport-security: max-age=31536000; includeSubDomains; preload
|     x-content-type-options: nosniff
|     x-frame-options: deny
|     x-xss-protection: 0
|     content-security-policy: default-src 'none'; style-src 'unsafe-inline'
|     <html>
|     <head>
|     <meta content="origin" name="referrer">
|     <title>Bad request &middot; GitHub</title>
|     <meta name="viewport" content="width=device-width">
|     <style type="text/css" media="screen">
|     body {
|     background-color: #f6f8fa;
|     color: rgba(0, 0, 0, 0.5);
|     font-family: -apple-system,BlinkMacSystemFont,Segoe UI,Helvetica,Arial,sans-serif,Apple Color Emoji,Segoe UI Emoji,Segoe UI Symbol;
|     font-size: 14px;
|     line-height: 1.5;
|_    margin: 50px auto; max-width: 600px; text-align: center; padding: 0 24p
| ssl-cert: Subject: commonName=github.com
| Subject Alternative Name: DNS:github.com, DNS:www.github.com
| Not valid before: 2026-03-06T00:00:00
|_Not valid after:  2026-06-03T23:59:59
```

Так как сканирование происходид над большим веб-сервисом, как можно заметить даже со всеми 
ключами nmap показал, что большинство портов скрыто, что может говорить о вероятном firewall, 
минимальное количество открытых портов

- Порт 22 (SSH) 
Неточное определение версии, хотя он и доступен, но также частично скрыт


- Порт 80 (HTTP)
Открыт, но принудительный редирект на HTTPS


- Порт 443 (HTTPS)
Показывает сертификаты, сроки действия, редирект на основной домен `https://github.com/`
