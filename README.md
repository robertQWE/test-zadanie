Описание микросервисов WarmHouse System

Billing Service
1. Бизнес-описание
Обеспечивает управление финансовыми операциями: выставление счетов, приём оплаты, интеграция с внешними платёжными системами.
2. Методы
•	POST /invoices — создать счёт
•	GET /invoices/{id} — получить информацию о счёте
•	POST /payments — провести платёж
•	GET /payments/{id} — статус платежа
•	GET /user/{userId}/balance — получить текущий баланс пользователя
3. База данных
•	invoices (id, user_id, amount, status, due_date, created_at)
•	payments (id, invoice_id, amount, method, status, processed_at)
4. Справочники
•	invoice_status: PENDING, PAID, CANCELLED
•	payment_status: INITIATED, SUCCESS, FAILED
•	payment_method: CARD, BANK_TRANSFER, WALLET
5. Мэппинг полей
Пример запроса:
{ "invoiceId": "string", "amount": "number", "method": "string" }
Пример ответа:
{ "paymentId": "string", "status": "string" }

User Management Service
1. Бизнес-описание
Регистрация, аутентификация, авторизация пользователей, управление личным кабинетом.
2. Методы
•	POST /users — регистрация
•	POST /auth/login — вход
•	GET /users/{id} — профиль пользователя
•	PUT /users/{id} — обновление данных
•	DELETE /users/{id} — деактивация
3. База данных
•	users (id, email, phone, password_hash, role, is_active)
•	sessions (id, user_id, token, created_at, expired_at)
4. Справочники
•	role: USER, ADMIN
•	auth_method: EMAIL, SMS, OAUTH
5. Мэппинг полей
Пример запроса:
{ "email": "string", "password": "string", "phone": "string" }
Пример ответа:
{ "userId": "string", "role": "USER" }

Space Management Service

1. Бизнес-описание
Управление пространствами (квартира, комната и т.д.) и доступом пользователей к ним.
2. Методы
•	POST /spaces — создать пространство
•	GET /spaces/{id} — получить пространство
•	POST /spaces/{id}/users — добавить пользователя
•	GET /spaces/{id}/users — список пользователей
•	DELETE /spaces/{id}/users/{userId} — удалить пользователя
3. База данных
•	spaces (id, name, owner_id, created_at)
•	space_users (id, space_id, user_id, role)
4. Справочники
•	space_user_role: OWNER, GUEST, MAINTAINER
5. Мэппинг полей
Пример запроса:
{ "name": "string", "ownerId": "string" }
Пример ответа:
{ "spaceId": "string" }

Device Management Service

1. Бизнес-описание
Регистрация и управление IoT-устройствами, контроль их состояния, связь с внешними приборами.
2. Методы
•	POST /devices — зарегистрировать устройство
•	GET /devices/{id} — получить инфу об устройстве
•	PATCH /devices/{id}/status — обновить статус
•	POST /devices/{id}/command — отправить команду
3. База данных
•	devices (id, space_id, type, name, status, last_seen)
•	device_logs (id, device_id, event, timestamp)
4. Справочники
•	device_type: HEATER, SENSOR, CONTROLLER
•	device_status: ONLINE, OFFLINE, ERROR
5. Мэппинг полей
Пример запроса:
{ "status": "ONLINE" }
Пример ответа:
{ "deviceId": "string", "status": "ONLINE" }

Support Service

1. Бизнес-описание
Приём и обработка обращений пользователей по техническим и организационным вопросам.
2. Методы
•	POST /tickets — создать тикет
•	GET /tickets/{id} — получить тикет
•	POST /tickets/{id}/comments — добавить комментарий
•	PATCH /tickets/{id} — обновить статус
3. База данных
•	tickets (id, user_id, subject, description, status, created_at)
•	comments (id, ticket_id, author_id, message, timestamp)
4. Справочники
•	ticket_status: OPEN, IN_PROGRESS, RESOLVED, CLOSED
5. Мэппинг полей
Пример запроса:
{ "userId": "string", "subject": "string", "description": "string" }
Пример ответа:
{ "ticketId": "string", "status": "OPEN" }

