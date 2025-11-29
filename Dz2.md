```mermaid

graph TD
    classDef frontend fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef backend fill:#e1f5fe,stroke:#0277bd,stroke-width:2px;
    classDef data fill:#fff3e0,stroke:#ef6c00,stroke-width:2px;
    classDef external fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,stroke-dasharray: 5 5;

    %% 1.  ПРЕДСТАВЛЕНИЯ
    subgraph Presentation ["Presentation Layer (Frontend)"]
        ClientWeb[Client Web App]:::frontend
        CourierApp[Courier Mobile App]:::frontend
    end

    %% 2.  БИЗНЕС-ЛОГИКA
    subgraph BackendSys ["Backend Ecosystem"]
        APIGateway[API Gateway]:::backend
        
        OrderCore[Order Management Service]:::backend
        Warehouse[Warehouse System]:::backend
        Optimizer[Route Optimizer Engine]:::backend
        Analytics[Analytics & Reporting]:::backend
        Notifier[Notification Service]:::backend
        CourierBridge[Courier Integration Adapter]:::backend
    end

    %% 3.  ДАННЫE
    subgraph DataLayer ["Data Layer"]
        DB[(Main Database SQL/NoSQL)]:::data
        Redis[Redis Cache]:::data
    end

    %% 4. ВНЕШНИЕ СИСТЕМЫ
    subgraph ExternalSys ["External Systems"]
        ExtLogistics[External Logistics APIs]:::external
        PaymentGw[Payment Gateway]:::external
        GeoAPI[Maps & Geo API]:::external
    end





    %%  СВЯЗИ 

    %% Frontend -> Gateway
    ClientWeb -- "HTTPS / REST JSON" --> APIGateway
    CourierApp -- "HTTPS / WebSocket" --> APIGateway

    %% Gateway Routing
    APIGateway --> OrderCore
    APIGateway --> Analytics

    %% Order Core Logic
    OrderCore -- "gRPC (Fast Calc)" --> Optimizer
    OrderCore -- "Internal API" --> Warehouse
    OrderCore -- "Async Events" --> Notifier
    OrderCore -- "JDBC / ORM" --> DB
    OrderCore -.-> Redis

    %% Warehouse Logic
    Warehouse --> DB

    %% Optimizer Logic
    Optimizer -- "REST" --> GeoAPI
    Optimizer -.-> DB

    %% Courier Bridge
    CourierBridge -- "Webhooks / REST" --> ExtLogistics
    CourierBridge -- "Update Status" --> OrderCore

    %% Payments
    OrderCore -- "Secure TLS" --> PaymentGw

    %% Analytics & Notifications
    Analytics -.-> DB
    Notifier -.-> ClientWeb
    Notifier -.-> CourierApp
```



API Gateway: Единая точка входа для всех запросов. Отвечает за маршрутизацию, безопасность (аутентификация) и балансировку нагрузки. Скрывает сложность микросервисов от фронтенда.

Order Management Service (Ядро): Центральный компонент. Управляет жизненным циклом заказа (создан -> оплачен -> собран -> передан курьеру -> доставлен).

Route Optimizer Engine: Математический модуль. Принимает список адресов и параметров груза, возвращает оптимальный маршрут. Вынесен отдельно, так как требует больших вычислительных ресурсов.

Courier Integration Adapter: Слой абстракции для общения с внешними курьерскими службами (DHL, FedEx, локальные курьеры). Преобразует внутренние форматы данных системы в форматы внешних API.

Warehouse System: Отвечает за учет остатков (SKU), бронирование товара под заказ и формирование накладных на сборку.
