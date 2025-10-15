# 🤖 Настройка Telegram бота для дайджестов

## 📋 Обзор

Telegram бот для AI Competitor Insight Hub позволяет пользователям получать персонализированные дайджесты новостей прямо в Telegram. Бот поддерживает:

- ✅ **Автоматическую отправку дайджестов** по расписанию
- ✅ **Интерактивные команды** (/start, /help, /digest, /settings)
- ✅ **Inline keyboards** для удобного взаимодействия
- ✅ **Webhook интеграцию** для real-time обработки команд
- ✅ **Персонализированные настройки** для каждого пользователя

---

## 🚀 Быстрая настройка

### 1. Добавьте токен бота в `.env`

```env
# Telegram
TELEGRAM_BOT_TOKEN=8358550051:AAFVEGVvywc3n636YgYnHFT56JAbYgGIKLA
TELEGRAM_CHANNEL_ID=@your_channel_name  # Опционально
```

### 2. Запустите сервер

```bash
# Запустите backend сервер
cd backend
python main.py

# В другом терминале запустите Celery worker
celery -A celery_app worker --loglevel=info

# В третьем терминале запустите Celery beat для планирования
celery -A celery_app beat --loglevel=info
```

### 3. Настройте webhook (для production)

```bash
# Установите webhook URL
curl -X GET "http://localhost:8000/api/v1/telegram/set-webhook?webhook_url=https://yourdomain.com/api/v1/telegram/webhook"

# Проверьте статус webhook
curl -X GET "http://localhost:8000/api/v1/telegram/get-webhook-info"
```

### 4. Протестируйте бота

```bash
# Запустите тестовый скрипт
cd backend
python scripts/test_telegram_bot.py
```

---

## 🎯 Как использовать

### Для пользователей:

1. **Найдите бота**: `@short_news_sender_bot`
2. **Отправьте `/start`** - получите Chat ID
3. **Скопируйте Chat ID** и добавьте в настройки профиля на сайте
4. **Настройте дайджесты** в веб-приложении
5. **Получайте дайджесты** автоматически!

### Команды бота:

- `/start` - Начать работу и получить Chat ID
- `/help` - Показать справку
- `/digest` - Получить дайджест сейчас
- `/settings` - Показать настройки

---

## 🔧 API Endpoints

### Webhook
- `POST /api/v1/telegram/webhook` - Обработка сообщений от Telegram

### Управление webhook
- `GET /api/v1/telegram/set-webhook` - Установить webhook URL
- `GET /api/v1/telegram/delete-webhook` - Удалить webhook
- `GET /api/v1/telegram/get-webhook-info` - Информация о webhook

### Тестирование
- `POST /api/v1/telegram/send-test-message` - Отправить тестовое сообщение

---

## 🏗️ Архитектура

### Компоненты:

1. **`TelegramService`** (`backend/app/services/telegram_service.py`)
   - Отправка сообщений через Telegram API
   - Управление webhook
   - Форматирование сообщений

2. **`Bot Handlers`** (`backend/app/bot/handlers.py`)
   - Обработка команд бота
   - Создание inline keyboards
   - Интеграция с базой данных

3. **`Webhook Endpoint`** (`backend/app/api/v1/endpoints/telegram.py`)
   - Прием обновлений от Telegram
   - Маршрутизация команд
   - Обработка callback queries

4. **`Digest Integration`** (`backend/app/tasks/digest.py`)
   - Автоматическая генерация дайджестов
   - Отправка через Telegram
   - Планирование через Celery

### Поток данных:

```
Telegram → Webhook → Bot Handlers → Database → Digest Service → Telegram
```

---

## 📱 Интерфейс бота

### Приветственное сообщение (/start):

```
👋 Добро пожаловать в AI Competitor Insight Hub!

Ваш Chat ID: 123456789

Скопируйте этот ID и добавьте его в настройки вашего профиля 
на веб-платформе, чтобы получать персонализированные дайджесты новостей.

Выберите действие:

[📊 Получить дайджест] [⚙️ Настройки]
[📚 Помощь] [🔗 Открыть веб-приложение]
```

### Меню дайджестов (/digest):

