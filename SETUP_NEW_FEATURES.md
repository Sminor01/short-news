# Setup Guide: Новые функции - ПОШАГОВАЯ ИНСТРУКЦИЯ

## 🚨 Важно: Выполните ВСЕ шаги по порядку

---

## Шаг 1: Проверка окружения

Убедитесь что запущены:

```bash
# Проверьте PostgreSQL
docker ps | grep postgres

# Проверьте Redis  
docker ps | grep redis

# Если нет - запустите
docker-compose up -d postgres redis
```

---

## Шаг 2: Применение миграций

```bash
cd backend

# Вариант 1: Если установлен alembic глобально
alembic upgrade head

# Вариант 2: Через Python module
python -m alembic upgrade head

# Вариант 3: Через poetry
poetry run alembic upgrade head
```

**Ожидаемый вывод:**
```
INFO  [alembic.runtime.migration] Running upgrade b5037d3c878c -> c1d2e3f4g5h6, add_digest_and_notifications
```

**Если ошибка:**
- Проверьте `DATABASE_URL` в `.env`
- Проверьте что PostgreSQL запущен
- Проверьте что база данных существует

---

## Шаг 3: Инициализация настроек пользователей

**ВАЖНО:** Запустите этот скрипт для создания настроек существующим пользователям:

```bash
cd backend

# Вариант 1
python scripts/init_all_settings.py

# Вариант 2  
python -m scripts.init_all_settings

# Вариант 3
poetry run python scripts/init_all_settings.py
```

**Ожидаемый вывод:**
```
INFO - Created 1 user preferences
INFO - Created 1 notification settings
```

Этот скрипт:
- ✅ Создает `UserPreferences` если их нет
- ✅ Создает `NotificationSettings` если их нет
- ✅ Безопасен для повторного запуска

---

## Шаг 4: Запуск Backend

```bash
cd backend

# Вариант 1
uvicorn main:app --reload

# Вариант 2
python -m uvicorn main:app --reload

# Вариант 3
poetry run uvicorn main:app --reload
```

**Проверка:** Откройте http://localhost:8000/docs

Должны быть новые endpoints:
- ✅ `/api/v1/digest/daily`
- ✅ `/api/v1/digest/weekly`
- ✅ `/api/v1/notifications/`
- ✅ `/api/v1/competitors/compare`
- ✅ `/api/v1/users/preferences/digest`

---

## Шаг 5: Запуск Frontend

В **новом терминале**:

```bash
cd frontend

# Установите зависимости если нужно
npm install

# Запустите dev server
npm run dev
```

**Проверка:** Откройте http://localhost:5173

---

## Шаг 6: Тестирование функций

### 6.1 Войдите в систему

```
http://localhost:5173/login
```

Используйте существующего пользователя или создайте нового

### 6.2 Настройки дайджестов

```
http://localhost:5173/digest-settings
```

**Что должно работать:**
- ✅ Страница загружается
- ✅ Показываются текущие настройки
- ✅ Можно включить/выключить дайджесты
- ✅ Можно выбрать частоту (daily/weekly/custom)
- ✅ Можно выбрать формат (short/detailed)
- ✅ Кнопка "Save Settings" сохраняет изменения

**Если ошибка "Failed to load":**
- Проверьте консоль браузера (F12)
- Проверьте Network tab - какой status code?
- Если 404 - вернитесь к Шагу 3

### 6.3 Уведомления

**Notification Center в Header:**
- ✅ Должна быть иконка колокольчика
- ✅ При клике открывается dropdown
- ✅ Показывается "No new notifications" (если нет уведомлений)

**Страница уведомлений:**
```
http://localhost:5173/notifications
```

- ✅ Страница загружается
- ✅ Показывается список уведомлений (или "No notifications")

### 6.4 Анализ конкурентов

```
http://localhost:5173/competitor-analysis
```

**Что должно работать:**
- ✅ Страница загружается
- ✅ Dropdown с компаниями работает
- ✅ Можно выбрать 2-5 компаний
- ✅ Можно выбрать даты
- ✅ Кнопка "Compare Companies" запускает анализ

