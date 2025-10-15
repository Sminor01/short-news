# ⚡ БЫСТРОЕ ИСПРАВЛЕНИЕ - 3 команды

## 🔴 Проблема: 500 Error при загрузке уведомлений

## ✅ Решение

Откройте PowerShell в директории проекта и выполните:

```powershell
# 1. Перейдите в backend
cd C:\Users\priah\Desktop\short-news\backend

# 2. Примените миграцию
python -m alembic upgrade head

# 3. Создайте настройки пользователей
python scripts\init_all_settings.py

# 4. Перезапустите backend
# Остановите текущий backend (Ctrl+C) и запустите снова:
python -m uvicorn main:app --reload
```

## ✅ Готово!

Обновите браузер (F5) - ошибка должна исчезнуть.

---

## 📋 Что это исправляет

- ✅ Создает таблицу `notifications`
- ✅ Создает таблицу `notification_settings`
- ✅ Создает таблицу `competitor_comparisons`
- ✅ Добавляет поля digest в `user_preferences`
- ✅ Создает настройки для всех пользователей

---

## 🔍 Проверка

После выполнения команд, в браузере:

1. Откройте http://localhost:5173
2. Колокольчик в Header должен работать (без ошибок)
3. Откройте http://localhost:5173/notifications (должна загрузиться)
4. Откройте http://localhost:5173/digest-settings (должна загрузиться)

**Не должно быть красных ошибок в консоли (F12)!**

---

## ⚠️ Если не помогло

См. подробную инструкцию: `FIX_NOTIFICATIONS_ERROR.md`



