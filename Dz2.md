```mermaid

componentDiagram
    classDef frontend fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef backend fill:#e1f5fe,stroke:#0277bd,stroke-width:2px;
    classDef data fill:#fff3e0,stroke:#ef6c00,stroke-width:2px;
    classDef external fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,stroke-dasharray: 5 5;

    %%  (FRONTEND)
    package "Presentation Layer" {
        [Client Web App]:::frontend
        [Courier Mobile App]:::frontend
    }

    %%  БИЗНЕС-ЛОГИКA (BACKEND)
    package "Backend Ecosystem" {
        [API Gateway]:::backend
        
        component "Order Management Service" as OrderCore:::backend
        component "Warehouse System" as Warehouse:::backend
        component "Route Optimizer Engine" as Optimizer:::backend
        component "Analytics & Reporting" as Analytics:::backend
        component "Notification Service" as Notifier:::backend
        component "Courier Integration Adapter" as CourierBridge:::backend
    }

    %% 3.  ДАННЫE
    package "Data Layer" {
        database "Main Database (SQL/NoSQL)" as DB:::data
        [Redis Cache]:::data
    }

    %% 4. ВНЕШНИЕ СИСТЕМЫ
    package "External Systems" {
        [External Logistics APIs]:::external
        [Payment Gateway]:::external
        [Maps & Geo API]:::external
    }




    %%  СВЯЗИ И ИНТЕРФЕЙСЫ 

    %% Frontend -> Backend
    [Client Web App] -- "HTTPS / REST JSON" --> [API Gateway]
    [Courier Mobile App] -- "HTTPS / REST / WebSocket" --> [API Gateway]

    %% Gateway -> Services
    [API Gateway] ..> OrderCore : Routing
    [API Gateway] ..> Analytics : Routing

    %% Order Core Interactions
    OrderCore -- "gRPC (Fast Calc)" --> Optimizer
    OrderCore -- "Internal API" --> Warehouse
    OrderCore -- "Async Events" --> Notifier
    OrderCore -- "JDBC / ORM" --> DB
    OrderCore ..> [Redis Cache] : Caching

    %% Warehouse Interactions
    Warehouse -- "JDBC" --> DB

    %% Optimization Interactions
    Optimizer -- "REST" --> [Maps & Geo API] : Geocoding
    Optimizer ..> DB : Read Orders

    %% Integrations
    CourierBridge -- "Webhooks / REST" --> [External Logistics APIs]
    CourierBridge -- "Update Status" --> OrderCore
    
    OrderCore -- "Secure TLS" --> [Payment Gateway]

    %% Analytics & Notifications
    Analytics ..> DB : Read Only Replicas
    Notifier -- "SMTP / Push" --> [Client Web App]
    Notifier -- "Push Notifications" --> [Courier Mobile App]

```



API Gateway: Единая точка входа для всех запросов. Отвечает за маршрутизацию, безопасность (аутентификация) и балансировку нагрузки. Скрывает сложность микросервисов от фронтенда.

Order Management Service (Ядро): Центральный компонент. Управляет жизненным циклом заказа (создан -> оплачен -> собран -> передан курьеру -> доставлен).

Route Optimizer Engine: Математический модуль. Принимает список адресов и параметров груза, возвращает оптимальный маршрут. Вынесен отдельно, так как требует больших вычислительных ресурсов.

Courier Integration Adapter: Слой абстракции для общения с внешними курьерскими службами (DHL, FedEx, локальные курьеры). Преобразует внутренние форматы данных системы в форматы внешних API.

Warehouse System: Отвечает за учет остатков (SKU), бронирование товара под заказ и формирование накладных на сборку.
