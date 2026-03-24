> Чтобы поставить агента есть два способа: через ```apt install``` и через ```deb-пакет```, предоставляемый самим Wazuh. Рассмортим оба варианта.

## Через apt install

Ставим пакет:

```
apt-get install gnupg apt-transport-https
```

Устанавливаем GPG-ключ:

```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```

Добавляем ропозиторий:

```
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```

Обновляем пакеты:

```
apt-get update
```

Ставим агента и одновременно подключаем его к серверу:

```
WAZUH_MANAGER="192.168.2.200" apt-get install wazuh-agent
```

*Если вдруг не законнектился к серверу, то меняем ручками IP сервера:*

Редактируем конфиг:

```
nano /var/ossec/etc/ossec.conf
```

Находим строки и меняем адрес:

```
<client>
  <server>
    <address>192.168.2.200</address>
  </server>
</client>
```

Включаем агента:

```
systemctl daemon-reload
```

```
systemctl enable wazuh-agent
```

```
systemctl start wazuh-agent
```

## Через deb-пакет