**Тест:**
1. Выберите 2 компании (например OpenAI и Anthropic)
2. Нажмите "Compare Companies"
3. Должны появиться графики и таблицы

---

## Шаг 7: (Опционально) Celery для автоматизации

Для автоматической генерации дайджестов и уведомлений:

**Terminal 3 - Celery Worker:**
```bash
cd backend
celery -A celery_app worker --loglevel=info
# или
python -m celery -A celery_app worker --loglevel=info
```

**Terminal 4 - Celery Beat:**
```bash
cd backend
celery -A celery_app beat --loglevel=info
# или
python -m celery -A celery_app beat --loglevel=info
```

**Проверка:** В логах Celery Beat должно быть:
```
beat: Starting...
Scheduler: Sending due task generate-daily-digests
Scheduler: Sending due task check-company-activity
```

---

## Шаг 8: (Опционально) Telegram

### 8.1 Создайте бота

1. Найдите @BotFather в Telegram
2. Отправьте `/newbot`
3. Следуйте инструкциям
4. Скопируйте токен

### 8.2 Добавьте в .env

```env
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
TELEGRAM_CHANNEL_ID=@your_channel  # опционально
```

### 8.3 Перезапустите backend

```bash
# Ctrl+C в терминале backend
# Затем снова:
uvicorn main:app --reload
```

### 8.4 Протестируйте

1. Найдите вашего бота в Telegram
2. Отправьте `/start`
3. Скопируйте Chat ID
4. Добавьте в `/digest-settings`
5. Включите "Send to Telegram"
6. Сохраните

---

## Проверочный список ✅

После выполнения всех шагов:

- [ ] Миграции применены (`alembic upgrade head`)
- [ ] UserPreferences созданы (`python scripts/init_all_settings.py`)
- [ ] Backend запущен (http://localhost:8000/docs работает)
- [ ] Frontend запущен (http://localhost:5173 работает)
- [ ] Можете войти в систему
- [ ] `/digest-settings` загружается и сохраняет настройки
- [ ] `/notifications` загружается
- [ ] `/competitor-analysis` загружается и работает
- [ ] NotificationCenter виден в Header

---

## Типичные ошибки и решения

### ❌ Миграция не применяется

**Ошибка:** `Target database is not up to date`

**Решение:**
```bash
cd backend
alembic stamp head  # Сбросить версию
alembic upgrade head  # Применить снова
```

### ❌ 404 при GET /api/v1/users/preferences/digest

**Причина:** У пользователя нет preferences

**Решение:**
```bash
python scripts/init_all_settings.py
```

### ❌ 500 Internal Server Error

**Причина:** Ошибка в backend

**Решение:**
1. Проверьте логи uvicorn
2. Проверьте что все модели импортированы
3. Проверьте что миграции применены

### ❌ Frontend не компилируется

**Причина:** Ошибки TypeScript

**Решение:**
```bash
cd frontend
npm install  # Переустановите зависимости
```

### ❌ CORS error

**Причина:** Backend не разрешает запросы с frontend

**Проверьте:** `backend/app/core/config.py`
```python
ALLOWED_HOSTS: List[str] = [
    "http://localhost:3000",
    "http://localhost:5173",  # <-- должно быть
]
```

---

## Минимальная рабочая конфигурация

Для тестирования достаточно:

1. ✅ PostgreSQL + Redis (Docker)
2. ✅ Backend (uvicorn)
3. ✅ Frontend (npm run dev)
4. ✅ Миграции применены
5. ✅ init_all_settings.py выполнен

**Celery и Telegram - опциональны для первого теста!**

---

## Готово! 🎉

После выполнения всех шагов попробуйте:

1. Зайти на http://localhost:5173/digest-settings
2. Включить дайджесты
3. Сохранить
4. Зайти на http://localhost:5173/competitor-analysis
5. Сравнить 2 компании

Если что-то не работает:
- Проверьте консоль браузера (F12)
- Проверьте логи backend
- Проверьте `TROUBLESHOOTING_DIGESTS.md`

