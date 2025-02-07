# 🖥️ Контейнерный мониторинг

## 📌 Техническое задание

Необходимо написать приложение на языках программирования **Go и JavaScript (TS)**, которое получает **IP-адреса контейнеров Docker, пингует их с определённым интервалом и сохраняет данные в базу данных**.

Получение данных о состоянии контейнеров доступно на динамически формируемой веб-странице.

### **Сервисы**

В результате выполнения задания должны появиться **4 сервиса**:

1. **Backend-сервис** — RESTful API для запроса данных из базы и добавления новых данных.
2. **Frontend-сервис** — веб-интерфейс на JavaScript (React+Vite+TS) для отображения статусов контейнеров.
3. **База данных PostgreSQL** — хранилище данных о контейнерах.
4. **Сервис Pinger** — получает список всех Docker-контейнеров, пингует их и отправляет результаты.

**Дополнительные требования:**
- Использование **Nginx** для обратного прокси.
- Возможность **работы через сервис очередей (RabbitMQ)**.
- Изоляция сервисов через **Docker Compose** и `netns`.

### **Результат**
- Созданы **Dockerfile** для каждого сервиса и общий **docker-compose.yml**.
- После сборки можно зайти через `http://localhost` и увидеть таблицу с контейнерами.
- Проект опубликован на **GitHub** с `README.md`, описывающим функционал и шаги запуска.

---

## **📌 Анализ ТЗ**

1. **Конкретных ручек на Backend нет** (реализованы две: получение и удаление данных).
2. **Требований по документации нет** (базовая Swagger-документация присутствует).
3. **Время пинга** — используется **дата последней проверки** и статус контейнера.
4. **Нет требований по использованию ORM** (используется **GORM**).
5. **Какие контейнеры отслеживать — не указано.**
   - **Решение:** Получаем контейнеры из **той же сети Docker Compose**, т.к. доступ к внешним контейнерам может быть небезопасным. В противном случае это требует дополнительных настроек,
   которые не входят в рамки данного ТЗ. 
6. **Нет требований по RabbitMQ и Nginx** — реализовано на своё усмотрение.
7. **В ТЗ указан REST-запрос от `pinger` к `backend`**.
   - Реализованы **оба варианта общения**:
     - **REST API** (прямые HTTP-запросы).
     - **RabbitMQ** (асинхронная очередь сообщений).

---

## **📌 Технологический стек**

| Компонент    | Технологии |
|-------------|------------|
| **Frontend** | React, Vite, TypeScript |
| **Backend**  | Go, GORM, Gin, PostgreSQL |
| **Pinger**   | Go, Docker API, RabbitMQ |
| **Очереди**  | RabbitMQ |
| **База данных** | PostgreSQL |
| **Прочее**   | Docker, Docker Compose, Nginx, Git, GitHub |

---

## **📌 Функционал**

- Веб-страница с **таблицей контейнеров** (IP-адрес, статус, дата проверки).
- `pinger` получает контейнеры через **Docker API (`/var/run/docker.sock`)**.
- Данные отправляются **либо в RabbitMQ, либо в REST API backend**.
- `backend` сохраняет данные в **PostgreSQL**.
- `frontend` получает данные через **REST API** и динамически обновляет интерфейс.
- `nginx` выполняет роль **обратного прокси** между `frontend` и `backend`.

---

## **📌 Реализация сервисов**

| Сервис | Описание |
|--------|----------|
| **Backend** | Получает данные от `pinger`, сохраняет в БД, отправляет на `frontend`. |
| **Frontend** | Запрашивает данные из `backend`, отображает таблицу в браузере. |
| **Pinger** | Получает список контейнеров, пингует их и отправляет на `backend` (через REST или RabbitMQ). |
| **Database** | PostgreSQL — хранит информацию о контейнерах. |
| **Nginx** | Обратный прокси — маршрутизирует запросы между сервисами. |
| **RabbitMQ** | Сервис очередей для асинхронной передачи данных. |

---

## **📌 Инструкция по настройке**

### **1️⃣ Проверка свободных портов**
Перед запуском убедитесь, что порты **80, 8080, 5672, 15672, 5432** свободны.
```sh
sudo netstat -tulnp | grep -E ':80|:8080|:5672|:15672|:5432'
```

### **2️⃣ Клонирование репозитория**
```sh
git clone git@github.com:Leganyst/ping_service.git
cd ping_service
```

### **3️⃣ Выбор реализации**
```sh
git checkout main  # Без RabbitMQ
git checkout dev/rabbitmq  # С RabbitMQ
```

### **4️⃣ Настройка `.env`**
Создайте `.env` файл и заполните переменные (пример):
```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=containers
POSTGRES_ADDRESS=database
POSTGRES_PORT=5432

BACKEND_URL=http://backend:8080

RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672/
RABBITMQ_DEFAULT_USER=quest
RABBITMQ_DEFAULT_PASS=guest
```

### **5️⃣ Запуск контейнеров**
```sh
docker-compose up -d --build
```
После запуска открой:
- **Frontend**: `http://localhost`
- **Backend API**: `http://localhost:8080/api/`
- **RabbitMQ UI**: `http://localhost:15672` (логин: `guest`, пароль: `guest`)

### **6️⃣ Остановка контейнеров**
```sh
docker-compose down -v
```

