# 📊 Руководство по мониторингу FastAPI Log Analyzer

## 🎯 Обзор

Ваш FastAPI проект теперь оснащен полноценной системой мониторинга на базе **Prometheus** и **Grafana**. Это позволяет отслеживать производительность приложения, ML-анализа и использование ресурсов в реальном времени.

---

## 📦 Установленные компоненты

### 1. **Prometheus FastAPI Instrumentator**
Автоматически собирает метрики FastAPI:
- HTTP запросы (RPS)
- Latency (время отклика)
- Статусы ответов (2xx, 4xx, 5xx)
- Запросы в обработке

### 2. **Кастомные метрики**
Специализированные метрики для анализа логов:
- `log_analysis_duration_seconds` - время обработки логов
- `log_records_processed_total` - количество обработанных записей
- `ml_model_inference_duration_seconds` - время ML инференса
- `memory_usage_bytes` - использование памяти (RSS, VMS)
- `anomalies_detected_total` - обнаруженные аномалии
- `problems_classified_total` - классифицированные проблемы
- `active_websocket_connections` - активные WebSocket соединения
- `zip_archives_processed_total` - обработанные архивы
- `ml_model_loading_duration_seconds` - время загрузки ML модели
- `zip_archive_size_bytes` - размер архивов

### 3. **Prometheus**
- Версия: 2.48.0
- Порт: 9090
- Хранение метрик: 30 дней
- Интервал сбора: 10 секунд

### 4. **Grafana**
- Версия: 10.2.2
- Порт: 3000
- Логин: `admin`
- Пароль: `admin`
- Автоматический datasource: Prometheus
- Преднастроенный дашборд

---

## 🚀 Быстрый старт

### Шаг 1: Установка зависимостей

```bash
pip install -r requirements.txt
```

Установятся:
- `prometheus-fastapi-instrumentator==6.1.0`
- `prometheus-client==0.19.0`
- `psutil==5.9.6`

### Шаг 2: Запуск с Docker Compose

```bash
docker-compose up -d
```

Это запустит 3 контейнера:
1. `fastapi-app` - основное приложение (порт 8001)
2. `prometheus` - сбор метрик (порт 9090)
3. `grafana` - визуализация (порт 3000)

### Шаг 3: Проверка работы

#### 3.1. FastAPI метрики
Откройте в браузере:
```
http://localhost:8001/metrics
```

Вы должны увидеть метрики в формате Prometheus:
```
# HELP log_records_processed_total Общее количество обработанных записей логов
# TYPE log_records_processed_total counter
log_records_processed_total{model_type="light",status="success"} 1523.0
...
```

#### 3.2. Prometheus Targets
Откройте:
```
http://localhost:9090/targets
```

Убедитесь, что `fastapi-log-analyzer` имеет статус **UP** (зеленая галочка).

#### 3.3. Grafana Dashboard
Откройте:
```
http://localhost:3000
```

**Логин:** `admin`  
**Пароль:** `admin`

Дашборд **"FastAPI Log Analyzer - Мониторинг"** должен быть доступен автоматически.

---

## 📊 Дашборд Grafana

### Секция 1: 📊 Общая статистика
- **Всего запросов** - общее количество HTTP запросов
- **ZIP архивы** - pie chart (Success/Error)
- **Обработано записей** - количество обработанных логов
- **WebSocket соединения** - активные соединения

### Секция 2: 🚀 Производительность API
- **Request Rate (RPS)** - запросы в секунду
- **Request Latency** - время отклика (p50, p95, p99)

### Секция 3: 🤖 ML Анализ и Обработка
- **Время анализа логов** - duration для light/heavy моделей
- **ML Inference Duration** - время инференса по этапам
- **Обнаруженные аномалии** - stacked graph по severity
- **Классифицированные проблемы** - по типам проблем

### Секция 4: 💾 Ресурсы и память
- **Использование памяти** - RSS и VMS
- **Размер ZIP архивов** - распределение размеров

---

## 🔧 Интеграция метрик в код

### В main.py
```python
import metrics

# При обработке файла
metrics.record_zip_processed(model, 'received', len(file_content))

# После успешной обработки
metrics.record_log_analysis(model, 'success', duration, total_records)
metrics.update_memory_metrics()

# При изменении WebSocket соединений
metrics.update_websocket_count(len(active_websockets))
```

### В processing/orchestrator.py
```python
import metrics

# Загрузка модели
model_load_start = time.time()
model = SentenceTransformer(model_name, device=device)
model_load_duration = time.time() - model_load_start
metrics.record_model_loading(model_choice, model_load_duration)

# ML инференс
classification_start = time.time()
classified_logs = run_analysis_pipeline(...)
classification_duration = time.time() - classification_start
metrics.record_ml_inference(model_choice, 'full_classification', classification_duration)

# Обнаруженные аномалии
metrics.record_anomalies_detected(model_choice, final_count, 'medium')

# Классифицированные проблемы
metrics.record_problems_classified(model_choice, unique_problems, 'generic')
```

