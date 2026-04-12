@startuml

skinparam linetype ortho
skinparam style strictuml
skinparam classAttributeIconSize 0
hide empty members
skinparam nodesep 80
skinparam ranksep 100
' === Сущности (классы) ===

class ClientUsers {
}

class ServiceUsers {
}

class Customers {
}

class Sellers {
}

class Admins {
}

class Moderators {
}

class Supports {
}

class Couriers {
}

class Managers {
}

class Carts {
}

class CartItems {
}

class Orders {
}

class OrderItems {
}

class PaymentOrders {
}

class DeliveryItems {
}

class DeliveryOrders {
}

class DeliveryAddresses {
}

class WarehouseAddresses {
}

class Products {
}

class ProductVariants {
}

class Categories {
}

class ProductReserves {
}

class Notifications {
}

class Reviews {
}

class Returns {
}

class WarehouseStocks {
}

' === Связи ===

' ClientUsers -> роли
ClientUsers --|> Customers : "имеет профиль"
ClientUsers --|> Sellers : "имеет профиль"

' ServiceUsers -> роли
ServiceUsers --|> Admins : "имеет профиль"
ServiceUsers --|> Moderators : "имеет профиль"
ServiceUsers --|> Supports : "имеет профиль"
ServiceUsers --|> Couriers : "имеет профиль"
ServiceUsers --|> Managers : "имеет профиль"

' Действия ролей
Customers --> Carts : "управляет"
Customers --> PaymentOrders : "исполняет"
Customers --> DeliveryAddresses : "имеет"
Customers --> Returns : "создает"
Customers --> Reviews : "создает"

Sellers --> Products : "продает"
Sellers --> WarehouseAddresses : "имеет"

Admins --> Categories : "создает"

Moderators --> Reviews : "согласовывает"
'Moderators --> Returns : "согласовывает"

Supports --> Returns : "согласовывает"

Couriers --> DeliveryOrders : "реализует"

Managers --> Returns : "утверждает"

' Корзина и заказы
Carts --> CartItems : "хранит"
Carts --> Orders : "создает"

CartItems --> ProductReserves : "создает"

Orders --> OrderItems : "содержит"
Orders --> PaymentOrders : "создает"

DeliveryOrders --> DeliveryAddresses : "относится к"
DeliveryOrders --> DeliveryItems : "состоит из"

OrderItems --> ProductVariants : "ссылается"
OrderItems --> DeliveryItems : "создает"
OrderItems --> ProductReserves : "утвержадет"

' Платежи и уведомления
PaymentOrders --> Notifications : "создает"


' Товарный каталог
Products --> ProductVariants : "включает в себя"
Products --> Categories : "наполняют"

ProductVariants --> WarehouseStocks : "относятся"
ProductVariants --> CartItems : "порождает"
Reviews --> ProductVariants : "относится к"
WarehouseStocks --> WarehouseAddresses : "хранится"

' Резервы и уведомления
ProductReserves --> Notifications : "создает"
ProductReserves --> WarehouseStocks : "блокирует"


' Возвраты и уведомления
Returns --> Notifications : "создает"
Returns --> OrderItems : "относится к"

' Уведомления клиентам
Notifications --> ClientUsers : "направляет"

@enduml
