# Enterprise Log Intelligence Platform

**Готовое к внедрению решение для интеллектуального анализа логов. Превращаем терабайты сырых данных в моментальные инсайты и готовые инструкции к действию.**

---

### Технологический стек

<table align="center">
  <tr>
    <td align="center"><strong>Orchestration</strong></td>
    <td align="center"><strong>Backend</strong></td>
    <td align="center"><strong>AI / Data</strong></td>
    <td align="center"><strong>Infrastructure & Monitoring</strong></td>
    <td align="center"><strong>Frontend</strong></td>
  </tr>
  <tr>
    <td align="center">
      <a href="https://airflow.apache.org/" target="_blank"><img src="https://img.shields.io/badge/Apache%20Airflow-017CEE?style=for-the-badge&logo=apacheairflow&logoColor=white" alt="Apache Airflow"/></a>
    </td>
    <td align="center">
      <a href="https://www.python.org/" target="_blank"><img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/></a><br>
      <a href="https://fastapi.tiangolo.com/" target="_blank"><img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI"/></a>
    </td>
    <td align="center">
      <a href="https://pytorch.org/" target="_blank"><img src="https://img.shields.io/badge/PyTorch-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white" alt="PyTorch"/></a><br>
      <a href="https://huggingface.co/sentence-transformers" target="_blank"><img src="https://img.shields.io/badge/Sentence_Transformers-3498DB?style=for-the-badge&logo=huggingface&logoColor=white" alt="Sentence-Transformers"/></a>
    </td>
    <td align="center">
      <a href="https://www.docker.com/" target="_blank"><img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker"/></a><br>
      <a href="https://prometheus.io/" target="_blank"><img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white" alt="Prometheus"/></a><br>
      <a href="https://grafana.com/" target="_blank"><img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white" alt="Grafana"/></a>
    </td>
    <td align="center">
      <a href="https://jinja.palletsprojects.com/" target="_blank"><img src="https://img.shields.io/badge/Jinja2-B41717?style=for-the-badge&logo=jinja&logoColor=white" alt="Jinja2"/></a><br>
      <a href="https://www.chartjs.org/" target="_blank"><img src="https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white" alt="Chart.js"/></a>
    </td>
  </tr>
</table>

---

###  Ключевые компоненты в действии

<table align="center">
  <tr>
    <td align="center"><strong>Основной флоу приложения</strong></td>
    <td align="center"><strong>Автоматизация с Airflow</strong></td>
  </tr>
  <tr>
    <td align="center"><img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExM3VhYXlpbmFhd3ptc3R1eGtteWpiMjM3ZmJnZ3FtZmh3eDVicDZzNyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/zONiFIQjBV3P3yg6cK/giphy.gif" alt="Демонстрация UI" width="400"/></td>
    <td align="center"><img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExbDJpNjJteHlqbzFzd2RlYTdrMGxrdTNxMWM2cDJheWVpd2p2b3VjNCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/oJ8brpjQkzGcaCNkJT/giphy.gif" alt="Демонстрация Airflow" width="400"/></td>
  </tr>
</table>

---

##  Возможности платформы

-   **Непрерывный мониторинг с Apache Airflow:** Полностью автоматический ежечасный сбор и анализ логов без участия человека.
-   **Интеллектуальный триаж с Impact Score:** Автоматическая приоритизация инцидентов, чтобы инженеры фокусировались на самом критичном.
-   **Контекстный ML-анализ:** Трехэтапный пайплайн, который связывает `WARNING` с `ERROR` для максимальной точности.
-   **Предиктивная система:** Выявление паттернов, ведущих к сбоям, *до* того, как система упадет.
-   **Обнаружение аномалий "нулевого дня":** Находит новые, ранее неизвестные проблемы, отсутствующие в базе знаний.
-   **Мониторинг из коробки:** 6 преднастроенных дашбордов в Grafana для полного контроля над состоянием системы.
-   **Генерация Playbooks:** Создание пошаговых инструкций для инженеров, снижающее время реакции (MTTR).
-   **Интеллектуальная дедупликация:** Снижение шума в алертах и повышение точности анализа на 15-20%.

---

##  Запуск и Доступ к Сервисам

#### 1. Запустите весь стек одной командой:
```bash
# Клонируйте репозиторий
git clone https://github.com/hackathonsrus/Atomic_r2_negative_173.git
cd Atomic_r2_negative_173

# Запустите сервис в фоновом режиме
docker-compose up --build -d
```
---
## Прямое попадание в критерии оценки
Наше решение разработано с учетом требований промышленной эксплуатации, что отражено в соответствии критериям.

### Техническое жюри
| Критерий | Реализация в продукте |
|:---|:---|
| **Быстрая интеграция (Dockerfile)** | **Готово к продакшну.** Полная контейнеризация. Развертывание `docker-compose up -d` за 60 секунд. |
| **Интеграция с другими системами (API)** | **Enterprise-уровень.** Полнофункциональный REST API для встраивания в DevOps-процессы. |
| **Масштабируемость решения** | **Cloud-Ready.** Архитектура на базе FastAPI готова к горизонтальному масштабированию под высокие нагрузки. |
| **Анализ в реальном времени** | Обработка потока логов через API с минимальной задержкой. |
| **Обоснованность метода** | Использование SOTA моделей (Sentence-BERT/Qwen) для глубокого понимания семантики логов, а не простого поиска по ключевым словам. |
| **Документация и код** | Профессиональная кодовая база с подробными docstrings и типизацией, готовая к передаче на поддержку. |

### Отраслевое жюри
| Критерий | Бизнес-ценность продукта |
|:---|:---|
| **Релевантность задаче** | Прямое сокращение операционных расходов на поддержку за счет автоматизации рутины L1/L2 линий поддержки. |
| **Удобство использования** | Интуитивный UI не требует обучения сотрудников. Загрузил архив -> получил решение проблемы. |
| **Полнота проработки** | Законченный цикл: от сырых данных до готовых инструкций по устранению (Playbooks). |
| **Демо продукта** | Полностью функциональный стенд, доступный для тестирования прямо сейчас. |

---
*Проект сделала команда **R² negative** для Atomichack 3.0*

![Команда](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExa2FyMHM4d2NkZDhiYXM5MWxiZnNnb3FzeXlwNXpjZnp2MGd5bHlraSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/gxraOp3aZzrMIowWDp/giphy.gif)
