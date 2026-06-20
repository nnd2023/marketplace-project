@startuml
!include <C4/C4_Container.puml>
HIDE_STEREOTYPE()

title Контейнерная диаграмма (C4 Container) Маркетплейса

' === Акторы ===
Person(Customer, "Покупатель", "Использует веб-браузер или мобильное приложение")
Person(Seller, "Продавец", "Использует личный кабинет продавца")
Person(Staff, "Персонал маркетплейса", "Использует внутренние веб-интерфейсы и приложение курьера")

' === Внешние системы ===
System_Ext(PaymentGateway, "Платежный шлюз", "Эквайринг и валидация карт")
System_Ext(LogisticsAPI, "Служба доставки", "Расчёт сроков, API трекинга")
System_Ext(NotificationSvc, "Сервис уведомлений", "SMS, Email и Push-шлюзы")
System_Ext(BankSystem, "Банковская система / ФНС", "Массовые выплаты и фискализация")
System_Ext(CDNStorage, "Файловое хранилище (CDN/S3)", "Хранение и доставка медиа-контента")

System_Boundary(Marketplace, "Платформа Маркетплейс") {
    Container(APIGateway, "API Gateway", "Kong/Nginx", "Маршрутизация, JWT-аутентификация, rate-limiting, SSL termination")
    Container(MessageBroker, "Message Broker", "RabbitMQ/Kafka", "Асинхронная шина событий (OrderCreated, PaymentCompleted и т.д.)")


Container_Boundary(frontend, "Клиентские приложения") {
        Container(WebCustomer, "Веб-приложение покупателя", "SPA (React/Vue)", "Поиск, корзина, оформление заказа, личный кабинет")
        Container(WebSeller, "Портал продавца", "SPA (React/Vue)", "Управление каталогом, заказами, финансами, отзывами")
        Container(WebAdmin, "Админ-панель персонала", "SPA (React/Angular)", "Модерация, управление логистикой, финансами, поддержкой")
        Container(MobileApp, "Мобильное приложение", "iOS/Android (Flutter)", "Покупки, трекинг, push-уведомления")
    }

    Container_Boundary(UsersBoundary, "Пользователи и Кабинеты") {
        Container(AuthAPI, "Auth & User API", "Go/REST", "Регистрация, вход, управление ролями, профили продавцов")
        ContainerDb(UsersDB, "Users DB", "PostgreSQL", "Схемы: UserAccount, UserRole, UserRoles, SellerStore")
    }

    Container_Boundary(CatalogBoundary, "Товарный каталог") {
        Container(CatalogAPI, "Catalog API", "Python/FastAPI", "CRUD товаров, категорий, управление ценами и остатками")
        ContainerDb(CatalogDB, "Catalog DB", "PostgreSQL", "Схемы: ProductCategory, Product, MediaAsset (метаданные)")
        Container(SearchEngine, "Search Engine", "Elasticsearch", "Индексирование, полнотекстовый поиск, фасетная фильтрация")
    }

    Container_Boundary(OrdersBoundary, "Заказ и Логистика") {
        Container(OrderAPI, "Order & Cart API", "Java/Spring Boot", "Корзина, checkout, статусная модель заказов")
        ContainerDb(OrdersDB, "Orders DB", "PostgreSQL", "Схемы: Order, OrderItem, Shipment")
        Container(LogisticsWorker, "Logistics Worker", "Python/Celery", "Фоновая синхронизация трекинг-номеров и статусов")
    }

    Container_Boundary(FinanceBoundary, "Платежи и Финансы") {
        Container(FinanceAPI, "Finance API", "Java/Spring Boot", "Приём платежей, расчёт комиссий, управление выплатами")
        ContainerDb(FinanceDB, "Finance DB", "PostgreSQL", "Схемы: Payment, Payout, PayoutItem")
        Container(PayoutScheduler, "Payout Scheduler", "Quartz/Cron", "Пакетная генерация реестров для массовых выплат")
    }

    Container_Boundary(EngagementBoundary, "Коммуникации и Репутация") {
        Container(ReputationAPI, "Review & Dispute API", "Node.js/NestJS", "Отзывы, рейтинги, модерация, работа со спорами")
        ContainerDb(ReputationDB, "Reputation DB", "PostgreSQL", "Схемы: Review, Dispute")
        Container(NotificationWorker, "Notification Service", "Go/gRPC", "Рендеринг шаблонов, маршрутизация по каналам, логирование")
        ContainerDb(NotifDB, "Notifications DB", "PostgreSQL", "Схема: Notification, история отправок")
    }
}

' === Связи: Акторы -> UI ===
Rel(Customer, WebCustomer, "Использует", "HTTPS")
Rel(Customer, MobileApp, "Использует", "HTTPS")
Rel(Seller, WebSeller, "Использует", "HTTPS")
Rel(Staff, WebAdmin, "Использует", "HTTPS")

' === Связи: UI -> Gateway -> API ===
Rel(WebCustomer, APIGateway, "REST/GraphQL", "HTTPS")
Rel(WebSeller, APIGateway, "REST/GraphQL", "HTTPS")
Rel(WebAdmin, APIGateway, "REST/GraphQL", "HTTPS")
Rel(MobileApp, APIGateway, "REST/GraphQL", "HTTPS")


Rel(APIGateway, AuthAPI, "Forward auth & user requests")
Rel(APIGateway, CatalogAPI, "Forward catalog requests")
Rel(APIGateway, OrderAPI, "Forward cart/order requests")
Rel(APIGateway, FinanceAPI, "Forward payment requests")
Rel(APIGateway, ReputationAPI, "Forward review/dispute requests")
Rel(APIGateway, NotificationWorker, "Forward notification config")

' === Связи: API -> DB & Internal ===
Rel(AuthAPI, UsersDB, "Чтение/запись (ACID)")
Rel(CatalogAPI, CatalogDB, "Чтение/запись")
Rel(CatalogAPI, SearchEngine, "Синхронизация индекса (CDC/Queue)")
Rel(OrderAPI, OrdersDB, "Чтение/запись")
Rel(FinanceAPI, FinanceDB, "Чтение/запись")
Rel(ReputationAPI, ReputationDB, "Чтение/запись")
Rel(NotificationWorker, NotifDB, "Чтение/запись")

' === Связи: Async / Event-Driven ===
Rel(OrderAPI, MessageBroker, "Публикует: OrderCreated, PaymentPending")
Rel(FinanceAPI, MessageBroker, "Публикует: PaymentCompleted, PayoutScheduled")
Rel(ReputationAPI, MessageBroker, "Публикует: ReviewSubmitted")
Rel(MessageBroker, NotificationWorker, "Потребляет события для рассылки")
Rel(MessageBroker, LogisticsWorker, "Потребляет события для трекинга")

' === Связи: Внешние интеграции ===
Rel(CatalogAPI, CDNStorage, "Загружает/получает presigned URLs на изображения")
Rel(FinanceAPI, PaymentGateway, "Инициирует холд/списание, получает статус транзакции")
Rel(FinanceAPI, BankSystem, "Отправляет реестры выплат, получает отчёты")
Rel(LogisticsWorker, LogisticsAPI, "Получает трекинг-номера и статусы")
Rel(NotificationWorker, NotificationSvc, "Отправляет SMS/Email/Push")

' === Cross-cutting / Background ===
Rel(LogisticsWorker, OrdersDB, "Обновляет статус Shipment")
Rel(PayoutScheduler, FinanceAPI, "Триггерит массовые выплаты")
Rel(PayoutScheduler, FinanceDB, "Формирует Payout/PayoutItem")

@enduml
