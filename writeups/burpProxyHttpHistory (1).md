# Burp Proxy + HTTP History


Нужно разобрать: 

* Как работает proxy
* intercept on / off
* Что важно в HTTP history, на что следует обращать внимание

Прокси Burp работает как последник трафика между сервером и браузером, перехватывая, анализируя
и позволяя менять трафик, изменяя запросы

Intercept on - перед отвравкой запросы в браузере открывается страница в burp, на которой отобразится 
текущий запрос с возможностью редактирования

HTTP history в Burp Suite - журнал всех request / response, которые проходят через прокси, 
в нем можно проанализировать трафикЮ типы ответов, параметры запроса и тд. в нем следует искать 
ошибки со стороны сервера при ответе за запросы, смотреть типы response и тд.

## Анализ трафика в Burp

Заходя на собственную веб страничку попадаю на форму заполнения пароля и входа в аккаунт,
лог у burp выглядит так:

``` response
Server: nginx/1.27.5
Date: Tue, 21 Apr 2026 09:42:28 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 3463
Connection: keep-alive

<!DOCTYPE html>
<html lang="ru">
<head>
/// - Остальной код
</head>
<body class="app-body">
/// - Остальной код
<div class="container pb-5">
    
<div class="row justify-content-center">
  <div class="col-lg-5 col-md-7">
    <div class="card shadow-sm">
      <div class="card-body p-4">
        <h1 class="h3 mb-3">Вход в систему</h1>
        <p class="text-muted">Авторизуйтесь, чтобы открыть дашборд и API.</p>
        <form method="post" action="/login" class="d-grid gap-3">
          <div>
            <label class="form-label">Email</label>
            <input class="form-control" type="email" name="email" required>
          </div>
          <div>
            <label class="form-label">Пароль</label>
            <input class="form-control" type="password" name="password" required>
          </div>
          <button class="btn btn-primary" type="submit">Войти</button>
        </form>
        
        <div class="small text-muted mt-3">Если учетной записи нет, используйте <a href="/register">регистрацию</a>.</div>
        
      </div>
    </div>
  </div>
</div>

</div>
<script>
    (() => {
        const button = document.getElementById('themeToggleButton');
        if (!button) {
            return;
        }
        const key = 'dl-theme';

        function applyTheme(theme) {
            document.documentElement.dataset.theme = theme;
            button.textContent = theme === 'dark' ? 'Светлая тема' : 'Темная тема';
        }

        applyTheme(localStorage.getItem(key) || document.documentElement.dataset.theme || 'light');
        button.addEventListener('click', () => {
            const nextTheme = document.documentElement.dataset.theme === 'dark' ? 'light' : 'dark';
            localStorage.setItem(key, nextTheme);
            applyTheme(nextTheme);
        });
    })();
</script>
</body>
</html>
```

Здесь можно заметить клиентский JavaScript в теге `<script>`, и обычную форму входа с 
ссылкой на форму регистрации в виде кода HTML

Введя пароль и логин браузер переходит на основную форму, нам нужно разобрать request / response
из этого нам нужны тольно методы, куки, заголовки и тд.

``` request / response
GET /sources HTTP/1.1
Host: 104.194.133.33
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://104.194.133.33/
Cookie: dl_session=eyJ1c2VyX2lkIjogMX0=.aedR9A.pjBuh2UIxvL7Hll4
Upgrade-Insecure-Requests: 1

-----------------------------------------------------------------------
response

HTTP/1.1 200 OK
Server: nginx/1.27.5
Date: Tue, 21 Apr 2026 10:31:39 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 24344
Connection: keep-alive
vary: Cookie
```

В request можно увидеть 

- Метод GET 
- Протокол HTTP/1.1
- Путь /sources
- Хост 104.194.133.33

Есть куки сессии, если разобрать по Base64 эту часть `eyJ1c2VyX2lkIjogMX0=`, получится JSON
 вида `{"user_id": 1}`, возможно содежит пользовательские данные, но это требует последующей 
проверки защищенностимеханизма подписи, невозможности подделки cookie и 
корректности серверной проверки сессии

В response можно просмотреть

- Статус код 200 OK
- Сервер nginx/1.27.5
- Тип контента text/html, кодировка charset=utf-8
- Кол-во строк контента 24344

Запрос означает, что делая GET запрос на страничку /sources, этот запрос идет от 
зарегестрированного пользователя, потому что есть куки сессии, сервер отвечает стандартным 
200 OK, после чего возвращает HTML страничку, а так как мы видим в ответе `vary: Cookie` ответ 
был получен и зависит от того, что пользователь уже вошел в систему

Пожалуй стоит разобрать запрос на вход в систему:
``` request / response
POST /login HTTP/1.1
Host: 104.194.133.33
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Content-Type: application/x-www-form-urlencoded
Content-Length: 45
Origin: http://104.194.133.33
Connection: keep-alive
Referer: http://104.194.133.33/login
Upgrade-Insecure-Requests: 1
Priority: u=0, i

login=admin&password=admin

-----------------------------------------------------------------------
response

HTTP/1.1 200 OK
Server: nginx/1.27.5
Date: Wed, 22 Apr 2026 17:34:13 GMT
Content-Length: 0
Connection: keep-alive
location: /
vary: Cookie
set-cookie: dl_session=eyJ1c2VyX2lkIjogMX0=.aekGlQ.hQAR18PnAXmPKFJC0q-Vc; 
path=/; Max-Age=604800; httponly; samesite=lax
```

Здесь можно выделить 

- Метод POST в request
- Путь /login
- login=admin&password=admin

А в response

- 200 OK
- set-cookie: dl_session=eyJ1c2VyX2lkIjogMX0=.aekGlQ.hQAR18PnAXmPKFJC0q-Vc;
- vary: Cookie

Тоесть, мы отправляем запрос на сервер методом POST в путь /login, чтобы авторизоваться
и по форме заполнения вписываем вписываем логин и пароль login=admin&password=admin,
на что сервер отвечает окей, кодом 200 OK, и выделяет на нас куки сессии set-cookie: dl_session, 
помечая, что следующие сраницы будут запускаться под этой кук-сессией vary: Cookie