2. Описание взаимодействия микросервисов с БД 
•	Billing Service -Billing DB — читает/записывает информацию о транзакциях, выставленных счетах и статусах оплат.
•	User Management Service -User DB — управляет данными пользователей, включая учетные записи, права доступа.
•	Space Management Service - Space DB — хранит данные о пространствах (комнаты, помещения), а также их связях с пользователями.
•	Device Management Service - Device DB — управляет конфигурацией, метаданными и состоянием подключённых устройств.
•	Support Service -Support DB — логирует обращения, инциденты, запросы пользователей.
•	Все микросервисы -Message Broker (Kafka) — асинхронный обмен событиями между сервисами (например, создание нового пользователя или регистрация нового устройства).
•	Billing Service -Платёжные системы — вызывает внешние API для обработки платежей.
•	Device Management Service -Внешние устройства / датчики — обменивается данными с отопительными приборами, температурными датчиками и др.

1. Дополнить архитектуру
Можно добавить Notification Service (уведомления через e-mail/SMS/Push) для информирования пользователей об изменениях статуса устройств или биллинга.
Почему:
Это позволит улучшить взаимодействие с пользователем, уведомляя его о критических событиях (например, утечка газа, завершение подписки и т.д.).
2. Указать тип API для взаимодействия микросервисов
•	Внутреннее взаимодействие:
Используем Kafka, особенно для масштабируемости и производительности.
•	Внешнее взаимодействие:
Используем REST API, так как оно проще для интеграции, отладки и взаимодействия с фронтендом/мобильными клиентами.


@startuml WarmHouse Container

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()

title Container diagram for WarmHouse System

Person(user, "Пользователь", "Пользователь умного дома")
Person(admin, "Администратор", "Администратор умного дома")

System_Boundary(warmhouse, "WarmHouse") {
    Container(api_gateway, "API Gateway", "Nginx", "Маршрутизация запросов и безопасность")
    Container(user_management_service, "User Management Service", "Java Spring Boot", "Управление пользователями и личным кабинетом")
    Container(space_management_service, "Space Management Service", "Node.js", "Управление пространствами и доступом")
    Container(device_management_service, "Device Management Service", "Python Flask", "Менеджмент устройств")
    Container(billing_service, "Billing Service", "Ruby on Rails", "Система биллинга")
    Container(support_service, "Support Service", "Go", "Поддержка и эксплуатация")
    
    ContainerDb(user_db, "User DB", "PostgreSQL", "Хранение данных пользователей")
    ContainerDb(space_db, "Space DB", "PostgreSQL", "Хранение данных пространств")
    ContainerDb(device_db, "Device DB", "PostgreSQL", "Хранение данных устройств")
    ContainerDb(billing_db, "Billing DB", "PostgreSQL", "Хранение данных биллинга")
    ContainerDb(support_db, "Support DB", "PostgreSQL", "Хранение данных поддержки")
    
    ContainerQueue(message_broker, "Message Broker", "Kafka", "Обмен сообщениями между сервисами")
}

System_Ext(controllers, "Внешние устройства", "Управляемые отопительные приборы")
System_Ext(sensors, "Внешние датчики", "Датчики температуры")
System_Ext(payment_providers, "Платежные системы", "Внешние процессинговые сервисы")

Rel(user, api_gateway, "Использует")
Rel(admin, api_gateway, "Использует")

Rel(api_gateway, user_management_service, "Маршрутизация")
Rel(api_gateway, space_management_service, "Маршрутизация")
Rel(api_gateway, device_management_service, "Маршрутизация")
Rel(api_gateway, billing_service, "Маршрутизация")
Rel(api_gateway, support_service, "Маршрутизация")

Rel(user_management_service, user_db, "управляет данными пользователей, включая учетные записи, права доступа")
Rel(space_management_service, space_db, "хранит данные о пространствах (комнаты, помещения), а также их связях с пользователями")
Rel(device_management_service, device_db, "управляет конфигурацией, метаданными и состоянием подключённых устройств")
Rel(billing_service, billing_db, "читает/записывает информацию о транзакциях, выставленных счетах и статусах оплат")
Rel(support_service, support_db, "логирует обращения, инциденты, запросы пользователей")

Rel(controllers, message_broker, "?")
Rel(sensors, message_broker, "?")
Rel(device_management_service, message_broker, "асинхронный обмен событиями между сервисами")
Rel(device_management_service, controllers, "обменивается данными с отопительными приборами, температурными датчиками и др.")

Rel(billing_service, payment_providers, "вызывает внешние API для обработки платежей")

@enduml
