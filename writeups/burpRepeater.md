# Burp Repeater

- зачем нужен Repeater
- отличие Proxy от Repeater
- как интерпретировать изменения ответа

Reprater в Burp Suite - инструмент для ручного тестирования, позволяющий 
редактировать, писать новые запросы и анализировать ответы от сервера. 
Можно перехватить запрос в Proxy, отправить его в Repeater, изменить заголовки, 
параметры или тело запроса и отправить снова.

Сам Reprater от Proxy отличается тем, что Proxy перехватывает трафик реального времени, а 
Reprater нужен для того, чтобы в ручном режиме манипулировать запросами

Интерпритация Reprater заключается во внимательном анализе responce, статус-коды, заголовки, 
тела ответов, внимательно наблюдать за тем, как именно сервер отвечат на те или иные запросы

Так вот, как было описано в отчете про Proxy и HTTP history, во время анализа трафика можно было
заметить подозрительные куки сессии 
`Cookie: dl_session=eyJ1c2VyX2lkIjogMX0=.aedR9A.pjBuh2UIxvL7Hll4`, если взять первую часть и декодировать ее в
Base 64, можно получить `{"user_id": 1}`, однако в том отчете так и не разобрались в том, можно ли 
подставить кук бругой сессии и зайти на другой аккаунт без пароля и логина, однако есть загвоздка,
если просто переставить значения, перехватив запрос с куком-сессии в intercept 
на эти `{"user_id": 2} = eyJ1c2VyX2lkIjogMn0=`, браузер не даст этого сделать, ибо он просто 
переотправит запрос с правильным куки, который есть у него. В этом нам может помочь Reprater,
просто заходим в HTTP history, находим Request подобного вида:

```
GET / HTTP/1.1
Host: 104.194.133.33
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Connection: keep-alive
Cookie: dl_session=eyJ1c2VyX2lkIjogMX0=.aekGlQ.hQAKFJedrDprC0q-Vc
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

И нажав ПКМ выбрать send to repeater, далее подставляем значение `dl_session` с
`Cookie: dl_session=eyJ1c2VyX2lkIjogMX0=.aekGlQ.hQAKFJedrDprC0q-Vc` на
`Cookie: dl_session=eyJ1c2VyX2lkIjogMn0=.aekGlQ.hQARFJedrDprC0q-Vc`, а после читаем 
Responce 

``` Responce 
HTTP/1.1 504 Gateway Time-out
Server: nginx/1.27.5
Date: Wed, 22 Apr 2026 17:43:39 GMT
Content-Type: text/html
Content-Length: 167
Connection: keep-alive

<html>
<head><title>504 Gateway Time-out</title></head>
<body>
<center><h1>504 Gateway Time-out</h1></center>
<hr><center>nginx/1.27.5</center>
</body>
</html>
```

Как можно заметить уязвимость обнаружена! Однако это не IDOR, видимо сервер, приняв 
видоизмененный кук учел и ключ сессии (.aekGlQ.hQARFJedrDprC0q-Vc), поняв, что ключ не подходит
к этой сессии бекэнд начал беспрерывно ее обрабатывать, что в итоге повлекло ошибку 504

Восстановив сервер можно 
