# Quick Start Guide - New Features

## Исправления выполнены ✅

Все ошибки в коде исправлены:
- ✅ Frontend компоненты убраны из DashboardLayout wrapper
- ✅ Добавлены routes в App.tsx
- ✅ NotificationCenter интегрирован в Header
- ✅ Все linter ошибки устранены

## Быстрый запуск

### 1. Запустите миграции базы данных

```bash
cd backend
alembic upgrade head
```

Это создаст новые таблицы:
- `notification_settings`
- `notifications`
- `competitor_comparisons`
- Добавит поля в `user_preferences`

### 2. Настройте переменные окружения (опционально)

Для работы Telegram бота добавьте в `backend/.env`:

```env
# Telegram (опционально)
TELEGRAM_BOT_TOKEN=your_bot_token_from_botfather
TELEGRAM_CHANNEL_ID=@your_channel_name
```

**Как получить токен:**
1. Найдите @BotFather в Telegram
2. Отправьте `/newbot`
3. Следуйте инструкциям
4. Скопируйте токен

Подробнее: `docs/TELEGRAM_SETUP.md`

### 3. Запустите сервисы

**Backend:**
```bash
cd backend
uvicorn main:app --reload
```

**Celery Worker (в отдельном терминале):**
```bash
cd backend
celery -A celery_app worker --loglevel=info
```

**Celery Beat для scheduled tasks (в отдельном терминале):**
```bash
cd backend
celery -A celery_app beat --loglevel=info
```

**Frontend:**
```bash
cd frontend
npm run dev
```

### 4. Доступные страницы

После входа в систему доступны:

- `/digest-settings` - Настройки дайджестов
- `/notifications` - Управление уведомлениями
- `/competitor-analysis` - Анализ конкурентов

### 5. Тестирование функций

#### Дайджесты:
1. Перейти на `/digest-settings`
2. Включить дайджесты
3. Настроить частоту и формат
4. (Опционально) Подключить Telegram
5. Сохранить настройки

#### Уведомления:
1. Нажать на иконку колокольчика в Header
2. Посмотреть список уведомлений
3. Отметить как прочитанные
4. Перейти на `/notifications` для полного списка

#### Анализ конкурентов:
1. Перейти на `/competitor-analysis`
2. Выбрать 2-5 компаний
3. Установить период
4. Нажать "Compare Companies"
5. Просмотреть результаты

## API Endpoints

Все новые endpoints доступны через `/api/v1`:

### Дайджесты
- `GET /api/v1/digest/daily` - дневной дайджест
- `GET /api/v1/digest/weekly` - недельный дайджест
- `GET /api/v1/digest/custom?start_date=...&end_date=...` - кастомный период
- `POST /api/v1/digest/generate?digest_type=daily` - генерация

### Настройки дайджестов
- `GET /api/v1/users/preferences/digest` - получить настройки
- `PUT /api/v1/users/preferences/digest` - обновить настройки

### Уведомления
- `GET /api/v1/notifications/` - список уведомлений
- `GET /api/v1/notifications/unread` - непрочитанные
- `PUT /api/v1/notifications/{id}/read` - отметить как прочитанное
- `PUT /api/v1/notifications/mark-all-read` - отметить все
- `DELETE /api/v1/notifications/{id}` - удалить
- `GET /api/v1/notifications/settings` - настройки уведомлений
- `PUT /api/v1/notifications/settings` - обновить настройки

### Анализ конкурентов
- `POST /api/v1/competitors/compare` - сравнить компании
- `GET /api/v1/competitors/comparisons` - список сравнений
- `GET /api/v1/competitors/comparisons/{id}` - детали сравнения
- `DELETE /api/v1/competitors/comparisons/{id}` - удалить
- `GET /api/v1/competitors/activity/{company_id}?days=30` - активность компании

## Автоматические задачи (Celery Beat)

При запущенном Celery Beat выполняются:

- **Каждый час:** Проверка пользователей с включенными дайджестами
- **Каждые 4 часа:** Проверка активности компаний → уведомления
- **Каждые 6 часов:** Проверка трендов категорий → уведомления
- **Ежедневно (полночь UTC):** Публикация дайджеста в Telegram канал
- **Ежедневно:** Очистка старых уведомлений (30+ дней)

## Troubleshooting

### Миграции не применяются
```bash
cd backend
alembic current  # Проверить текущую версию
alembic history  # Посмотреть историю
alembic upgrade head  # Применить все миграции
```

### Frontend не компилируется
```bash
cd frontend
npm install  # Установить зависимости
npm run dev  # Запустить
```

### Celery tasks не выполняются
Проверьте что Redis запущен:
```bash
docker ps | grep redis
# или
redis-cli ping  # должен вернуть PONG
```

### API возвращает 401 Unauthorized
- Убедитесь что вы авторизованы
- Проверьте что токен не истек
- Попробуйте перелогиниться

### Уведомления не приходят
- Проверьте что `NotificationSettings.enabled = true`
- Убедитесь что Celery worker запущен
- Проверьте логи Celery

## Документация

Полная документация:
- `docs/TELEGRAM_SETUP.md` - настройка Telegram
- `docs/FEATURES_GUIDE.md` - руководство пользователя
- `NEW_FEATURES_SUMMARY.md` - описание функций
- `IMPLEMENTATION_REPORT.md` - технический отчет

## Готово! 🎉

Все функции реализованы и готовы к использованию.

Если возникнут вопросы - проверьте документацию выше или логи сервисов.