```
📰 Выберите тип дайджеста:

[📅 Дневной дайджест] [📊 Недельный дайджест]
[⚙️ Настройки дайджеста]
```

### Настройки:

```
⚙️ Настройки дайджеста:

📊 Дайджесты: ✅ Включены
📅 Частота: daily
📝 Формат: short
🌐 Часовой пояс: UTC

[🔗 Открыть настройки]
```

---

## 🔄 Автоматические дайджесты

### Планирование:

- **Ежедневные дайджесты**: Отправляются каждый день в указанное время
- **Еженедельные дайджесты**: Отправляются раз в неделю
- **Кастомные дайджесты**: По настроенному расписанию

### Процесс отправки:

1. Celery beat запускает задачу по расписанию
2. Система находит пользователей с включенными дайджестами
3. Для каждого пользователя генерируется персонализированный дайджест
4. Если Telegram включен, дайджест отправляется через `TelegramService`

---

## 🛠️ Разработка и отладка

### Логирование:

```python
from loguru import logger

logger.info("Telegram message sent successfully")
logger.error(f"Failed to send message: {error}")
```

### Тестирование:

```bash
# Тест токена бота
curl "https://api.telegram.org/bot<TOKEN>/getMe"

# Тест отправки сообщения
curl -X POST "http://localhost:8000/api/v1/telegram/send-test-message" \
  -H "Content-Type: application/json" \
  -d '{"chat_id": "YOUR_CHAT_ID", "message": "Test message"}'
```

### Отладка webhook:

```bash
# Проверить webhook info
curl "http://localhost:8000/api/v1/telegram/get-webhook-info"

# Удалить webhook (для отладки)
curl "http://localhost:8000/api/v1/telegram/delete-webhook"
```

---

## 🔒 Безопасность

### Важные моменты:

1. **Никогда не коммитьте токен бота** в git
2. **Используйте HTTPS** для webhook в production
3. **Валидируйте все входящие данные** от Telegram
4. **Ограничьте доступ** к webhook endpoint
5. **Мониторьте логи** на предмет подозрительной активности

### Настройка webhook с секретом:

```python
# В TelegramService
payload = {
    "url": webhook_url,
    "secret_token": "your_secret_token",
    "allowed_updates": ["message", "callback_query"]
}
```

---

## 📊 Мониторинг

### Ключевые метрики:

- Количество отправленных дайджестов
- Успешность доставки сообщений
- Количество активных пользователей
- Время генерации дайджестов

### Логи для мониторинга:

```bash
# Логи Telegram сервиса
grep "Telegram" logs/app.log

# Логи отправки дайджестов
grep "Digest sent" logs/app.log

# Ошибки Telegram API
grep "Telegram API error" logs/app.log
```

---

## 🚨 Устранение неполадок

### Бот не отвечает:

1. Проверьте токен в `.env`
2. Убедитесь, что сервер запущен
3. Проверьте webhook настройки
4. Посмотрите логи на ошибки

### Дайджесты не отправляются:

1. Проверьте настройки пользователя
2. Убедитесь, что Celery worker запущен
3. Проверьте Celery beat планировщик
4. Проверьте Chat ID пользователя

### Webhook не работает:

1. Убедитесь, что URL доступен извне
2. Проверьте HTTPS сертификат
3. Проверьте настройки файрвола
4. Используйте ngrok для локальной разработки

### Ошибки форматирования:

1. Проверьте Markdown синтаксис
2. Экранируйте специальные символы
3. Разбивайте длинные сообщения
4. Тестируйте на разных устройствах

---

## 📚 Дополнительные ресурсы

- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)
- [BotFather Guide](https://core.telegram.org/bots#6-botfather)
- [Webhook Setup Guide](https://core.telegram.org/bots/webhooks)
- [Inline Keyboards Guide](https://core.telegram.org/bots/api#inlinekeyboardmarkup)

---

## 🎉 Готово!

Ваш Telegram бот настроен и готов к работе! Пользователи теперь могут:

- Получать персонализированные дайджесты
- Взаимодействовать с ботом через команды
- Настраивать свои предпочтения
- Получать уведомления в реальном времени

Для начала работы найдите бота `@short_news_sender_bot` в Telegram и отправьте `/start`!
