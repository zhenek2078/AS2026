# Доступ к WEB ClickHouse только с localhost

У ClickHouse веб-интерфейс обычно:

8123 — HTTP

8443 — HTTPS (если есть)

## Вариант 1: биндим только на localhost - предпочтительнее

Открываем конфиг:

```
nano /etc/clickhouse-server/config.xml
```

Ищем строку:

```
<listen_host>::</listen_host>
```

Заменяем ее на:

```
<listen_host>127.0.0.1</listen_host>
```

Перезапускаем:

```
systemctl restart clickhouse-server
```

## Вариант 2: через firewall - вряд ли оценят

Создаем правило:

```
nft add rule inet filter input tcp dport 8123 ip saddr != 127.0.0.1 dropnft add rule inet filter input tcp dport 8443 ip saddr != 127.0.0.1 drop
```

# Скрыть все панели администратора с внешним доступом

Сначала смотришь, что вообще торчит наружу:

```
ss -tulpn
```

Исходя из этого есть несколько вариантов сокрытия админ-панели:

## Вариант 1: nginx

Открываем конфиг сайта:

```
nano /etc/nginx/sites-available/*
```

И добавляем:

```
location /admin {
    allow 127.0.0.1;
    deny all;
}
```

## Вариант 2: Python Flask / FastAPI

Ищем расположение проекта:

```
ps aux | grep -E 'python|gunicorn'
```

Или:

```
ss -tulpn | grep python
```

Например, запущен файл ```app.py```, тогда ищем его расположение:

```
find / -name "app.py" 2>/dev/null
```

В файл добавляем строки:

```
@app.route("/admin")
def admin():
    if request.remote_addr != "127.0.0.1":
        abort(403)
```

## Вариант 3: Django

Ищем файл ```settings.py``` по путям:

```
/opt/project/
/var/www/project/
/srv/project/
```

В файл ```settings.py``` добавляем следующее:

```
ALLOWED_IPS = ["127.0.0.1"]
```

## Вариант 4: Node.js (Express)

Ищем файл запуска:

```
ps aux | grep node
```

Обычно ```/app/server.js/app/app.js```.

Открываем этот файл и добавляем:

```
app.use('/admin', (req, res, next) => {
    const ip = req.ip;
    if (ip !== '127.0.0.1') {
        return res.status(403).send('Forbidden');
    }
    next();
});
```

## Вариант 5: Java (Spring)

Ищем java-файлы:

```
find / -name "*SecurityConfig*.java"
```

Или:

```
find / -name "application.yml"
```

В ```SecurityConfig.java``` добавляем:

```
http  .authorizeHttpRequests().requestMatchers("/admin/**").hasIpAddress("127.0.0.1");
```

## Вариант 6: Apache

Ищем в:

```
/etc/apache2/apache2.conf
/etc/apache2/sites-available/
/etc/apache2/sites-enabled/
```

Добавляем:

```
<Location "/admin">Require ip 127.0.0.1</Location>
```

## Вариант 7: firewall

Если админка на отдельном порту, например, 8080, то просто добавляем правило:

```
nft add rule inet filter input tcp dport 8080 ip saddr != 127.0.0.1 drop
```
