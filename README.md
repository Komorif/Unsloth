# Unsloth Studio

Compose запускает Unsloth с доступом к NVIDIA GPU. Интерфейс доступен по
**http://192.168.1.183:9898**. Порт привязан к этому локальному IP; внутри
контейнера Studio слушает порт 8000. Если IP компьютера изменится, обновите
привязку в `docker-compose.yml` и адрес в основном Caddy.

## Запуск

Нужны Docker Desktop с Linux containers / WSL 2 и драйвер NVIDIA с поддержкой WSL.

```powershell
cd E:\Unsloth
docker compose up -d
docker compose ps
docker compose logs --tail 100 -f unsloth
```

Файл `.env` уже содержит случайный начальный пароль. Для новой установки
скопируйте `.env.example` в `.env` и задайте собственный длинный пароль.
Пользователь: `unsloth`. Начальный пароль: `UNSLOTH_STUDIO_PASSWORD` из `.env`.
Переменная применяется только при первой инициализации. Для дальнейшей смены
используйте интерфейс или `docker compose exec unsloth unsloth studio reset-password`.

Если команда `docker` не найдена:

```powershell
$env:PATH = "$env:LOCALAPPDATA\Programs\DockerDesktop\resources\bin;" + $env:PATH
docker compose up -d
```

## Доступ из интернета

HTTPS обслуживает существующий Caddy на основном медиасервере. Добавьте в его
Caddyfile блок из `Caddyfile.gateway.example` и перезагрузите конфигурацию:

```caddyfile
unsloth.komapp.net {
    reverse_proxy 192.168.1.183:9898
}
```

Цепочка: интернет → основной Caddy → `192.168.1.183:9898` → Unsloth.
DNS `unsloth.komapp.net` и перенаправление TCP 80/443 на роутере остаются направлены
на основной медиасервер. Он получает и продлевает публичный сертификат.
Порт 9898 должен быть доступен с медиасервера по локальной сети.

В этом Compose Caddy больше нет. Порты 80 и 443 этим проектом не занимаются.
Файл `Caddyfile.gateway.example` — пример для основного сервера, Compose его не загружает.

Проверка с ПК:

```powershell
curl.exe http://192.168.1.183:9898/api/health
```

После настройки основного Caddy открывайте **https://unsloth.komapp.net**.

## Данные и управление

Пользователи, чаты, настройки и результаты Studio, рабочая директория и кэш
Hugging Face хранятся в постоянных Docker volumes. Каталог `E:\Unsloth\work`
доступен внутри контейнера как `/workspace/host`. Jupyter и SSH на хост не публикуются.
Данные текущего контейнера сохранены при смене конфигурации.

Существующая Windows-установка на `127.0.0.1:8888` работает отдельно;
её чаты, модели и настройки автоматически не переносятся.

```powershell
# Остановить с сохранением данных
docker compose down
# Обновить образ и запустить
docker compose pull
docker compose up -d
```

`docker compose down -v` удаляет постоянные данные. Для старта после перезагрузки
включите автозапуск Docker Desktop.
