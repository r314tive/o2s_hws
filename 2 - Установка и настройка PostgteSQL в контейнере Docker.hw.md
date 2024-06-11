# Домашнее задание: Установка и настройка PostgreSQL в контейнере Docker


## Шаги выполнения

### 1. ssh connect
```bash
ssh r314tive@84.201.139.214
```

### 2. Установка Docker Engine

1. Установка Docker:
   ```bash
   sudo apt update
   sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io
   ```

2. Своего пользователя в группу Docker:
   ```bash
   sudo usermod -aG docker $USER
   ```

3. Чтобы изменения вступили в силу:
   ```bash
   newgrp docker
   ```

### 3. Создание каталога /var/lib/postgres

```bash
sudo mkdir -p /var/lib/postgres
sudo chown $USER:$USER /var/lib/postgres
```

### 4. Разворачиваем контейнер с PostgreSQL 14

Запускаем контейнер с PostgreSQL, смонтировав каталог /var/lib/postgres:
```bash
docker run -d \
  --name postgres-server \
  -e POSTGRES_PASSWORD=password \
  -v /var/lib/postgres:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:14
```

### 5. Разворачиваем контейнер с клиентом PostgreSQL

Запускаем контейнер с клиентом PostgreSQL:
```bash
docker run -it --rm --network host postgres:14 psql -h 127.0.0.1 -U postgres
```

### 6. Создаём таблицу и добавляем данные

Подключаемся к контейнеру с клиентом и создаём таблицу:
```sql
CREATE TABLE persons(id serial PRIMARY KEY, first_name TEXT, second_name TEXT);
INSERT INTO persons(first_name, second_name) VALUES ('ivan', 'ivanov');
INSERT INTO persons(first_name, second_name) VALUES ('petr', 'petrov');
SELECT * FROM persons;
```

### 7. Подключаемся к контейнеру с сервера извне

Настраиваем правила файервола, чтобы разрешить внешние подключения к PostgreSQL. В YC это можно сделать через консоль управления. Затем подключаемся к контейнеру с PostgreSQL с вашего локального компьютера с помощью любого клиента PostgreSQL, например, `psql`

```bash
psql -h 84.201.139.214  -U postgres 
```

### 8. Удаляем и повторно создаём контейнер

Удаляем контейнер с сервером:
```bash
docker rm -f postgres-server
```

Создаём контейнер заново:
```bash
docker run -d \
  --name postgres-server \
  -e POSTGRES_PASSWORD=password \
  -v /var/lib/postgres:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:14
```

### 9. Проверяем сохранность данных

Снова подключаемся из контейнера с клиентом к контейнеру с сервером и проверяем данные:
```bash
docker run -it --rm --network host postgres:14 psql -h 127.0.0.1 -U postgres
SELECT * FROM persons;
```

### Комментарии по выполнению и решения проблем

- При настройке Docker убеждаемся, что все команды выполняются от имени пользователя с правами администратора.
- Для подключения извне убеждаемся, что порты открыты и разрешены правилами файервола.
- Если возникают проблемы с подключением, проверяем настройки сети и правильность указанных IP-адресов.

## Выводы

Все шаги были успешно выполнены, PostgreSQL был установлен и настроен в Docker-контейнере, данные сохраняются при перезапуске контейнера.


# Краткий отчет:

## Цель
Развернуть ВМ в GCP/ЯО/Аналоги, установить Docker, развернуть контейнер с PostgreSQL, настроить контейнер для внешнего подключения и проверить сохранность данных при перезапуске контейнера.

## Шаги выполнения
1. Создаём инстанс с Ubuntu 20.04
2. Устанавливаем Docker Engine
3. Создаём каталог `/var/lib/postgres`
4. Разворачиваем контейнер с PostgreSQL 14
5. Разворачиваем контейнер с клиентом PostgreSQL
6. Создаём таблицу и добавляем данные
7. Подключаемся к контейнеру с сервера извне
8. Удаляем и повторно создаём контейнер
9. Проверяем сохранность данных

## Комментарии и решения проблем
- Убеждаемся в правильности выполнения всех команд от имени администратора.
- Настраиваем правила файервола для разрешения внешних подключений.
- Проверяем настройки сети при возникновении проблем с подключением.

## Выводы
Все шаги успешно выполнены, данные сохраняются при перезапуске контейнера.

## Additional 

Лучше конечно же юзать docker-compose, удобнее.

### Установка Docker Compose

1. Устанавливаем Docker Compose:
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -Po '(?<="tag_name": ")(v[0-9]+\.[0-9]+\.[0-9]+)')" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

2. Проверяем установку:
   ```bash
   docker-compose --version
   ```

### Создание файла `docker-compose.yml`

Создаём файл `docker-compose.yml` в проекте с следующим содержимым:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    container_name: postgres-server
    environment:
      POSTGRES_PASSWORD: yourpassword
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  pgclient:
    image: postgres:14
    container_name: postgres-client
    depends_on:
      - postgres
    entrypoint: ["tail", "-f", "/dev/null"]

volumes:
  postgres-data:
```


1. Останавливаем контейнер:
   ```bash
   docker-compose down
   ```

2. Запускаем контейнер заново:
   ```bash
   docker-compose up -d
   ```
