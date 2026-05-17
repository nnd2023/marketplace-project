@startuml
title Логическая модель данных Маркетплейса (Logical Level)

skinparam linetype ortho
skinparam classAttributeIconSize 0
hide empty members
skinparam nodesep 80
skinparam ranksep 180

package "Пользователи и Кабинеты" {
    class UserAccount {
        * user_id : Identifier <<PK>>
        --
        email : String
        phone : String
        password_hash : String
        first_name : String
        last_name : String
        status : StatusEnum
        created_at : DateTime
        updated_at : DateTime
    }
    class UserRole {
        * role_id : Identifier <<PK>>
        --
        role_name : String
        permissions : List<String>
    }
    class UserRoles {
        * user_id : Identifier <<PK, FK>>
        * role_id : Identifier <<PK, FK>>
        --
        assigned_at : DateTime
    }
    class SellerStore {
        * store_id : Identifier <<PK>>
        * owner_user_id : Identifier <<FK>>
        --
        store_name : String
        description : Text
        legal_info : LegalData
        status : StatusEnum
        created_at : DateTime
    }
}

package "Товарный каталог" {
    class ProductCategory {
        * category_id : Identifier <<PK>>
        + parent_category_id : Identifier <<FK>>
        --
        name : String
        slug : String
        is_active : Boolean
    }
    class Product {
        * product_id : Identifier <<PK>>
        * store_id : Identifier <<FK>>
        * category_id : Identifier <<FK>>
        --
        sku : String
        title : String
        description : Text
        price : Decimal
        currency : String
        stock_quantity : Integer
        status : StatusEnum
        created_at : DateTime
        updated_at : DateTime
    }
    class MediaAsset {
        * media_id : Identifier <<PK>>
        * product_id : Identifier <<FK>>
        --
        url : String
        mime_type : String
        alt_text : String
        is_primary : Boolean
        sort_order : Integer
    }
}

package "Заказ и Логистика" {
    class Order {
        * order_id : Identifier <<PK>>
        * customer_id : Identifier <<FK>>
        --
        order_number : String
        status : OrderStatusEnum
        total_amount : Decimal
        currency : String
        delivery_address : Address
        created_at : DateTime
        updated_at : DateTime
    }
    class OrderItem {
        * order_item_id : Identifier <<PK>>
        * order_id : Identifier <<FK>>
        * product_id : Identifier <<FK>>
        --
        quantity : Integer
        unit_price : Decimal
        subtotal : Decimal
        status : OrderItemStatusEnum
    }
    class Shipment {
        * shipment_id : Identifier <<PK>>
        * order_item_id : Identifier <<FK>>        
        --
        carrier_code : String
        tracking_number : String
        status : ShipmentStatusEnum
        shipped_at : DateTime
        delivered_at : DateTime
        created_at : DateTime
    }
}

package "Финансы" {
    class Payment {
        * payment_id : Identifier <<PK>>
        * order_id : Identifier <<FK>>
        --
        amount : Decimal
        currency : String
        payment_method : String
        status : PaymentStatusEnum
        gateway_transaction_id : String
        created_at : DateTime
        processed_at : DateTime
    }
    class Payout {
        * payout_id : Identifier <<PK>>
        * store_id : Identifier <<FK>>
        --
        amount : Decimal
        currency : String
        status : PayoutStatusEnum
        requested_at : DateTime
        completed_at : DateTime
        bank_transfer_ref : String
    }
    class PayoutItem {
        * payout_id : Identifier <<PK, FK>>
        * payment_id : Identifier <<PK, FK>>
        --
        commission_amount : Decimal
        net_amount : Decimal
    }
}

package "Коммуникации и Репутация" {
    class Review {
        * review_id : Identifier <<PK>>
        * author_id : Identifier <<FK>>
        + product_id : Identifier <<FK>>
        + store_id : Identifier <<FK>>
        --
        rating : Integer
        title : String
        comment : Text
        status : ModerationStatusEnum
        created_at : DateTime
    }
    class Dispute {
        * dispute_id : Identifier <<PK>>
        * initiator_id : Identifier <<FK>>
        * order_item_id : Identifier <<FK>>
        --
        reason : String
        description : Text
        status : DisputeStatusEnum
        resolution : Text
        created_at : DateTime
    }
    class Notification {
        * notification_id : Identifier <<PK>>
        * user_id : Identifier <<FK>>
        --
        channel : String
        template_code : String
        payload : Map<String, Any>
        status : DeliveryStatusEnum
        sent_at : DateTime
        created_at : DateTime
    }
}

' === Связи (Логический уровень) ===
UserAccount "1" --> "0..*" UserRoles : имеет роль
UserRole "1" --> "0..*" UserRoles : присваивается
UserAccount "1" --> "0..1" SellerStore : владеет магазином

SellerStore "1" --> "0..*" Product : публикует товары
Product "0..*" --> "1" ProductCategory : принадлежит категории
Product "1" --> "0..*" MediaAsset : имеет медиа
ProductCategory "0..*" --> "0..1" ProductCategory : иерархия

Order "1" --> "1..*" OrderItem : содержит позиции
Order "0..*" --> "1" UserAccount : размещает
OrderItem "0..*" --> "1" Product : ссылается на товар
Shipment "0..1" --> "1..*" OrderItem : агрегирует позиции

Order "1" --> "0..*" Payment : оплачивается
Payment "1" --> "0..*" PayoutItem : входит в выплату
Payout "1" --> "0..*" PayoutItem : состоит из платежей
Payout "0..*" --> "1" SellerStore : выплачивается магазину

Review "0..*" --> "1" UserAccount : написан пользователем
Review "0..1" --> "1" Product : оценивает товар
Review "0..1" --> "1" SellerStore : оценивает магазин

Dispute "0..*" --> "1" UserAccount : создан пользователем
Dispute "0..*" --> "1" OrderItem : касается позиции

Notification "0..*" --> "1" UserAccount : адресована пользователю
@enduml

