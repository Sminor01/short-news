# Quick Start: Дайджесты и Уведомления

## 🚀 Быстрый запуск (3 команды)

### Windows (PowerShell):
```powershell
.\setup_features.ps1
cd backend; python -m uvicorn main:app --reload
# В новом терминале: cd frontend; npm run dev
```

### Linux/Mac:
```bash
chmod +x setup_features.sh
./setup_features.sh
cd backend && uvicorn main:app --reload &
cd frontend && npm run dev
```

---

## 📝 Что делает скрипт

1. ✅ Применяет миграции БД
2. ✅ Создает UserPreferences для всех пользователей
3. ✅ Создает NotificationSettings для всех пользователей

---

## 🎯 После запуска

### Откройте в браузере:

```
http://localhost:5173
```

### Войдите в систему

Используйте существующий аккаунт или зарегистрируйтесь

### Доступные страницы:

- **`/digest-settings`** - Настройки дайджестов
  - Включить/выключить дайджесты
  - Выбрать частоту (daily/weekly/custom)
  - Выбрать формат (short/detailed)
  - Подключить Telegram (опционально)

- **`/notifications`** - Уведомления
  - Список всех уведомлений
  - Фильтр (all/unread)
  - Управление (mark as read, delete)

- **`/competitor-analysis`** - Анализ конкурентов
  - Выбрать 2-5 компаний
  - Выбрать период
  - Сравнить по метрикам

### Колокольчик в Header

- Клик → показывает последние уведомления
- Badge показывает количество непрочитанных

---

## ❗ Если что-то не работает

### 1. Страница не загружается

**Проверьте консоль браузера (F12):**
- Какая ошибка?
- Какой HTTP status code?

### 2. "Failed to load digest settings"

**Решение:**
```bash
cd backend
python scripts/init_all_settings.py
```

Этот скрипт создает settings для пользователей

### 3. 401 Unauthorized

**Решение:** Перелогиньтесь
```
http://localhost:5173/login
```

### 4. Backend не запускается

**Проверьте:**
```bash
# PostgreSQL запущен?
docker ps | grep postgres

# Redis запущен?
docker ps | grep redis

# Если нет:
docker-compose up -d postgres redis
```

---

## 🔍 Быстрая диагностика

### Test API напрямую:

```bash
# Получите токен после логина
TOKEN="your_access_token_here"

# Test digest endpoint
curl -H "Authorization: Bearer $TOKEN" http://localhost:8000/api/v1/users/preferences/digest

# Если 200 - работает!
# Если 404 - запустите init_all_settings.py
# Если 401 - токен неверный
```

---

## ✅ Полная документация

Если нужны детали:

- **`SETUP_NEW_FEATURES.md`** - Подробная настройка
- **`TROUBLESHOOTING_DIGESTS.md`** - Решение проблем
- **`docs/FEATURES_GUIDE.md`** - Руководство пользователя
- **`docs/TELEGRAM_SETUP.md`** - Настройка Telegram бота

---

## 🎉 Готово!

После выполнения скрипта и запуска сервисов:

1. ✅ Зайдите на `/digest-settings`
2. ✅ Настройте дайджест
3. ✅ Сохраните
4. ✅ Проверьте уведомления
5. ✅ Сравните конкурентов

**Всё должно работать!**

Если возникли проблемы - смотрите `TROUBLESHOOTING_DIGESTS.md`

