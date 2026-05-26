# Лабораторная работа «Администрирование платформ на ОС Linux»
**ИТМО, 3 курс, АП2, 2026**

**Студент:** Малахов Алексей Витальевич  
**Домен:** israel.zomai.ru  
**VPS IP:** 186.246.9.175  
**Дата выполнения:** 26.05.2026

---

## Задание 1. Корпоративный веб-сервер с SSL сертификатом

### 1.1 Установка пакетов

На VPS под управлением Debian была выполнена установка необходимых пакетов:

```bash
apt update && apt upgrade -y && apt install -y nginx curl resolvconf certbot ssh fail2ban pip python3-pip python3-venv jq uuid-runtime && pip install certbot-dns-beget-api --break-system-packages
```

### 1.2 Настройка nginx на порту 8080

Создан конфигурационный файл `/etc/nginx/sites-available/israel.zomai.ru`:

```nginx
server {
    listen 8080;
    server_name israel.zomai.ru;

    access_log /var/log/nginx/israel.zomai.ru-access.log;
    error_log  /var/log/nginx/israel.zomai.ru-error.log;

    root /var/www/israel.zomai.ru;
    index index.html;

    ssi on;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 443 ssl;
    server_name israel.zomai.ru;

    ssl_certificate     /etc/letsencrypt/live/israel.zomai.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/israel.zomai.ru/privkey.pem;

    access_log /var/log/nginx/israel.zomai.ru-access.log;
    error_log  /var/log/nginx/israel.zomai.ru-error.log;

    root /var/www/israel.zomai.ru;
    index index.html;

    ssi on;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Создана индексная страница `/var/www/israel.zomai.ru/index.html`:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>israel.zomai.ru</title>
</head>
<body>
    <p>ФИО: Малахов Алексей Витальевич</p>
    <p>Виртуальный хост: <!--# echo var="host" --></p>
    <p>Дата создания: 26.05.2026</p>
</body>
</html>
```

Активирован симлинк и перезапущен nginx:

