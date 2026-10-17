# SLA_TicketingSystem_php
SLA = Service Level Agreement. In this ticketing system context, it's the promised time window within which support must respond to a ticket. If breached, the ticket escalates.
"If you report a critical bug, we'll respond within 4 hours — or we pay a penalty."
# Example
Think of it like a pizza delivery guarantee:
🍕 "Delivery in 30 minutes or it's free" = SLA
⏰ 30 minutes = SLA window (like slaHours())
🕐 12:00 + 30min = 12:30 = SLA deadline (like slaDueAt)
📦 Pizza arrives at 12:45 = SLA breached (like isSlaBreached())
💰 Refund issued = escalation (like escalate())
Same idea, different domain. SLA is the clock that enforces accountability.

# 📊 Requirement Coverage
Requirement	Status	Where
UI	✅	views/dashboard.php + GET / (no auth)
REST API	✅	10 endpoints in Router (see README)
DB	✅	SQLite auto-created by Database::migrate() — 5 tables, 2 indexes
Error handling	✅	ErrorHandler (422/404/401/400/500) + domain exceptions + swallow on audit failure
Time complexity	✅	Indexed idx_sla_scan (worker), idx_org_status_priority (list), bounded LIMIT 200, pagination cap
Design patterns	✅	Repository, Strategy, Observer, DI, Factory, Front Controller, Middleware, DTO — all used in code, not just named
SOLID	✅	Verified — see bootstrap.php: TicketService receives interfaces only

# 🗂️ Project Structure
sla-ticketing/
├── composer.json
├── .env
├── .env.example
├── bootstrap.php
├── bin/
│   ├── migrate.php
│   ├── seed.php
│   └── worker.php
├── public/
│   └── index.php
├── views/
│   └── dashboard.php
└── src/
    ├── Domain/
    │   ├── Event/
    │   │   ├── DomainEvent.php
    │   │   └── Event.php
    │   ├── Ticket/
    │   │   ├── Ticket.php
    │   │   ├── TicketPriority.php
    │   │   ├── TicketRepository.php
    │   │   └── TicketStatus.php
    │   └── User/
    │       ├── User.php
    │       ├── UserRepository.php
    │       └── UserRole.php
    ├── Application/
    │   ├── AuthService.php
    │   ├── DTO/
    │   │   ├── CreateTicketRequest.php
    │   │   ├── LoginRequest.php
    │   │   └── TicketResponse.php
    │   ├── EscalationService.php
    │   ├── Strategy/
    │   │   ├── CriticalSlaStrategy.php
    │   │   ├── HighSlaStrategy.php
    │   │   ├── LowSlaStrategy.php
    │   │   ├── NormalSlaStrategy.php
    │   │   ├── SlaStrategy.php
    │   │   └── SlaStrategyFactory.php
    │   └── TicketService.php
    └── Infrastructure/
        ├── Auth/
        │   └── Jwt.php
        ├── Container.php
        ├── Env.php
        ├── Event/
        │   ├── EventDispatcher.php
        │   ├── EventStoreListener.php
        │   └── LogListener.php
        ├── Http/
        │   ├── AuthMiddleware.php
        │   ├── CorsMiddleware.php
        │   ├── Middleware.php
        │   ├── Request.php
        │   ├── Response.php
        │   └── Router.php
        ├── Logger.php
        └── Persistence/
            ├── Database.php
            ├── PdoEventStore.php
            ├── PdoTicketRepository.php
            ├── PdoUserRepository.php
            └── Schema.php
        
# --- How to Run --- #
# 1. Create project folder
mkdir sla-ticketing && cd sla-ticketing
# 2. Create every file above (structure in tree)
# 3. Install dependencies
composer install
# 4. Seed demo data
php bin/seed.php
# 5. Start the server
php -S localhost:8000 public/index.php

# --- Then test the API --- # 
# Get JWT (valid 1 hour)
TOKEN=$(curl -s -X POST localhost:8000/api/auth/login \
    -d '{"email":"admin@demo.io","password":"changeme"}' \
    | php -r 'echo json_decode(stream_get_contents(STDIN))->token;')
# Create a ticket
curl -X POST localhost:8000/api/tickets \
    -H "Authorization: Bearer $TOKEN" \
    -d '{"subject":"Prod DB down","priority":"critical"}'
# List tickets
curl localhost:8000/api/tickets -H "Authorization: Bearer $TOKEN"
# Trigger the escalation worker manually
curl -X POST localhost:8000/api/escalate -H "Authorization: Bearer $TOKEN"
# Dashboard (no auth required for demo)
open http://localhost:8000/
# 6. Open http://localhost:8000 in your browser

======================================================