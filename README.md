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
├── .gitignore
├── bootstrap.php
├── README.md
├── public/
│   └── index.php
├── bin/
│   ├── seed.php
│   └── worker.php
├── views/
│   └── dashboard.php
├── storage/                      ← auto-created; leave empty
└── src/
    ├── Domain/
    │   ├── ValueObject/Priority.php
    │   ├── ValueObject/TicketStatus.php
    │   ├── Event/TicketEvent.php
    │   ├── Event/TicketEventType.php
    │   ├── Entity/Ticket.php
    │   └── Repository/TicketRepository.php
    ├── Application/
    │   ├── DTO/CreateTicketRequest.php
    │   ├── Exception/ValidationException.php
    │   ├── Exception/NotFoundException.php
    │   ├── Exception/AuthException.php
    │   ├── Service/TicketService.php
    │   ├── Service/EscalationService.php
    │   ├── Service/AuthService.php
    │   ├── Strategy/SlaCalculator.php
    │   └── Strategy/SlaPolicyByPriority.php
    └── Infrastructure/
        ├── Container.php
        ├── Database.php
        ├── Env.php
        ├── Logger.php
        ├── Event/EventDispatcher.php
        ├── Event/InMemoryDispatcher.php
        ├── Event/LoggingListener.php
        ├── Http/Response.php
        ├── Http/Router.php
        ├── Http/ErrorHandler.php
        ├── Http/Middleware/AuthMiddleware.php
        ├── Http/Controller/AuthController.php
        ├── Http/Controller/TicketController.php
        └── Persistence/PdoTicketRepository.php
        
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
