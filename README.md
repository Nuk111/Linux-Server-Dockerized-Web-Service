# Linux Server + Dockerized Web Service on Arch Linux

## Описание проекта

Этот проект демонстрирует навыки системного администрирования Linux и контейнеризации приложений. 

* настраивать безопасный Linux-сервер на Arch Linux без GUI
* управлять пользователями и правами
* обеспечивать доступ через SSH с ключами
* конфигурировать firewall и базовую сетевую безопасность
* разворачивать контейнеризированные приложения с Docker и Docker Compose
* интегрировать контейнеры с хостовым веб-сервером (nginx)

## Архитектура

```
[ Internet ]
     |
  Firewall (nftables)
     |
  SSH (keys only)
     |
  Host Nginx
     |
  ---------------------
  | Docker network    |
  |                   |
  | nginx (container) | ---> backend (container)
  |                   |
  |        volume     |
  ---------------------
```

* **Host Nginx** — прокси, обеспечивает доступ извне
* **Docker network** — изолированная сеть контейнеров
* **Backend container** — приложение (Python)
* **Frontend container** — веб-сервер или UI, опционально

## Функциональные возможности

* SSH-доступ с ключами (root-доступ запрещён)
* Пользовательский аккаунт с sudo
* Firewall с минимально необходимыми правилами (`ssh`, `http`)
* Docker и Docker Compose для контейнеризации приложений
* Nginx на хосте как reverse proxy к контейнерам
* Логи и базовая диагностика ошибок
* Переменные окружения и volumes для хранения данных вне контейнеров

## Стек технологий

* **ОС:** Arch Linux (systemd-based, CLI-only)
* **Сервисы на хосте:** OpenSSH, nftables, nginx
* **Контейнеризация:** Docker, Docker Compose
* **Языки / приложения:** Python (FastAPI)
* **Мониторинг и диагностика:** systemctl, journalctl, docker logs

## Установка и запуск

### 1. Настройка сервера (Arch Linux)

```bash
# Обновление системы
sudo pacman -Syu

# Создание пользователя и добавление в группу wheel
sudo useradd -m -G wheel -s /bin/bash devops
sudo passwd devops

# Настройка sudo
sudo pacman -S sudo
sudo EDITOR=vim visudo
# Раскомментировать: %wheel ALL=(ALL:ALL) ALL

# SSH
sudo pacman -S openssh
sudo systemctl enable --now sshd
# Добавить свой публичный ключ в ~/.ssh/authorized_keys

# Firewall (nftables)
sudo pacman -S nftables
sudo systemctl enable --now nftables
# Настроить /etc/nftables.conf
sudo nft -f /etc/nftables.conf
```

### 2. Установка Docker и Docker Compose

```bash
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker
```

### 3. Запуск контейнеров

```bash
cd docker
docker-compose up -d
```

Проверка контейнеров:

```bash
docker ps
docker logs <container_name>
```

### 4. Проверка Nginx

* Доступ к приложению через браузер или curl:

```
http://<your-server-ip>
```

* Логи Nginx на хосте: `/var/log/nginx/access.log` и `/var/log/nginx/error.log`

## Структура репозитория

```
linux-docker-service/
├── README.md
├── docs/                   # документация
├── host/                   # конфигурации для хоста
│   ├── nginx/
│   ├── firewall.md
│   └── users.md
├── docker/
│   ├── docker-compose.yml
│   ├── backend/
│   │   ├── Dockerfile
│   │   └── app.py
│   └── frontend/
│       └── Dockerfile
```

## Документация

* **docs/architecture.md** — схема архитектуры и объяснение сетевых связей
* **docs/security.md** — меры безопасности и объяснение firewall, SSH
* **docs/troubleshooting.md** — типовые ошибки и как их исправлять

## Результат

После выполнения всех шагов получим:

* безопасный Arch Linux сервер
* доступ по SSH с ключами
* firewall с базовыми правилами
* работающий Dockerized web service
* nginx как reverse proxy
* полностью воспроизводимая среда

## Возможные улучшения / следующие шаги

* Добавить monitoring: Prometheus + Grafana
* Настроить CI/CD для автоматического деплоя с GitHub Actions
* Настроить HTTPS (Let's Encrypt)
* Ввести логирование и алерты на уровне Docker контейнеров
