Looks good in view mode

					HTTP REQUEST
                         │
                         ▼
              ┌────────────────────┐
              │      Tomcat        │
              │  Network / Servlet │
              │      container     │
              └─────────┬──────────┘
                        │
                        ▼
              ┌────────────────────┐
              │ DispatcherServlet  │
              │  Front Controller   │
              └─────────┬──────────┘
                        │
                        │ "Who handles this?"
                        ▼
              ┌────────────────────┐
              │  HandlerMapping    │
              │                    │
              │ GET /users/{id} ───┼──► UserController
              └─────────┬──────────┘
                        │
                        │ HandlerMethod
                        ▼
              ┌────────────────────┐
              │   HandlerAdapter   │
              └─────────┬──────────┘
                        │
                        ▼
              ┌────────────────────┐
              │  UserController    │
              │                    │
              │ getUser(42)        │
              └─────────┬──────────┘
                        │
                        ▼
                    Service
                        │
                        ▼
                   Repository
                        │
                        ▼
                     Database


Database
   ↓
Repository
   ↓
Service
   ↓
Controller
   ↓
HandlerAdapter
   ↓
DispatcherServlet
   ↓
MessageConverter
   ↓
JSON
   ↓
Tomcat
   ↓
HTTP Response
   ↓
Client