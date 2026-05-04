@startuml
title Концептуальная модель данных Маркетплейса (Final)

skinparam linetype ortho
skinparam style strictuml
skinparam classAttributeIconSize 0
hide empty members
skinparam nodesep 70
skinparam ranksep 150

package "Пользователи и Кабинеты" {
    class UserAccount <<Entity>>
    class UserRole <<Entity>>
    class SellerStore <<Entity>>
}

package "Товарный каталог" {
    class ProductCategory <<Entity>>
    class Product <<Entity>>
    class MediaAsset <<Entity>>
}

package "Заказ и Логистика" {
    class Order <<Entity>>
    class OrderItem <<Entity>>
    class Shipment <<Entity>>
}

package "Финансы" {
    class Payment <<Entity>>
    class Payout <<Entity>>
}

package "Коммуникации и Репутация" {
    class Review <<Entity>>
    class Dispute <<Entity>>
    class Notification <<Entity>>
}

' === Связи сущностей ===
UserAccount --> UserRole : имеет
UserAccount --> SellerStore : управляет
UserAccount --> Order : размещает

SellerStore --> Product : публикует
Product --> MediaAsset : содержит
Product --> ProductCategory : принадлежит
ProductCategory --> ProductCategory : образует иерархию

Order --> OrderItem : состоит из
Order --> Payment : требует
OrderItem --> Product : резервирует
Shipment --> OrderItem : агрегирует

UserAccount --> Review : публикует
Review --> Product : оценивает
Review --> SellerStore : оценивает

UserAccount --> Dispute : инициирует
Dispute --> OrderItem : относится к

Notification --> UserAccount : доставляется

SellerStore --> Payout : получает
Payout --> Payment : формируется из
@enduml


[Концептуальная модель данных](https://github.com/nnd2023/marketplace-project/blob/main/%D0%9A%D0%BE%D0%BD%D1%86%D0%B5%D0%BF%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F%20%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85/%D0%9A%D0%BE%D0%BD%D1%86%D0%B5%D0%BF%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F%20%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85.md)
