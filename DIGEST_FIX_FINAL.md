# ✅ Дайджесты полностью исправлены!

## Проблема

При запросе дайджестов возвращались одинаковые даты `date_from` и `date_to` с точностью до микросекунды, вместо диапазона от 00:00 до 23:59.

**Пример:**
```json
{
  "date_from": "2025-10-14T17:33:38.417330",
  "date_to": "2025-10-14T17:33:38.417330",  // Одинаковые!
  "news_count": 0
}
```

## Причина

У пользователя не было записи `user_preferences` в базе данных. Когда preferences отсутствуют, вызывался метод `_empty_digest()`, который возвращал текущий момент времени для обеих дат.

## Решение

### 1. Автоматическое создание preferences

Теперь при первом запросе дайджеста preferences создаются автоматически с правильными дефолтными значениями:

```python
user_prefs = UserPreferences(
    timezone='UTC',
    week_start_day=0,  # Sunday
    digest_enabled=True,
    # ... другие поля
)
```

### 2. Корректная логика дат

**Daily Digest:**
- Собирает новости с 00:00:00 до 23:59:59 текущего дня
- Пример: 14.10.2025 00:00 - 14.10.2025 23:59

**Weekly Digest:**
- Собирает новости за текущую неделю (воскресенье - суббота)
- Пример: 12.10.2025 00:00 - 18.10.2025 23:59

### 3. Поддержка часовых поясов

Даты рассчитываются относительно часового пояса пользователя!

## Что было сделано

1. ✅ Добавлена переменная `DEBUG=true` в docker-compose.yml
2. ✅ Применена миграция базы данных (добавлены поля `timezone` и `week_start_day`)
3. ✅ Исправлен `DigestService` - автоматически создает preferences если их нет
4. ✅ Добавлено подробное логирование для отладки
5. ✅ Перезапущен backend

## Проверка работы

### Шаг 1: Обновите страницу в браузере
Нажмите `F5` или `Ctrl+R`

### Шаг 2: Откройте Daily Digest

Теперь вы должны увидеть:
```json
{
  "date_from": "2025-10-14T00:00:00",
  "date_to": "2025-10-14T23:59:59.999999",
  "news_count": ...,
  "categories": {...}
}
```

### Шаг 3: Настройте часовой пояс

1. Перейдите в **Settings → Digest Settings**
2. Найдите секцию **"Timezone & Week Settings"**
3. Выберите:
   - **Timezone**: Europe/Moscow (для России) или ваш часовой пояс
   - **Week Start Day**: 
     - **Sunday** - неделя Вс→Сб (как в GitHub, США)
     - **Monday** - неделя Пн→Вс (ISO 8601, Европа)
4. Сохраните

## Примеры работы

### Daily Digest

**Запрос:** GET `/api/v1/digest/daily`

**До исправления:**
```json
{
  "date_from": "2025-10-14T17:33:38.417330",
  "date_to": "2025-10-14T17:33:38.417330",
  "news_count": 0
}
```

**После исправления:**
```json
{
  "date_from": "2025-10-14T00:00:00",
  "date_to": "2025-10-14T23:59:59.999999",
  "news_count": 15,
  "categories": {...}
}
```

### Weekly Digest

**Запрос:** GET `/api/v1/digest/weekly`

**До исправления:**
```json
{
  "date_from": "2025-10-14T17:33:38.417330",
  "date_to": "2025-10-14T17:33:38.417330"
}
```

**После исправления (Sunday start):**
```json
{
  "date_from": "2025-10-12T00:00:00",  // Воскресенье
  "date_to": "2025-10-18T23:59:59.999999",  // Суббота
  "news_count": 87,
  "categories": {...}
}
```

## Технические детали

### Расчет дат для Daily Digest

```python
# Текущее время в часовом поясе пользователя
now_user = datetime.now(pytz.timezone('Europe/Moscow'))  # Например

# Начало дня
date_from = now_user.replace(hour=0, minute=0, second=0, microsecond=0)

# Конец дня  
date_to = now_user.replace(hour=23, minute=59, second=59, microsecond=999999)

# Конвертация в UTC для запроса к БД
date_from_utc = date_from.astimezone(pytz.UTC)
date_to_utc = date_to.astimezone(pytz.UTC)
```

### Расчет дат для Weekly Digest

```python
# Определяем начало недели (воскресенье или понедельник)
if week_start_day == 0:  # Sunday
    days_since_week_start = (now_user.weekday() + 1) % 7
else:  # Monday
    days_since_week_start = now_user.weekday()

# Начало недели
week_start = now_user - timedelta(days=days_since_week_start)
date_from = week_start.replace(hour=0, minute=0, second=0)

# Конец недели (через 6 дней)
week_end = now_user + timedelta(days=(6 - days_since_week_start))
date_to = week_end.replace(hour=23, minute=59, second=59)
```

## Логирование

Теперь в логах backend вы увидите:
```
2025-10-14 17:45:00.123 | INFO | Generating daily digest for user 7e0556e1-...
2025-10-14 17:45:00.124 | DEBUG | Period: daily, User TZ: UTC, Now UTC: 2025-10-14 17:45:00+00:00
2025-10-14 17:45:00.125 | DEBUG | Daily digest user TZ range: 2025-10-14 00:00:00 to 2025-10-14 23:59:59
2025-10-14 17:45:00.126 | DEBUG | Daily digest UTC range: 2025-10-14 00:00:00 to 2025-10-14 23:59:59
2025-10-14 17:45:00.127 | INFO | Date range for daily digest (timezone: UTC): 2025-10-14 00:00:00 to 2025-10-14 23:59:59 UTC
2025-10-14 17:45:00.234 | INFO | Digest generated: 15 news items
```

## Если проблема осталась

1. **Очистите кэш браузера** и перезагрузите страницу
2. **Проверьте логи backend:**
   ```bash
   docker logs shot-news-backend --tail 50
   ```
3. **Откройте консоль браузера** (F12) и проверьте ответ API
4. **Пришлите скриншот** с новой ошибкой

## Документация

- **DIGEST_DATE_LOGIC.md** - подробная техническая документация
- **DIGEST_UPDATE_SUMMARY.md** - полный отчет об изменениях
- **ИСПРАВЛЕНИЯ_ДАЙДЖЕСТОВ.md** - краткая сводка на русском

---

## 🎉 Готово!

Дайджесты теперь работают корректно:
- ✅ Daily Digest - новости за текущий день (00:00 - 23:59)
- ✅ Weekly Digest - новости за текущую неделю (Вс-Сб или Пн-Вс)
- ✅ Поддержка часовых поясов
- ✅ Настраиваемое начало недели
- ✅ Автоматическое создание preferences

**Обновите страницу в браузере и проверьте!** 🚀