---

## 📈 Prometheus Queries (примеры)

### 1. Request Rate за последние 5 минут
```promql
rate(http_requests_total[5m])
```

### 2. P95 latency
```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### 3. Среднее время анализа логов
```promql
rate(log_analysis_duration_seconds_sum[5m]) / rate(log_analysis_duration_seconds_count[5m])
```

### 4. Общее количество обработанных записей
```promql
sum(log_records_processed_total)
```

### 5. Использование памяти
```promql
memory_usage_bytes{type="rss"}
```

### 6. Error rate (%)
```promql
(sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) * 100
```

---

## 🎨 Кастомизация дашборда

### Добавление новой панели
1. Откройте дашборд в Grafana
2. Нажмите **"Add" → "Visualization"**
3. Выберите datasource: **Prometheus**
4. Введите PromQL запрос
5. Настройте визуализацию (тип графика, цвета, легенду)
6. Нажмите **"Apply"**

### Экспорт дашборда
1. Откройте дашборд
2. Нажмите на иконку ⚙️ (Settings)
3. Выберите **"JSON Model"**
4. Скопируйте JSON и сохраните в `grafana/dashboards/`

---

## 🐛 Troubleshooting

### Проблема: Prometheus не видит FastAPI target

**Решение:**
1. Проверьте, что все контейнеры запущены:
   ```bash
   docker-compose ps
   ```
2. Проверьте сеть:
   ```bash
   docker network inspect fastapigitproject_monitoring
   ```
3. Проверьте логи Prometheus:
   ```bash
   docker logs prometheus
   ```

### Проблема: Метрики не отображаются в Grafana

**Решение:**
1. Проверьте datasource в Grafana: **Configuration → Data Sources → Prometheus**
2. Нажмите **"Test"** - должно быть "Data source is working"
3. Проверьте, что в Prometheus есть данные: `http://localhost:9090/graph`

### Проблема: High memory usage

**Решение:**
1. Уменьшите retention time в `prometheus.yml`:
   ```yaml
   --storage.tsdb.retention.time=7d  # Вместо 30d
   ```
2. Перезапустите:
   ```bash
   docker-compose restart prometheus
   ```

---

## 📚 Дополнительные ресурсы

- [Prometheus Docs](https://prometheus.io/docs/)
- [Grafana Docs](https://grafana.com/docs/)
- [FastAPI Instrumentator](https://github.com/trallnag/prometheus-fastapi-instrumentator)
- [PromQL Tutorial](https://prometheus.io/docs/prometheus/latest/querying/basics/)

---

## 🔐 Безопасность в Production

### 1. Изменить пароль Grafana
В `docker-compose.yml`:
```yaml
environment:
  - GF_SECURITY_ADMIN_PASSWORD=STRONG_PASSWORD_HERE
```

### 2. Ограничить доступ к метрикам
В `main.py`:
```python
from fastapi import Security, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

@app.get("/metrics")
async def metrics_endpoint(credentials: str = Security(security)):
    # Проверка токена
    if credentials.credentials != "SECRET_TOKEN":
        raise HTTPException(status_code=403)
    # Вернуть метрики
```

### 3. HTTPS для Grafana
Используйте Nginx или Traefik как reverse proxy с SSL сертификатами.

---

## ⚡ Performance Tips

### 1. Оптимизация сбора метрик
- Увеличьте `scrape_interval` для редко меняющихся метрик
- Используйте `scrape_timeout` меньше `scrape_interval`

### 2. Оптимизация хранения
- Используйте remote storage для больших объемов данных
- Настройте агрегацию через recording rules

### 3. Grafana
- Используйте переменные (variables) в дашбордах
- Кешируйте запросы с помощью query caching
- Ограничьте временной диапазон при большом объеме данных

---

## 🎉 Готово!

Ваша система мониторинга настроена и готова к использованию. Загрузите тестовый ZIP файл и наблюдайте за метриками в реальном времени!

**Основные URL:**
- FastAPI App: http://localhost:8001
- FastAPI Metrics: http://localhost:8001/metrics
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000 (admin/admin)

**Файлы конфигурации:**
- `prometheus.yml` - конфигурация Prometheus
- `docker-compose.yml` - оркестрация контейнеров
- `metrics.py` - кастомные метрики
- `grafana/provisioning/` - автоматическая настройка Grafana
- `grafana/dashboards/logs-analysis.json` - преднастроенный дашборд

Удачного мониторинга! 🚀


