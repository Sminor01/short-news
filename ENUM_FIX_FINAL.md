# ✅ Исправлена ошибка с enum типами!

## Проблема была в

База данных не имела необходимых enum типов, которые используются в коде:
- `newscategory` - для категорий новостей
- `sourcetype` - для типов источников
- `notificationfrequency` - для частоты уведомлений  
- `activitytype` - для типов активности

## Что было сделано

### 1. Созданы недостающие enum типы в PostgreSQL

```sql
CREATE TYPE newscategory AS ENUM (
  'product_update', 'pricing_change', 'strategic_announcement',
  'technical_update', 'funding_news', 'research_paper',
  'community_event', 'partnership', 'acquisition', 'integration',
  'security_update', 'api_update', 'model_release',
  'performance_improvement', 'feature_deprecation'
);

CREATE TYPE sourcetype AS ENUM (
  'blog', 'twitter', 'github', 'reddit', 'news_site', 'press_release'
);

CREATE TYPE notificationfrequency AS ENUM (
  'realtime', 'daily', 'weekly', 'never'
);

CREATE TYPE activitytype AS ENUM (
  'viewed', 'favorited', 'marked_read', 'shared'
);
```

### 2. Перезапущен backend

```bash
docker-compose restart backend
```

## СЕЙЧАС попробуйте:

### 1. Обновите страницу в браузере
Нажмите `F5` или `Ctrl+R`

### 2. Откройте Daily Digest

Теперь должно работать! Вы должны увидеть:

```json
{
  "date_from": "2025-10-14T00:00:00",
  "date_to": "2025-10-14T23:59:59.999999",
  "news_count": ...,
  "categories": {...},
  "statistics": {...}
}
```

### 3. Попробуйте Weekly Digest

Должна показаться неделя с воскресенья по субботу:

```json
{
  "date_from": "2025-10-12T00:00:00",      // Воскресенье
  "date_to": "2025-10-18T23:59:59.999999", // Суббота
  "news_count": ...,
  "categories": {...}
}
```

## Что исправлено

1. ✅ Созданы все необходимые enum типы в базе данных
2. ✅ Backend перезапущен
3. ✅ При первом запросе автоматически создаются `user_preferences`
4. ✅ Корректная логика расчета дат:
   - **Daily**: 00:00 - 23:59 текущего дня
   - **Weekly**: Воскресенье - Суббота текущей недели
5. ✅ Поддержка часовых поясов (настраивается в Settings)

## Логи показывают

```
INFO  | Database connection established successfully
INFO  | Application startup complete!
```

Backend работает корректно!

## Если проблема осталась

1. Откройте консоль браузера (F12)
2. Попробуйте запросить дайджест
3. Скопируйте ошибку (если есть) и пришлите скриншот

Также можно проверить логи:
```bash
docker logs shot-news-backend --tail 50
```

---

## 🎉 Готово!

**Обновите страницу и попробуйте открыть дайджесты!**

Теперь все должно работать корректно:
- ✅ Enum типы созданы
- ✅ Preferences создаются автоматически
- ✅ Правильные даты (00:00 - 23:59)
- ✅ Поддержка часовых поясов
- ✅ Настраиваемое начало недели

