@startuml
!include <C4/C4_Context.puml>
HIDE_STEREOTYPE()

title Контекстная диаграмма (C4 Context) Маркетплейса — расширенный вид

Person(Customer, "Покупатель", "Использует веб-браузер или мобильное приложение")
Person(Seller, "Продавец", "Использует личный кабинет продавца")
Person(Staff, "Персонал маркетплейса", "Использует внутренние веб-интерфейсы и мобильное приложение курьера")

System_Boundary(marketplace, "Границы системы Маркетплейс") {
    System(ProductCatalog, "Товарный каталог", "Управление карточками товаров, категориями, ценами, остатками. Поиск.")
    System(OrderFulfillment, "Заказ и доставка", "Корзина, оформление заказа, статусная модель, логистика.")
    System(PaymentsFinance, "Платежи и Финансы", "Приём платежей, возвраты, выплаты продавцам, комиссии.")
    System(EngagementReputation, "Коммуникации и Репутация", "Уведомления, отзывы, рейтинги, споры.")
    System(UsersDashboards, "Пользователи и Кабинеты", "Регистрация, вход, личные кабинеты, админ-панель.")
}

System_Ext(PaymentGateway, "Платежный шлюз", "Обработка платежей (эквайринг)")
System_Ext(LogisticsAPI, "Служба доставки", "Расчёт сроков и трекинг доставки")
System_Ext(NotificationSvc, "Сервис уведомлений", "SMS, Email и Push-шлюзы")
System_Ext(BankSystem, "Банковская система / Налоговая", "Выплаты продавцам и фискализация чеков")
System_Ext(CDNStorage, "Файловое хранилище", "Хранение медиа-контента")

' Связи людей с контекстами
Rel(Customer, ProductCatalog, "Ищет и просматривает товары")
Rel(Customer, OrderFulfillment, "Оформляет заказ, отслеживает доставку")
Rel(Customer, EngagementReputation, "Оставляет отзывы")
Rel(Customer, UsersDashboards, "Регистрируется, входит")

Rel(Seller, UsersDashboards, "Входит в кабинет")
Rel(Seller, ProductCatalog, "Управляет товарами")
Rel(Seller, PaymentsFinance, "Запрашивает вывод средств")

Rel(Staff, UsersDashboards, "Входит в панель управления")
Rel(Staff, ProductCatalog, "Модерирует товары")
Rel(Staff, OrderFulfillment, "Управляет логистикой")
Rel(Staff, PaymentsFinance, "Контролирует финансы")
Rel(Staff, EngagementReputation, "Модерирует отзывы и споры")

' Связи контекстов с внешними системами
Rel(ProductCatalog, CDNStorage, "Загружает/читает изображения")
Rel(OrderFulfillment, LogisticsAPI, "Заказывает доставку, получает трек-номер")
Rel(PaymentsFinance, PaymentGateway, "Инициирует платёж, получает статус")
Rel(PaymentsFinance, BankSystem, "Запрашивает массовые выплаты")
Rel(EngagementReputation, NotificationSvc, "Отправляет уведомления")

@enduml

