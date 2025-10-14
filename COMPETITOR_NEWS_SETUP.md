# Настройка импорта новостей от конкурентов

## ✅ Выполненные задачи

### 1. Импорт компаний из CSV
- ✅ Импортировано **131 компания** из CSV файла
- ✅ Включены все GEO/LLM monitoring конкуренты из матрицы
- ✅ Автоматическое обнаружение и пропуск дубликатов

### 2. Система скрейпинга новостей
- ✅ Создан универсальный скрейпер для любых компаний
- ✅ Автоматическое определение blog/news URL
- ✅ Поддержка множества паттернов: `/blog`, `/news`, `/insights`, `/updates`, `/press`
- ✅ Работает с 9 основными источниками + универсальный скрейпер для остальных

### 3. База данных
- ✅ В базе **23 новости** от компаний
- ✅ Новости от: Anthropic (9), Hugging Face (9), Google (4), Cohere (1)
- ✅ Исправлены проблемы с PostgreSQL ENUM типами

### 4. Демо-пользователь
- ✅ Email: `demo@shot-news.com`
- ✅ Пароль: `demo123`
- ✅ Активирован и верифицирован

---

## 🚀 Как использовать

### Войти в систему

**URL**: http://localhost:5173

**Логин**: demo@shot-news.com  
**Пароль**: demo123

### Запуск скрейпинга новостей

```bash
# Скрейпинг от существующих компаний (9 AI-компаний)
docker-compose exec backend python scripts/populate_news.py

# Полный импорт: CSV компаний + скрейпинг их новостей
docker-compose exec backend python scripts/import_from_docker.py
```

### Проверка данных

```bash
# Количество компаний
docker exec shot-news-postgres psql -U shot_news -d shot_news -c "SELECT COUNT(*) FROM companies;"

# Количество новостей
docker exec shot-news-postgres psql -U shot_news -d shot_news -c "SELECT COUNT(*) FROM news_items;"

# Новости по компаниям
docker exec shot-news-postgres psql -U shot_news -d shot_news -c "SELECT c.name, COUNT(n.id) as news_count FROM companies c LEFT JOIN news_items n ON c.id = n.company_id GROUP BY c.name ORDER BY news_count DESC LIMIT 20;"
```

---

## 📊 Текущее состояние

### База данных
- **Компании**: 131
- **Новости**: 23
- **Пользователи**: 2 (admin + demo)

### Компании с новостями
| Компания | Новостей |
|----------|----------|
| Anthropic | 9 |
| Hugging Face | 9 |
| Google | 4 |
| Cohere | 1 |

### Доступные endpoint'ы
- `GET /api/v1/news/` - Получить список новостей
- `GET /api/v1/news/{id}` - Получить конкретную новость
- `GET /api/v1/news/search` - Поиск новостей
- `POST /api/v1/auth/login` - Авторизация

---

## 🔧 Исправленные проблемы

### 1. SQLAlchemy Enum vs PostgreSQL Enum
**Проблема**: SQLAlchemy создавал свои enum типы вместо использования существующих  
**Решение**: Использование нативных PostgreSQL ENUM с `create_type=False`

```python
source_type_enum = ENUM(
    'blog', 'twitter', 'github', 'reddit', 'news_site', 'press_release',
    name='source_type',
    create_type=False
)
```

### 2. Дубликаты компаний в CSV
**Проблема**: Компания "Gauge" встречается дважды в CSV  
**Решение**: Добавление компаний по одной с `flush()` и обработкой ошибок

### 3. Отсутствие некоторых компаний
**Проблема**: "Mistral AI" и "Stability AI" не в списке базовых компаний  
**Решение**: Нужно добавить их в `backend/app/tasks/seed_companies.py` или импортировать из CSV

---

## 🐛 Известные ограничения

1. **Не все сайты скрейпятся успешно**
   - Некоторые сайты имеют нестандартную структуру
   - Некоторые блокируют скрейперы
   - SSL ошибки на некоторых сайтах (например, Perplexity)

2. **Ограниченное количество статей**
   - По умолчанию скрейпер берет максимум 5 статей с каждого сайта
   - Это сделано для избежания блокировки

3. **Отсутствие актуальных дат публикации**
   - Скрейпер не всегда может извлечь реальную дату
   - Используются искусственные даты (текущая дата минус дни)

---

## 📝 Следующие шаги

### Для получения большего количества новостей:

1. **Запустить скрейпинг для всех 131 компаний**:
```bash
docker-compose exec backend python scripts/import_from_docker.py
```
Это займет 10-20 минут, так как скрейпер будет проверять ~131 сайт.

2. **Настроить автоматический скрейпинг через Celery Beat**
- Периодический скрейпинг каждые несколько часов
- Автоматическое добавление новых новостей

3. **Улучшить скрейпинг**
- Добавить поддержку RSS feeds
- Использовать API где возможно
- Добавить более точное извлечение дат

---

## 🎯 Подключение через DBeaver

**Параметры подключения:**
```
Host:     localhost
Port:     5432
Database: shot_news
Username: shot_news
Password: shot_news_dev
```

После подключения вы можете:
- Просматривать все 131 компанию
- Проверять импортированные новости
- Выполнять SQL-запросы для анализа

---

## 🌐 Доступ к приложению

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **Demo Login**:
  - Email: demo@shot-news.com
  - Password: demo123

---

## Созданные файлы

```
backend/
├── scripts/
│   ├── import_competitors_from_csv.py  # Импорт из CSV  
│   ├── scrape_all_companies.py          # Скрейпинг новостей
│   ├── import_from_docker.py             # Полный процесс (Docker)
│   ├── create_demo_user.py               # Создание демо-пользователя
│   └── README.md                          # Документация
├── app/
│   └── scrapers/
│       └── universal_scraper.py          # Универсальный скрейпер
```