```bash
ln -s /etc/nginx/sites-available/israel.zomai.ru /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

**Скриншот браузера — http на порту 8080:**

![Браузер http://israel.zomai.ru:8080](screenshots/browser.png)

**Скриншот конфигурационного файла nginx:**

![Конфиг /etc/nginx/sites-available/israel.zomai.ru](screenshots/nginx-conf.png)

### 1.3 Открытые порты — вывод ss -4tunl

```
Netid    State     Recv-Q    Send-Q    Local Address:Port    Peer Address:Port
udp      UNCONN    0         0         186.246.9.175%eth0:68  0.0.0.0:*
tcp      LISTEN    0         511       0.0.0.0:8080           0.0.0.0:*
tcp      LISTEN    0         128       0.0.0.0:10050          0.0.0.0:*
tcp      LISTEN    0         511       0.0.0.0:80             0.0.0.0:*
tcp      LISTEN    0         128       0.0.0.0:22             0.0.0.0:*
tcp      LISTEN    0         511       0.0.0.0:443            0.0.0.0:*
```

**Скриншот команды ss -4tunl:**

![Вывод ss -4tunl](screenshots/ss.png)

### 1.4 Получение SSL сертификата Let's Encrypt

Сертификат получен через DNS-челлендж. В панели reg.ru была создана TXT запись:

- Имя: `_acme-challenge.israel`
- Значение: предоставленное certbot

```bash
certbot certonly --manual --preferred-challenges dns -d israel.zomai.ru
```

Сертификат сохранён:
- `/etc/letsencrypt/live/israel.zomai.ru/fullchain.pem`
- `/etc/letsencrypt/live/israel.zomai.ru/privkey.pem`

Срок действия: до 24.08.2026.

**Скриншот браузера с информацией о SSL сертификате:**

![Браузер https://israel.zomai.ru с информацией о сертификате](screenshots/cert.png)

### 1.5 Журналы nginx

Журналы расположены в:
- `/var/log/nginx/israel.zomai.ru-access.log`
- `/var/log/nginx/israel.zomai.ru-error.log` (пустой — ошибок не было)

**Содержимое access.log:**

```
88.201.131.150 - - [26/May/2026:22:20:56 +0300] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.7.1"
5.61.209.126 - - [26/May/2026:22:33:19 +0300] "GET /SDK/webLanguage HTTP/1.1" 404 181 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.85 Safari/537.36 Edg/90.0.818.46"
88.201.131.150 - - [26/May/2026:22:35:40 +0300] "GET / HTTP/1.1" 200 263 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36"
88.201.131.150 - - [26/May/2026:22:35:40 +0300] "GET /favicon.ico HTTP/1.1" 404 181 "https://israel.zomai.ru/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36"
66.132.224.231 - - [26/May/2026:22:37:21 +0300] "GET / HTTP/1.1" 200 274 "-" "Mozilla/5.0 (compatible; CensysInspect/1.1; +https://about.censys.io/)"
45.198.224.186 - - [26/May/2026:22:38:43 +0300] "GET / HTTP/1.1" 200 322 "-" "Mozilla/5.0"
45.205.1.196 - - [26/May/2026:22:41:41 +0300] "GET / HTTP/1.1" 200 322 "-" "Mozilla/5.0"
45.148.10.67 - - [26/May/2026:22:42:26 +0300] "GET / HTTP/1.1" 200 274 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
87.236.176.4 - - [26/May/2026:22:44:29 +0300] "GET / HTTP/1.1" 200 274 "-" "Mozilla/5.0 (compatible; InternetMeasurement/1.0; +https://internet-measurement.com/)"
```

### 1.6 Файлы сертификата и закрытого ключа

Файлы сертификата получены через Let's Encrypt и сохранены локально:
- `fullchain.pem`
- `privkey.pem`

**fullchain.pem:**

```
-----BEGIN CERTIFICATE-----
MIIDhzCCAw6gAwIBAgISBnMwgiHSaZuhH0HYP5phjHPBMAoGCCqGSM49BAMDMDIx
CzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MQswCQYDVQQDEwJF
NzAeFw0yNjA1MjYxODMyMjhaFw0yNjA4MjQxODMyMjdaMBoxGDAWBgNVBAMTD2lz
cmFlbC56b21haS5ydTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABJgWRy5j9bPL
1CyMiTZYaIfgQoUy0BdbH6rYIjvZBOkERl+rgplB6T5qJwDjg92ThzdCadGkwI6c
PoJGTZZQkmejggIaMIICFjAOBgNVHQ8BAf8EBAMCB4AwEwYDVR0lBAwwCgYIKwYB
BQUHAwEwDAYDVR0TAQH/BAIwADAdBgNVHQ4EFgQUZRQfsiFAeU7XFrCZVKoogzzH
D+8wHwYDVR0jBBgwFoAUrkie3IcdRKBv2qLlYHQEeMKcAIAwMgYIKwYBBQUHAQEE
JjAkMCIGCCsGAQUFBzAChhZodHRwOi8vZTcuaS5sZW5jci5vcmcvMBoGA1UdEQQT
MBGCD2lzcmFlbC56b21haS5ydTATBgNVHSAEDDAKMAgGBmeBDAECATAtBgNVHR8E
JjAkMCKgIKAehhxodHRwOi8vZTcuYy5sZW5jci5vcmcvNjYuY3JsMIIBCwYKKwYB
BAHWeQIEAgSB/ASB+QD3AHUAyzj3FYl8hKFEX1vB3fvJbvKaWc1HCmkFhbDLFMMU
WOcAAAGeZcTU3wAABAMARjBEAiB0Dh0NZSTg0HC3sFq5x9Nj4W0ZHpQqxsUcOh2Y
0S1I+QIgNDGJrMpDMcpC40LgUPyu3mi+TtqnRLKCvQIFRzK4WwoAfgBGr4Y9Oz7l
n6V33qgkXTaw2e0ioiP0YXdBIpRS7pVQXwAAAZ5lxNTyAAgAAAUAB7ewPgQDAEcw
RQIhAMv6Whd2PEn+14PAS5r187ZuRuPeMZBbsiw2LqrKjfueAiB3KjZUnqWOgPlL
1n+ogGpeylLo9EVpXcvhdcLwKEINBTAKBggqhkjOPQQDAwNnADBkAjAwMurK8J4y
FFsQJ9X87qjsfDRvWbEb4dv27jMTEefxL7OznNuO+AWyoXHqBiNfykUCMEAzNMKl
fqDJqQoo0G1o7rfFHwfPgLtEEUnyllbPXhLRr23iP15DnjbeG49drmBsXg==
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIEVzCCAj+gAwIBAgIRAKp18eYrjwoiCWbTi7/UuqEwDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMjQwMzEzMDAwMDAw
WhcNMjcwMzEyMjM1OTU5WjAyMQswCQYDVQQGEwJVUzEWMBQGA1UEChMNTGV0J3Mg
RW5jcnlwdDELMAkGA1UEAxMCRTcwdjAQBgcqhkjOPQIBBgUrgQQAIgNiAARB6AST
CFh/vjcwDMCgQer+VtqEkz7JANurZxLP+U9TCeioL6sp5Z8VRvRbYk4P1INBmbef
QHJFHCxcSjKmwtvGBWpl/9ra8HW0QDsUaJW2qOJqceJ0ZVFT3hbUHifBM/2jgfgw
gfUwDgYDVR0PAQH/BAQDAgGGMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
ATASBgNVHRMBAf8ECDAGAQH/AgEAMB0GA1UdDgQWBBSuSJ7chx1EoG/aouVgdAR4
wpwAgDAfBgNVHSMEGDAWgBR5tFnme7bl5AFzgAiIyBpY9umbbjAyBggrBgEFBQcB
AQQmMCQwIgYIKwYBBQUHMAKGFmh0dHA6Ly94MS5pLmxlbmNyLm9yZy8wEwYDVR0g
BAwwCjAIBgZngQwBAgEwJwYDVR0fBCAwHjAcoBqgGIYWaHR0cDovL3gxLmMubGVu
Y3Iub3JnLzANBgkqhkiG9w0BAQsFAAOCAgEAjx66fDdLk5ywFn3CzA1w1qfylHUD
aEf0QZpXcJseddJGSfbUUOvbNR9N/QQ16K1lXl4VFyhmGXDT5Kdfcr0RvIIVrNxF
h4lqHtRRCP6RBRstqbZ2zURgqakn/Xip0iaQL0IdfHBZr396FgknniRYFckKORPG
yM3QKnd66gtMst8I5nkRQlAg/Jb+Gc3egIvuGKWboE1G89NTsN9LTDD3PLj0dUMr
OIuqVjLB8pEC6yk9enrlrqjXQgkLEYhXzq7dLafv5Vkig6Gl0nuuqjqfp0Q1bi1o
yVNAlXe6aUXw92CcghC9bNsKEO1+M52YY5+ofIXlS/SEQbvVYYBLZ5yeiglV6t3S
M6H+vTG0aP9YHzLn/KVOHzGQfXDP7qM5tkf+7diZe7o2fw6O7IvN6fsQXEQQj8TJ
UXJxv2/uJhcuy/tSDgXwHM8Uk34WNbRT7zGTGkQRX0gsbjAea/jYAoWv0ZvQRwpq
Pe79D/i7Cep8qWnA+7AE/3B3S/3dEEYmc0lpe1366A/6GEgk3ktr9PEoQrLChs6I
tu3wnNLB2euC8IKGLQFpGtOO/2/hiAKjyajaBP25w1jF0Wl8Bbqne3uZ2q1GyPFJ
YRmT7/OXpmOH/FVLtwS+8ng1cAmpCujPwteJZNcDG0sF2n/sc0+SQf49fdyUK0ty
+VUwFj9tmWxyR/M=
-----END CERTIFICATE-----
```

**privkey.pem:**

```
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgldAaBjQtYuu7poC4
VncAWrHaeQa3JVQoKuq+8HMyIDChRANCAASYFkcuY/Wzy9QsjIk2WGiH4EKFMtAX
Wx+q2CI72QTpBEZfq4KZQek+aicA44Pdk4c3QmnRpMCOnD6CRk2WUJJn
-----END PRIVATE KEY-----
```

---
