# 🚀 Полный Гайд: Развертывание FastAPI Проекта на Сервере

## 📋 Содержание
1. [Выбор и подготовка сервера](#1-выбор-и-подготовка-сервера)
2. [Подключение к серверу](#2-подключение-к-серверу)
3. [Установка необходимого ПО](#3-установка-необходимого-по)
4. [Загрузка проекта на сервер](#4-загрузка-проекта-на-сервер)
5. [Настройка окружения](#5-настройка-окружения)
6. [Настройка Gunicorn/Uvicorn](#6-настройка-gunicornuvicorn)
7. [Настройка Nginx](#7-настройка-nginx)
8. [Настройка SSL (HTTPS)](#8-настройка-ssl-https)
9. [Автозапуск через systemd](#9-автозапуск-через-systemd)
10. [Мониторинг и логи](#10-мониторинг-и-логи)

---

## 1. Выбор и Подготовка Сервера

### Рекомендуемые хостинги:
- **VPS/VDS:** DigitalOcean, Linode, Hetzner, AWS EC2, Google Cloud, Azure
- **Российские:** Timeweb, Beget, REG.RU, Selectel
- **Минимальные требования:** 2 GB RAM, 2 CPU, 20 GB SSD

### Рекомендуемая конфигурация:
```
OS: Ubuntu 22.04 LTS (или 20.04 LTS)
RAM: 4 GB
CPU: 2 vCPU
SSD: 40 GB
```

---

## 2. Подключение к Серверу

### Для Linux/Mac:
```bash
ssh root@ВАШ_IP_АДРЕС
# Введите пароль или используйте SSH ключ
```

### Для Windows:
1. **Через PowerShell:**
```powershell
ssh root@ВАШ_IP_АДРЕС
```

2. **Через PuTTY:**
   - Скачайте PuTTY: https://www.putty.org/
   - Введите IP адрес сервера
   - Нажмите "Open"

### Создание нового пользователя (рекомендуется):
```bash
# Создаем пользователя
adduser fastapi_user

# Добавляем в группу sudo
usermod -aG sudo fastapi_user

# Переключаемся на нового пользователя
su - fastapi_user
```

---

## 3. Установка Необходимого ПО

### 3.1. Обновление системы:
```bash
sudo apt update
sudo apt upgrade -y
```

### 3.2. Установка Python 3.10+:
```bash
# Установка Python 3.11
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.11 python3.11-venv python3.11-dev -y

# Проверка версии
python3.11 --version
```

### 3.3. Установка pip:
```bash
sudo apt install python3-pip -y
pip3 --version
```

### 3.4. Установка дополнительных пакетов:
```bash
sudo apt install -y \
    git \
    nginx \
    supervisor \
    ufw \
    curl \
    wget \
    build-essential \
    libpq-dev
```

---

## 4. Загрузка Проекта на Сервер

### Способ 1: Через Git (Рекомендуется)
```bash
# Переходим в домашнюю директорию
cd ~

# Клонируем репозиторий
git clone https://github.com/ВАШ_ПОЛЬЗОВАТЕЛЬ/FASTAPIGITPROJECT.git

# Переходим в директорию проекта
cd FASTAPIGITPROJECT
```

### Способ 2: Через SCP (если проект не на GitHub)
```bash
# На вашем локальном компьютере (Windows PowerShell):
scp -r D:\Downloads\FASTAPIGITPROJECT root@ВАШ_IP:/home/fastapi_user/
```

### Способ 3: Через FileZilla (GUI)
1. Скачайте FileZilla: https://filezilla-project.org/
2. Подключитесь к серверу через SFTP
3. Перетащите папку проекта

---

## 5. Настройка Окружения

### 5.1. Создание виртуального окружения:
```bash
cd ~/FASTAPIGITPROJECT

# Создаем виртуальное окружение
python3.11 -m venv venv

# Активируем окружение
source venv/bin/activate

# Обновляем pip
pip install --upgrade pip
```

### 5.2. Установка зависимостей:
```bash
# Устанавливаем зависимости из requirements.txt
pip install -r requirements.txt

# Дополнительно устанавливаем gunicorn для production
pip install gunicorn
```

### 5.3. Проверка установки:
```bash
python -c "import fastapi; print('FastAPI:', fastapi.__version__)"
python -c "import uvicorn; print('Uvicorn: OK')"
```

---

## 6. Настройка Gunicorn/Uvicorn

### 6.1. Создание конфигурации Gunicorn:
```bash
nano ~/FASTAPIGITPROJECT/gunicorn_config.py
```

Вставьте следующий код:
```python
# gunicorn_config.py
import multiprocessing

# Основные настройки
bind = "127.0.0.1:8001"
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = "uvicorn.workers.UvicornWorker"

# Таймауты
timeout = 120
keepalive = 5

# Логирование
accesslog = "/home/fastapi_user/FASTAPIGITPROJECT/logs/access.log"
errorlog = "/home/fastapi_user/FASTAPIGITPROJECT/logs/error.log"
loglevel = "info"

# Производительность
max_requests = 1000
max_requests_jitter = 50
```

### 6.2. Создание директории для логов:
```bash
mkdir -p ~/FASTAPIGITPROJECT/logs
```

### 6.3. Тестовый запуск:
```bash
cd ~/FASTAPIGITPROJECT
source venv/bin/activate
gunicorn -c gunicorn_config.py main:app
```

Проверьте, что сервер запустился без ошибок, затем остановите (Ctrl+C).

---

## 7. Настройка Nginx

### 7.1. Создание конфигурации Nginx:
```bash
sudo nano /etc/nginx/sites-available/fastapi
```

Вставьте конфигурацию:
```nginx
upstream fastapi_app {
    server 127.0.0.1:8001;
}

server {
    listen 80;
    server_name ВАШ_ДОМЕН_ИЛИ_IP;
    
    client_max_body_size 100M;
    
    # Основной location
    location / {
        proxy_pass http://fastapi_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Таймауты
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # Статические файлы
    location /static {
        alias /home/fastapi_user/FASTAPIGITPROJECT/static;
        expires 30d;
    }
    
    # Логи
    access_log /var/log/nginx/fastapi_access.log;
    error_log /var/log/nginx/fastapi_error.log;
}
```

### 7.2. Активация конфигурации:
```bash
# Создаем символическую ссылку
sudo ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled/

# Удаляем дефолтную конфигурацию (опционально)
sudo rm /etc/nginx/sites-enabled/default

# Проверяем конфигурацию
sudo nginx -t

# Перезапускаем Nginx
sudo systemctl restart nginx
```

---

## 8. Настройка SSL (HTTPS)

### 8.1. Установка Certbot:
```bash
sudo apt install certbot python3-certbot-nginx -y
```

### 8.2. Получение SSL сертификата:
```bash
# Замените на ваш домен и email
sudo certbot --nginx -d ваш_домен.com -d www.ваш_домен.com --email ваш_email@example.com --agree-tos --non-interactive
```

### 8.3. Автообновление сертификата:
```bash
# Проверка автообновления
sudo certbot renew --dry-run
```

Certbot автоматически настроит cron для обновления сертификатов.

---

## 9. Автозапуск через systemd

### 9.1. Создание systemd service:
```bash
sudo nano /etc/systemd/system/fastapi.service
```

Вставьте конфигурацию:
```ini
[Unit]
Description=FastAPI Application
After=network.target

[Service]
Type=notify
User=fastapi_user
Group=www-data
WorkingDirectory=/home/fastapi_user/FASTAPIGITPROJECT
Environment="PATH=/home/fastapi_user/FASTAPIGITPROJECT/venv/bin"

ExecStart=/home/fastapi_user/FASTAPIGITPROJECT/venv/bin/gunicorn \
    -c /home/fastapi_user/FASTAPIGITPROJECT/gunicorn_config.py \
    main:app

ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true

Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### 9.2. Активация и запуск сервиса:
```bash
# Перезагружаем systemd
sudo systemctl daemon-reload

# Включаем автозапуск
sudo systemctl enable fastapi

# Запускаем сервис
sudo systemctl start fastapi

# Проверяем статус
sudo systemctl status fastapi
```

### 9.3. Управление сервисом:
```bash
# Запуск
sudo systemctl start fastapi

# Остановка
sudo systemctl stop fastapi

# Перезапуск
sudo systemctl restart fastapi

# Проверка статуса
sudo systemctl status fastapi

# Просмотр логов
sudo journalctl -u fastapi -f
```

---

## 10. Мониторинг и Логи

### 10.1. Просмотр логов приложения:
```bash
# Логи Gunicorn
tail -f ~/FASTAPIGITPROJECT/logs/access.log
tail -f ~/FASTAPIGITPROJECT/logs/error.log

# Логи systemd
sudo journalctl -u fastapi -f

# Логи Nginx
sudo tail -f /var/log/nginx/fastapi_access.log
sudo tail -f /var/log/nginx/fastapi_error.log
```

### 10.2. Мониторинг ресурсов:
```bash
# Использование CPU и RAM
htop

# Установка htop (если не установлен)
sudo apt install htop -y

# Дисковое пространство
df -h

# Использование памяти
free -m
```

### 10.3. Настройка ротации логов:
```bash
sudo nano /etc/logrotate.d/fastapi
```

Вставьте:
```
/home/fastapi_user/FASTAPIGITPROJECT/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 fastapi_user www-data
    sharedscripts
    postrotate
        systemctl reload fastapi > /dev/null 2>&1 || true
    endscript
}
```

---

## 11. Настройка Firewall (UFW)

### 11.1. Настройка базовых правил:
```bash
# Разрешаем SSH (ВАЖНО! Сделайте это ПЕРВЫМ)
sudo ufw allow 22/tcp

# Разрешаем HTTP и HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Включаем firewall
sudo ufw enable

# Проверяем статус
sudo ufw status
```

---

## 12. Обновление Проекта на Сервере

### 12.1. Через Git:
```bash
cd ~/FASTAPIGITPROJECT
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart fastapi
```

### 12.2. Автоматизация обновления (создайте скрипт):
```bash
nano ~/update_app.sh
```

Вставьте:
```bash
#!/bin/bash
cd ~/FASTAPIGITPROJECT
git pull origin main
source venv/bin/activate
pip install -r requirements.txt --quiet
sudo systemctl restart fastapi
echo "✅ Приложение обновлено и перезапущено!"
```

Сделайте скрипт исполняемым:
```bash
chmod +x ~/update_app.sh
```

Запуск обновления:
```bash
~/update_app.sh
```

---

## 13. Резервное Копирование

### 13.1. Создание backup скрипта:
```bash
nano ~/backup.sh
```

Вставьте:
```bash
#!/bin/bash
BACKUP_DIR="/home/fastapi_user/backups"
PROJECT_DIR="/home/fastapi_user/FASTAPIGITPROJECT"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/backup_$DATE.tar.gz $PROJECT_DIR

# Удаляем старые бэкапы (старше 7 дней)
find $BACKUP_DIR -name "backup_*.tar.gz" -mtime +7 -delete

echo "✅ Backup создан: backup_$DATE.tar.gz"
```

Сделайте исполняемым:
```bash
chmod +x ~/backup.sh
```

### 13.2. Автоматический backup через cron:
```bash
crontab -e
```

Добавьте строку (backup каждый день в 3:00 AM):
```
0 3 * * * /home/fastapi_user/backup.sh
```

---

## 14. Проверка Работоспособности

### 14.1. Проверка через curl:
```bash
curl http://localhost:8001
curl http://ВАШ_IP
curl https://ваш_домен.com
```

### 14.2. Проверка API:
```bash
curl http://ВАШ_IP/docs
```

### 14.3. Проверка SSL:
```bash
curl -I https://ваш_домен.com
```

---

## 15. Troubleshooting (Решение Проблем)

### Проблема: Сервис не запускается
```bash
# Проверьте логи
sudo journalctl -u fastapi -n 50

# Проверьте синтаксис конфигурации
sudo nginx -t

# Проверьте права доступа
ls -la ~/FASTAPIGITPROJECT
```

### Проблема: 502 Bad Gateway
```bash
# Проверьте, запущен ли сервис
sudo systemctl status fastapi

# Проверьте логи Nginx
sudo tail -f /var/log/nginx/fastapi_error.log

# Попробуйте перезапустить
sudo systemctl restart fastapi
sudo systemctl restart nginx
```

### Проблема: Недостаточно памяти
```bash
# Проверьте использование памяти
free -m

# Уменьшите количество workers в gunicorn_config.py
# Измените: workers = 2
```

### Проблема: Порт уже занят
```bash
# Узнайте, кто использует порт
sudo lsof -i :8001

# Убейте процесс
sudo kill -9 PID
```

---

## 16. Оптимизация Производительности

### 16.1. Настройка Nginx для кеширования:
```bash
sudo nano /etc/nginx/sites-available/fastapi
```

Добавьте в блок `server`:
```nginx
# Кеширование статики
location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
    expires 365d;
    add_header Cache-Control "public, immutable";
}
```

### 16.2. Включение gzip сжатия:
```bash
sudo nano /etc/nginx/nginx.conf
```

Убедитесь, что включено:
```nginx
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss application/rss+xml font/truetype font/opentype application/vnd.ms-fontobject image/svg+xml;
```

---

## 17. Быстрые Команды (Шпаргалка)

```bash
# Статус сервиса
sudo systemctl status fastapi

# Перезапуск
sudo systemctl restart fastapi

# Логи в реальном времени
sudo journalctl -u fastapi -f

# Обновление проекта
cd ~/FASTAPIGITPROJECT && git pull && sudo systemctl restart fastapi

# Проверка Nginx
sudo nginx -t && sudo systemctl reload nginx

# Использование ресурсов
htop

# Дисковое пространство
df -h

# Проверка портов
sudo netstat -tulpn | grep LISTEN
```

---

## 18. Альтернатива: Развертывание через Docker

Если хотите использовать Docker (уже есть в вашем проекте):

### 18.1. Установка Docker:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

### 18.2. Установка Docker Compose:
```bash
sudo apt install docker-compose -y
```

### 18.3. Запуск проекта:
```bash
cd ~/FASTAPIGITPROJECT
docker-compose up -d
```

### 18.4. Управление:
```bash
# Запуск
docker-compose up -d

# Остановка
docker-compose down

# Логи
docker-compose logs -f

# Перезапуск
docker-compose restart

# Обновление
git pull && docker-compose up -d --build
```

---

## 19. Полный Скрипт Автоматической Установки

Создайте файл `deploy.sh` на вашем локальном компьютере:

```bash
#!/bin/bash

echo "🚀 Автоматическое развертывание FastAPI проекта"
echo "================================================"

# Переменные (ЗАМЕНИТЕ НА СВОИ!)
SERVER_IP="YOUR_SERVER_IP"
SERVER_USER="fastapi_user"
DOMAIN="your-domain.com"
EMAIL="your@email.com"

echo "📦 Загрузка проекта на сервер..."
scp -r ../FASTAPIGITPROJECT $SERVER_USER@$SERVER_IP:~/

echo "🔧 Настройка сервера..."
ssh $SERVER_USER@$SERVER_IP << 'EOF'
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка необходимого ПО
sudo apt install -y python3.11 python3.11-venv python3-pip nginx git ufw

# Создание виртуального окружения
cd ~/FASTAPIGITPROJECT
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install gunicorn

# Создание директории для логов
mkdir -p ~/FASTAPIGITPROJECT/logs

# Настройка firewall
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable

echo "✅ Настройка завершена!"
EOF

echo "✅ Развертывание завершено!"
echo "🌐 Проект доступен по адресу: http://$SERVER_IP"
```

---

## 20. Контрольный Список

- [ ] Сервер арендован и доступен через SSH
- [ ] Установлен Python 3.10+
- [ ] Установлены все зависимости
- [ ] Проект загружен на сервер
- [ ] Создано виртуальное окружение
- [ ] Установлены пакеты из requirements.txt
- [ ] Настроен Gunicorn
- [ ] Настроен Nginx
- [ ] Настроен SSL (если есть домен)
- [ ] Создан systemd service
- [ ] Сервис включен в автозапуск
- [ ] Настроен Firewall (UFW)
- [ ] Проверена работоспособность
- [ ] Настроен мониторинг логов
- [ ] Настроено резервное копирование

---

## 📞 Поддержка

Если возникли проблемы:
1. Проверьте логи: `sudo journalctl -u fastapi -f`
2. Проверьте статус: `sudo systemctl status fastapi`
3. Проверьте Nginx: `sudo nginx -t`
4. Проверьте порты: `sudo netstat -tulpn | grep LISTEN`

---

**Удачи с развертыванием! 🎉**





