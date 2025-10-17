# 🐳 Контейнер для поиска новостей

## ✅ Создан отдельный Docker контейнер для автоматического поиска новостей

### 📋 Что было создано:

1. **`backend/Dockerfile.scraper`** - Dockerfile для контейнера скрапера
2. **`backend/scripts/scraper-entrypoint.sh`** - Entrypoint скрипт с cron
3. **`backend/scripts/scraper-cron`** - Конфигурация cron jobs
4. **`backend/scripts/run-scraper.sh`** - Скрипт для запуска скрапера
5. **Обновлен `docker-compose.yml`** - добавлен сервис `news-scraper`

### ⏰ Расписание работы:

- **Каждый час** (в 00 минут): Запуск `scrape_all_companies.py`
- **Каждые 6 часов**: Запуск `populate_news.py`
- **Еженедельно**: Очистка старых логов

### 🚀 Запуск:

```bash
# Запуск всех сервисов (включая скрапер)
docker-compose up -d

# Запуск только скрапера
docker-compose up news-scraper -d

# Одноразовый запуск скрапера
docker-compose run --rm news-scraper /usr/local/bin/run-scraper.sh
```

### 📊 Мониторинг:

```bash
# Просмотр логов скрапера
docker-compose logs -f news-scraper

# Логи скрапинга
docker-compose exec news-scraper tail -f /var/log/scraper.log

# Проверка cron jobs
docker-compose exec news-scraper crontab -l
```

### 🔧 Управление:

```bash
# Остановка
docker-compose stop news-scraper

# Перезапуск
docker-compose restart news-scraper

# Пересборка
docker-compose build news-scraper
```

### 📁 Файлы:

- `backend/scripts/SCRAPER_CONTAINER_README.md` - Подробная документация
- `backend/Dockerfile.scraper` - Dockerfile контейнера
- `backend/scripts/scraper-entrypoint.sh` - Entrypoint скрипт
- `backend/scripts/scraper-cron` - Cron конфигурация
- `backend/scripts/run-scraper.sh` - Скрипт запуска

### ✨ Особенности:

- ✅ Автоматический перезапуск при сбоях
- ✅ Health check для мониторинга
- ✅ Подробное логирование
- ✅ Ожидание готовности БД
- ✅ Изолированная среда выполнения
- ✅ Минимальное потребление ресурсов

Контейнер готов к использованию! 🎉



