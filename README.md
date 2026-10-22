# Unsloth Studio

Compose запускает Unsloth с доступом к NVIDIA GPU. Интерфейс доступен по
**http://localhost:9898**. внутри
контейнера Studio слушает порт 8000

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

Для этого настройте свой веб сервер Caddy/nginx/apache. Добавьте в его
Caddyfile блок и и перезагрузите конфигурацию:

```caddyfile
unsloth.domain.com {
    reverse_proxy ip:9898
}
```
Проверка с ПК:

```powershell
curl.exe http://ip:9898/api/health
```

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
