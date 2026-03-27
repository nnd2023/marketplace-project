@startuml
!include <C4/C4_Context.puml>

title Контекстная диаграмма (C4 Context): Система Маркетплейса

' Определение системы
System_Boundary(marketplace, "Границы системы Маркетплейс") {
    System(MarketplaceSystem, "Платформа Маркетплейса")
}

' --- Люди (Акторы) ---
Person(СustomerWeb, "Покупатель использующий браузер", "Ищет товары, оформляет заказы, оплачивает, оставляет отзывы. (UC-001..008, UC-020)")
Person(СustomerMob, "Покупатель использующий мобильное приложение", "Ищет товары, оформляет заказы, оплачивает, оставляет отзывы. (UC-001..008, UC-020)")
Person(Seller, "Продавец", "Регистрируется, загружает товары, управляет ценами и остатками, выводит средства. (UC-009..013)")
Person(Admin, "Администратор маркетплейса", "Управляет настройками системы, промо-акциями и баннерами. (UC-027, UC-028)")
Person(WarehouseOp, "Оператор склада", "Принимает товары, комплектует заказы, передает курьерам. (UC-014..016)")
Person(Courier, "Курьер доставки", "Получает задания, обновляет статус доставки, подтверждает вручение. (UC-016, UC-017)")
Person(Support, "Поддержка", "Обрабатывает обращения, споры и возвраты. (UC-018, UC-033)")
Person(Moderator, "Модератор контента", "Проверяет карточки товаров и отзывы перед публикацией. (UC-019, UC-020)")
Person(Manager, "Менеджер по работе с продавцами", "Курирует продавцов, решает сложные вопросы. (UC-009)")
Person(FinOperator, "Финансовый оператор", "Контролирует транзакции, комиссии, начисления и выплаты. (UC-024, UC-025, UC-013)")
Person(SysAdmin, "Системный администратор", "Мониторит здоровье системы, управляет доступом. (UC-031)")

' --- Внешние системы (External Systems) ---
System_Ext(PaymentGateway, "Платежный шлюз", "Внешняя система обработки платежей (эквайринг).", $tags="external")
System_Ext(LogisticsAPI, "Служба доставки", "Внешняя система логистических операторов для расчета сроков и трекинга.", $tags="external")
System_Ext(NotificationSvc, "Сервис уведомлений", "SMS, Email и Push-шлюзы для оповещения пользователей.", $tags="external")
System_Ext(BankSystem, "Банковская система / Налоговая", "Система для выплат продавцам (трансферы) и фискализации чеков.", $tags="external")
System_Ext(CDNStorage, "CDN / Файловое хранилище", "Хранилище медиа-контента (фото/видео товаров).", $tags="external")


' --- Связи (Люди -> Система) ---
Rel(СustomerWeb, MarketplaceSystem, "Ищет товары, делает заказ, оплачивает", "REST / GraphQL (HTTPS) и WebSocket (SSE)")
Rel(СustomerMob, MarketplaceSystem, "Ищет товары, делает заказ, оплачивает", "GraphQL (HTTPS) и Push Notifications")
Rel(Seller, MarketplaceSystem, "Управляет ассортиментом, ценами, заказами", "REST (HTTPS) и WebSocket")
Rel(Admin, MarketplaceSystem, "Настраивает витрину и акции", "REST (HTTPS) и WebSocket")
Rel(WarehouseOp, MarketplaceSystem, "Сканирует товары, обновляет статусы", "WebSocket (TLS)")
Rel(Courier, MarketplaceSystem, "Получает маршрут, меняет статус", "REST (HTTPS) и Local DB Sync")
Rel(Support, MarketplaceSystem, "Ведет тикеты, управляет спорами", "REST + WebSocket")
Rel(Moderator, MarketplaceSystem, "Одобряет/отклоняет контент", "REST + WebSocket")
Rel(Manager, MarketplaceSystem, "Анализирует метрики продавцов", "GraphQL")
Rel(FinOperator, MarketplaceSystem, "Сверяет платежи, инициирует выплаты", "REST (Idempotent)")
Rel(SysAdmin, MarketplaceSystem, "Мониторит логи и метрики", "Prometheus API")



' --- Связи (Система -> Внешние системы) ---

Rel(MarketplaceSystem, PaymentGateway, "Инициирует платеж, получает статус", "REST + Webhooks")
Rel(MarketplaceSystem, LogisticsAPI, "Заказывает курьера, получает трек-номер", "REST + Webhooks")
Rel(MarketplaceSystem, BankSystem, "Запрашивает массовые выплаты продавцам", "SFTP (Reestry) + API и Message Queue + HTTP")
Rel(MarketplaceSystem, NotificationSvc, "Отправляет события для уведомлений", "Message Queue (Kafka/RabbitMQ)")
Rel(MarketplaceSystem, CDNStorage, "Загружает/читает изображения товаров", "S3 API (Write) + HTTP (Read)")

@enduml
